---
title: Python 基础 (一)
date: 2020-02-12 00:39:00
categories:
- 计算机科学
tags: 
- python
mathjax: true
---

python 解释风格、python 变量和数据类型、python 基本运算、python 模块<!--more-->



---

### Python 解释风格

1. #### 注释

   ```python
   # this is annotation
   ```

2. #### 缩进

   ```python
   if i > 0:
       x = 1 # 有4个空格的缩进
   elif i == 0:
       x = 2
   else i < 0:
       x = 3
   ```

3. #### 续行

   * 续行符\

   ```python
   if i > 0 and\
       j < 0:
       x = 1
   ```

   * 无需续行符

   括号内部、三引号内部'''无需续行符直接换行
   
   

---

### Python 变量和数据类型

1. #### 动态的强类型语言

   ```python
   # python是动态强类型语言，变量无需声明，自动确定数据类型
   >>>pi = 3.1415926
   >>>PI = 'apple pie'
   >>>print (type(pi))
   >>>print (type(PI))
   
   <type 'float'>
   <type 'str'>
   ```

2. #### 基本数据类型

   * ##### 整型（integer）

   * ##### 浮点型（float）

   * ##### 布尔型（boolean）True，False

   * ##### 复数型（complex）

     ```python
     2+3j #虚数部分必须加j
     ```

   * ##### 序列（sequence）

     * 字符串（string）--不可变类型

       ```python
       >>>str = 'this is a string'
       >>>str = "this is also a string"
       >>>str = ''' this is a string, too'''
       
       # 当要输出含双引号的字符串时，可用单引号包裹，反之亦然
       >>> astr = 'a "blue" apple'
       >>> print(astr)
       a "blue" apple
       
       >>> bstr = "a 'black' orange"
       >>> print(bstr)
       a 'black' orange
       
       # 三引号内可以实现方便的换行，不需要转义符
       >>> cstr = '''Who is he?
       ... James Bond'''
       >>> print(cstr)
       Who is he?
       James Bond
       ```
   
     * 列表（list）--可变类型
   
       ```python
       >>>list = [123, 'sun', True]
       >>>print (type(list))
       
       <type 'list'>
       
       >>>''.join(list)        # 列表转字符串
       ```
   
     * 元组（tuple）--不可变类型
   
       ```python
       >>>tuple = (213, 'sun', True)
       >>>print (type(tuple))
            
       <type 'tuple'>
       ```
   
     * 引用方式 [下限:上限:步长]
   
       ```python
       # 列表下标从 0 开始
       >>>print (list[0])
       123
       
       # 若写明上限，则上限不包括在内
       >>>s1[:5]     # 从开始到下标4 （下标5的元素不包括在内）
       >>>s1[2:]     # 从下标2到最后
       >>>s1[0:5:2]  # 从下标0到下标4 (5不包括在内)，每隔2取一个元素
       >>>s1[::-1]   # 倒序 相当于s1[-1:-len(s1)-1:-1]（缺省默认填值）
       
       # 尾部元素引用
       >>>s1[-1]     # 序列最后一个元素
       >>>s1[-3]     # 序列倒数第三个元素
       ```
       
     * 重复 sequence*copies
   
       ```python
       >>>'apple'*3
       'appleappleapple'
       
       >>>[0]*3
       '[0,0,0]'
       ```
   
     * 拼接 sequence+sequence
   
       ```python
       >>>print('pine'+'apple')
       'pineapple'
       
       >>>print([1,2,3]+[4,5,6])
       [1, 2, 3, 4, 5, 6]
       
       >>>print((1,2,3)+(4,5,6))
       (1, 2, 3, 4, 5, 6)
       ```
   
     * 判断成员 in，not in
   
       ```python
       >>>namelist = ['Jack','Russell','Mary']
       >>>'Russell' in namelist
       True
       ```
   
     * 排序sorted，倒序reversed
   
       ```python
       >>>a = [3,6,1,9]
       >>>print(sorted(a))       
       [1, 3, 6, 9]
       >>>print(a)
       [3, 6, 1, 9]                # sorted()函数不改变原对象
       
       >>>a.sort()                  
       >>>print(a)
       [1, 3, 6, 9]                # .sort()方法对原对象进行改动
       
       # reversed()和.reverse()同理
       ```
   
   * #####  字典（dictionary）
   
      字典和序列一样，是可以储存多个元素的类（称为容器），不同的是字典包含映射关系：字典的元素有两部分——“键”和“值”；且字典没有顺序。
   
      ```python
      dic = {'name':'orange', 'shape':'sphere', 'price':2.6}
      print (dic['price'])         # 字典通过“键”引用
      
      dic['color'] = 'orange'      # 可在字典中加入新的元素
      
      for key in dic:              # 在循环中，也是通过“键”引用值
          print (dic[key])
      ```
      
      创建字典的方法
      
      ```python
      # dict()函数
      dict([('Ola',23),('Kary','24'),('Mike','22')])
      dict([['Ola',23],['Kary','24'],['Mike','22']])
      dict((('Ola',23),('Kary','24'),('Mike','22')))
      dict(Ola = 23), Kary = 24, Mike = 22)
      dict(zip(name,age))
      
      # fromkeys(seq,value) 批量赋值
      a = {}.fromkeys(('Ola','Kary','Mike'),23)
      ```
      
      字典的常见用法
      
      ```python
      dic.keys()           # 返回dic所有的键
      dic.values()         # 返回dic所有的值
      dic.items()          # 返回dic所有的元素（键值对）
      dic.clear()          # 清空dic
      del dic['xxx']       # 删除dic中的‘xxx’元素
      dic.update(newdic)   # 添加新的字典键值对
      ```
      
   * ##### 集合（set）
   
      集合是一个无序的容器，用{ }表示，且其中不包含重复的元素。
      
      ```python
      a={"Tom.py", "Mike.py","Anne.py","Denny.py","Jack.py","Fan.py"}
      b={"Tom.py", "Lily.py", "Anne.py", "Richard.py","Jack.py"}
      print('(2)',a&b)      # & 交集
      print('(4)',a|b)      # | 并集
      print('(1)',a-b)      # - 补集，a-b为差集
      print('(3)',a^b)      # ^ 对称差集
      
      {'Tom.py', 'Anne.py', 'Jack.py'}
      {'Tom.py', 'Denny.py', 'Lily.py', 'Fan.py', 'Anne.py', 'Mike.py', 
      'Richard.py', 'Jack.py'}
      {'Fan.py', 'Denny.py', 'Mike.py'}
      {'Denny.py', 'Mike.py', 'Lily.py', 'Fan.py', 'Richard.py'}
      ```
      
      集合的常见方法
      
      ```python
      set.add()             # 添加元素
      set.remove()          # 删除元素
      x in set              # 判断x是否在集合中
      ```
   
3. #### 赋值

   ```python
   >>>a = 10            # 普通赋值
   >>>a /= 5            # 增量赋值 a = a/5
   >>>b = a = a + 1     # 链式赋值 a = a+1 , b = a
   >>>a,b = 10,'orange' # 多重赋值 a = 10 , b = 'orange'
   >>>a,b,c = [1,2,3]   # 解包
   ```
   
   python中，数值、字符串、元祖等是**值类型**的对象，本身不可变；列表、字典是**引用类型**的对象，可变。
   
   ```python
   # 值类型对象是不可变的，对值类型变量的修改是使变量指向了一个新的对象
   >>>a = 10
   >>>print id(a)
   33521053L
   
   >>>a = 20
   >>>print id(a)
   27629312L
   
   # 引用类型对象是可变的，对引用类型的修改则是修改的对象本身
   >>>l = [1,2,3]
   >>>print id(l)
   39774280L
   
   >>>l[0] = 0
   >>>print id(l)
   39774280L
   ```

---

### Python 基本运算

优先级：算术运算>位运算>关系运算>逻辑运算

1. #### 算术运算

   按优先级：

   ** （乘方）

   +,-（正负）

   *（乘）, //（整除）, /（除）, %（取余） 

   +,-（加减）

2. #### 位运算（二进制运算）

   ```python
   >>>~1           # 取反
   -2
   >>> 16 << 2     # 左移
   64
   >>> 16 >> 2     # 右移
   4
   >>> 64 & 15     # 与
   0
   >>> 64 | 15     # 或
   79
   >>> 64 ^ 14     # 异或
   78
   ```

3. #### 关系运算

   !=（不等于）

4. #### 逻辑运算

   优先级：not > and > or

   

---

### Python 模块

1. #### 单个模块

   一个.py文件就是一个模块，用 import 函数导入模块，可以使用其中的函数、类等

	```python
	>>> import math          # 导入模块 math
	>>> math.pi              # 使用“模块.对象”格式调用模块中的对象

	3.141592653589793
	```
	
	还有其它导入模块的方式：
	
	```python
	import a as b            # 导入a，重命名为b
	
	from a import func1      # 从模块a中引入func1对象，调用时直接使用func1，不用再写a.function1
	
	from a import *          # 从模块a中引入所有对象，调用时直接使用对象，不用再写	a.对象
	```
	
2. #### 模块包
   
	将功能相似的模块放在同一个**文件夹**，构成一个模块包
	
	```python
	import AFolder.module    # 导入AFolder文件夹中的模块
	```

  	该文件夹中必须包含一个\_\_init\_\_.py的文件，提醒Python，该文件夹为一个模块包。\_\_init\_\_.py可以是一个空文件。

 



<br/>

<br/>

<small>*参考*</small>

<small>*[Vamei 博客园-Python快速教程](https://www.cnblogs.com/vamei/archive/2012/09/13/2682778.html)*</small>

<small>*[菜鸟教程 Python内置函数](https://www.runoob.com/python/python-built-in-functions.html)*</small>

<small>*延伸*</small>

<small>*[w3school python string methods](https://www.w3schools.com/python/python_ref_string.asp)*</small>

<small>*[菜鸟教程 Python 直接赋值、浅拷贝和深度拷贝解析](https://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html)*</small>

