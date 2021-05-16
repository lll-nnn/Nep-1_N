---
title: Nep-week6
date: 2021-5-16
author: 1_N

---

## 总
这两周除了刷题外，看了看津门杯，ISCC，国赛    

一道道题都好难，感觉自己永远都达不到能解决那些题的程度，好难

······
之前都是复制博客里的内容，感觉也没什么意义，所以想着在这里写下写题中遇到的知识点吧

## ruby的ERB模板注入

    ```ruby
    if params[:do] == "#{params[:name][0,7]} is working" then

    auth["jkl"] = auth["jkl"].to_i + SecureRandom.random_number(10)
    auth = JWT.encode auth,ENV["SECRET"] , 'HS256'
    cookies[:auth] = auth
    ERB::new("<script>alert('#{params[:name][0,7]} working successfully!')</script>").result
    ```

REB模板注入格式`<%=7*7%>`---->49    
其中ruby中有些[预定义变量](https://docs.ruby-lang.org/en/2.4.0/globals_rdoc.html)，比如`$'`返回最后一次成功匹配的右边的字符串      

## php中session反序列化
* PHP中的session以文件存储，文件以`sess_session`命名，内容为session值序列化后的内容     
* `php_ini`中的`session.serialize_handler`定义用来序列化/反序列化的处理器名字（`php_binary`、`php`、`php_serialize`）    
* `php_binary`:键名长度对应的ascii字符+键名+经过序列化后的值    
* `php`: 键名+`|`+序列化后的值   
* `php_serialize`:将键名和值当作数组序列化      

通过改变序列化的模式（比如传入`serialize_handler=php`）可以达到特定的目的

## php 原生类SoapClient    
这个类的`__call`方法可以发送`http/https`请求，以此进行`SSRF`   

## CRLF Injection

简单说就是在请求头中注入`\r\n`,从而生成新的请求头，或者覆盖原来的请求头，或者注入`\r\n\r\n`,注入请求（头和体由两个`\r\n`分割） 

## php_flag    

Apache的配置文件中的`php_flag engine`如果设为`off`或`0`，则会使所设目录下的文件没有执行权限    

## .htaccess 

通过传入.htaccess文件来覆盖配置文件中的设置（.htaccess文件作用所在的目录和所有子目录，指令按查找顺序依次生效，所以子目录下的.htaccess可能会覆盖父目录下或者配置文件中的指令）