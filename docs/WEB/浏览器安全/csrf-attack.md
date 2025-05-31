# CSRF 攻击详解

## 什么是 CSRF 攻击

CSRF（Cross-Site Request Forgery，跨站请求伪造）是一种恶意攻击，它利用用户已经认证的身份，诱导用户在不知情的情况下执行非预期的操作。攻击者构造恶意请求，当已登录的用户访问包含这些请求的页面时，浏览器会自动携带用户的认证信息（如 Cookie）发送请求，从而以用户身份执行恶意操作。

## 攻击原理与条件

**核心机制**：

1. 用户登录目标网站，获得认证 Cookie
2. 攻击者创建包含恶意请求的页面
3. 通过各种方式让用户访问恶意页面
4. 浏览器自动携带 Cookie 发送请求
5. 服务器认为这是合法的用户请求并执行

**攻击条件**：

- 目标网站存在可被利用的功能（转账、修改密码等）
- 用户已经登录目标网站且 Session 未过期
- 目标网站仅依靠 Cookie 进行身份验证
- 攻击者能够诱导用户访问恶意页面

## 攻击类型

**GET 型 CSRF**：通过图片标签、链接等方式触发，利用浏览器自动发送请求的特性。

```html
<!-- 隐藏的恶意请求 -->
<img
  src="https://bank.example.com/transfer?to=attacker&amount=10000"
  style="display:none;"
/>
```

**POST 型 CSRF**：需要构造表单进行提交，可以传递更多参数。

```html
<!-- 自动提交的隐藏表单 -->
<form
  id="maliciousForm"
  action="https://victim.com/change-password"
  method="POST"
  style="display:none;"
>
  <input type="hidden" name="new_password" value="hacked123" />
</form>
<script>
  document.getElementById("maliciousForm").submit();
</script>
```

**JSON 型 CSRF**：针对现代 Web API，利用 fetch API 或 XMLHttpRequest，需要绕过 CORS 限制。

**高级攻击技术**：绕过 Referer 检查（data URI、meta 标签）、链式 CSRF（多步骤攻击）、WebSocket CSRF 等。

## 攻击危害

- **财务安全**：未授权的资金转移、恶意购买商品、修改支付方式
- **账户安全**：修改密码和邮箱、更改个人信息、添加恶意联系人
- **数据完整性**：删除重要数据、发布恶意内容、修改系统配置
- **权限提升**：添加管理员账户、修改用户权限、植入后门程序

## 防护措施

**CSRF Token（最重要）**：

- 服务器为每个会话生成唯一的随机 Token
- 所有状态改变请求必须包含有效的 Token
- 攻击者无法预测或获取用户的 Token

**SameSite Cookie**：

- 限制 Cookie 在跨站请求中的发送
- 三种模式：Strict（最严格）、Lax（默认）、None（需要 Secure）

```javascript
document.cookie = "sessionId=abc123; SameSite=Strict; Secure; HttpOnly";
```

**其他防护手段**：

- **Referer 检查**：验证请求来源域名
- **自定义请求头**：利用浏览器同源策略
- **二次确认**：关键操作增加验证码或密码验证
