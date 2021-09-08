# XXE和PHP反序列化漏洞利用总结

## XXE

**利用前提，XML文档未禁止引用外部DTD**
	XXE全称是一XML External Entity,也就是XML外部实体注入攻击。通过XML实体,"SYSTEM"关键词导致XML解析器可以从本地文件或者远程URI中读取数据。所以攻击者可以通过XML实体传递自己构造的恶意值，使处理程序解析它。当引用外部实体时，通过构造恶意内容，可导致读取任意文件、执行系统命令、探测内网端口、攻击内网网站等危害。准确的来说XXE就是XML注入

XML文档有回显可以实现文件读取。

1. file_get_contents(“php://input”)可以读取 POST 提交的数据
2. 那么我们通过 POST 提交 XML 代码
3. XML 代码中引用外部 DTD，读取黑客想要的系统文件
4. 通过 simplexml_load_string()函数显示数据。

```xml-dtd
#读取配置文件
<!ENTITY xxe SYSTEM "file:////etc/passwd" >]>

#读取PHP文件，需要将结果进行base64解码
<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=xxe.php" >]>

#使用http协议，进行端口探测
<!ENTITY xxe SYSTEM "http://192.168.0.1:22" >]>
```

XML文档无回显实现文件读取。
Attacker需要再kali服务器上构造dtd文档和接收资源的文件。

过程解析

1. 攻击者构造XML请求，目的是访问kali服务器上的dtd资源文件
2. 服务器接收到第一步的请求之后，就会去访问kali服务器中的dtd
3. 然后获取到dtd文件，再服务器解析的过程就会执行攻击者构造的代码
4. 就向kali服务器发送本地的相关资源信息

## PHP反序列化

​	PHP 反序列化漏洞又叫做 PHP 对象注入漏洞，成因在于代码中的 unserialize() 接收的参数可
控，函数的参数是一个序列化的对象，而序列化的对象只含有对象的属性，那我们
就要利用对对象属性的篡改实现最终的攻击。

**序列化可以实现将对象压缩并格式化（变成字符串），方便数据的传输和存储PHP 序列化函数**
**serialize()**    //将一个对象转换成一个字符串

**反序列化将序列化后的字符串进行反序列化还原成对象。**
**unserialize()** //将字符串还原成一个对象

对象漏洞出现得满足两个前提：

1. **unserialize() 参数用户可控**
2. **参数被传递到方法中被执行，并且方法中使用了危险函数。**
  如 php 代码执行函数、文件读取函数、文件写入函数等等。

 序列化-魔术方法
PHP 将所有以 __（两个下划线）开头的类方法保留为魔术方法，这些都是 PHP 内置的方法。

**__construct 当一个对象创建时被调用，**
**__destruct 当一个对象销毁时被调用，**
**__wakeup() 使用 unserialize 时触发**
**__sleep() 使用 serialize 时触发**
**__call() 在对象上下文中调用不可访问的方法时触发**
__callStatic() 在静态上下文中调用不可访问的方法时触发
__get() 用于从不可访问的属性读取数据
__set() 用于将数据写入不可访问的属性
__isset() 在不可访问的属性上调用 isset()或 empty()触发
__unset() 在不可访问的属性上使用 unset()时触发
__toString() 把类当作字符串使用时触发,返回值需要为字符串
__invoke() 当脚本尝试将对象调用为函数时触发
更多魔术方法详见：https://www.php.net/manual/zh/language.oop5.magic.php