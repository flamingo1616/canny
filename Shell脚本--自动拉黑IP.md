# Shell脚本--自动拉黑IP

#### 脚本介绍


脚本设计思路：通过计划任务统计单位时间内的IP访问量，设定一个阀值，访问量超过阀值就自动拉黑。

```shell
#!/bin/bash
#该脚本可以根据web日志的访问量，自动拉黑IP（加入计划任务，结合计划任务在固定时间段内执行，并根据该时间段内产生的日志进行分析）

#首先把日志保存到根目录一份，计算日志有多少行
line1=`wc -l /access_log|awk '{print$1}'`
cp /var/log/httpd/access_log /

#计算现有的日志有多少行
line2=`wc -l /var/log/httpd/access_log |awk '{print$1}'`

#根据上一次备份的日志和现在拥有的行数差值，作为单位时间内分析日志访问量
tail -n $((line2-line1)) /var/log/httpd/access_log|awk '{print$1}'|sort -n|uniq -c|sort >/1.txt

cat /1.txt|while read line
do
echo $line >/line
num=`awk '{print$1}' /line`

#设定阀值num，单位时间内操作这个访问量的ip会被自动拉黑
if (($num>12))
then
ip=`awk '{print$2}' /line`
firewall-cmd --add-rich-rule="rule family=ipv4 source address='${ip}' port port=80 protocol=tcp reject" --permanent
firewall-cmd --reload

fi
done 

```



#### 脚本测试


一台centos7虚拟机，搭建有http服务

1.Web可以正常访

2.启动虚拟机centos的防火墙


3.把脚本加入计划任务

4.在kali中用nikto模拟大量的访问

nikto -h http://example.com


5.再访问可以看到本地的IP已经无法访问网页


6.通过firewall-cmd --list-all 可以看到自己本地的ip地址已经被拉黑


7.我们也可以对在/etc/firewalld/zones/public.xml文件中对防火墙的规则进行直接操作，方便我们后期对拉黑的ip进行移除，修改等操作 