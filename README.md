# Vuln Labs Notes

> 本仓库收录个人在 **本地授权靶场**（DVWA / sqli-labs / Pikachu / Pikachu-SSRF / xss-labs）上**独立完成**的漏洞复现与绕过记录。每篇 writeup 包含：靶场环境、Payload、原始响应截图、修复建议、复测验证。

## 目录

| 类别 | 关卡 / 漏洞 | writeup |
|---|---|---|
| SQL 注入 | DVWA 联合查询（Low/Medium/High） | [`sqli/01-dvwa-union.md`](sqli/01-dvwa-union.md) |
| SQL 注入 | DVWA 盲注自动化（自写 Python 脚本） | [`sqli/02-dvwa-blind-automation.md`](sqli/02-dvwa-blind-automation.md) |
| XSS | xss-labs 1-18 过关全记录 | [`xss/01-xss-labs-level1-18.md`](xss/01-xss-labs-level1-18.md) |
| SSRF | Pikachu file:// 读文件 / Gopher 攻击 MySQL | [`ssrf/01-pikachu-file-protocol.md`](ssrf/01-pikachu-file-protocol.md) |
| CSRF | Pikachu GET/POST 型 + DVWA low/medium | [`csrf/01-pikachu-dvwa-csrf.md`](csrf/01-pikachu-dvwa-csrf.md) |

## 写作约定

每篇 writeup 模板：

```markdown
# <靶场> <关卡> <漏洞类型>

**靶场**：xxx（本地授权 / 公司平台 ctf.eagleslab.com）
**安全等级**：Low / Medium / High
**时间**：20xx.xx.xx

## 1. 漏洞验证
## 2. 利用 / 复现步骤
## 3. 实测结果（附原始响应截图）
## 4. 修复建议
## 5. 复测
```

## 范围声明

- **不**收录公开商业产品 0day
- **不**收录未授权真实站点测试
- 截图中的 session / cookie 已脱敏
- 理论原理不写在本仓库（参考 OWASP / 培训机构讲义）；本仓库只记录**「我怎么打的」**
