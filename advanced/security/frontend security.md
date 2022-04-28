# 前端安全问题

## 跨站脚本攻击 XSS

### 概述

Cross-Site Scripting（跨站脚本攻击）简称 XSS，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如 Cookie、SessionID 等，进而危害数据安全。

为了和 CSS 区分，这里把攻击的第一个字母改成了 X，于是叫做 XSS。

### 分类

#### 反射型XSS攻击

#### 基于DOM的XSS攻击

#### 存储型XSS攻击

### 防范

## 跨站请求伪造 CSRF

### 概述

CSRF（Cross-site request forgery）跨站请求伪造：攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。

### 分类

#### GET类型CSRF

#### POST类型CSRF

## CDN劫持

## 参考

+ [前端常见的安全问题及防范措施](https://segmentfault.com/a/1190000041454108)
+ [如何放置CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)
+ [吃透浏览器安全](https://juejin.cn/post/6991888178890145828)

