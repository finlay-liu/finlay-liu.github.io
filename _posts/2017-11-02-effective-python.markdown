---
layout:     post
title:      "Effective Python读书笔记"
subtitle:   " \"Reading Effective Python\""
date:       2017-11-2 20:10:00
author:     "Hux"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - 读书
---

# ch1 用Pythonic的方式来思考

## 确认自己使用的Python版本

```python
# 在shell
python --version

# 或者py代码
import sys
print(sys.version_info)
print(sys.version)
```

- 有两个活跃的Python版本：Python2和Python3；
- 有很多流兴的Python运行环境，如CPython、Jython和PyPy等；
- 运行代码时，确认系统的Python版本和代码是否相符合；
- 如果可能优先使用Python3；

## 遵循PEP8风格指南

英文：https://www.python.org/dev/peps/pep-0008/

中文：http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/

- **空白**：Python中空白会影响代码的含义，还会影响代码的清晰程度；
  - 使用空格而不是tab来缩进；
  - 使用4个空格来完成语法缩进；
  - 每行字符不超过79个；
  - 对于多行表达式，其余各行应在首行缩进上加上4个空格；
  - 函数和类使用两行隔开；
  - 在同一个类中，方法之间使用一个空行隔开；
  - 使用下标来获取元素、调用函数或给关键字赋值时，两旁不需要空格；
  - 变量赋值时，赋值符号左右各加一个空格；
- **命名**：命名方式可以清晰看出他们的角色；
  - 函数、变量及属性应该用小写字母，各单词以下划线组合；
  - 受保护的实例属性使用单个下划线开头；
  - 私有的视力属性使用两个下划线开头；
  - 类与异常以首字母大写方式命名；
  - 模块级别的常量全部使用大写字母组成，单词之间以下划线组合；
  - 类中的实例方法，应该把收个参数命名为`self`来表示对象自身；
  - 类方法的首个参数命名为cls，表示类本身；
- **表达式和语句**：语句最好有直接简洁的表达方式；
  - 采用内联的否定词，不用把否定词放在整个表达式前面，写`if a is not b`而不是`if not a is b`
  - 不要通过检测长度的方法来判断变量是否为空，而是采用`if not somelist`的方法；
  - 不要编写单行由`if`、`for`或者`while`组成的复合语句，而是拆分成多行；
  - `import`语句要放在开头；
  - 引入模块尽量使用绝对名称；
  - 使用三个部门来`import`库，分别是标准库、第三方库和自用模块；

## 了解`bytes`、`str`和`unicode`区别

Python3中有两种表示字符序列的类型：`bytes`和`str`，前者实例包括原始的8位值，后者实例包含`unicode`字符。Python2也有两种表示字符序列的类型：`str`和`unicode`，前者实例包括原始的8位值，后者实例包含`unicode`字符。

Python3的`str`实例和Python2的`unicode`实例并没有特定的二进制编码形式关联。要想把Unicode字符转换成二进制数据，就需要使用`encode`方法；要想把二进制数据转换成Unicode字符，就需要使用`decode`方法。

编写程序时，一定要把编码和解码操作放在最外围来操作，程序的核心代码应该使用Unicode字符类型，而不要对字符编码做任何假设。

## 用辅助函数来渠道复杂的表达式

表达式如果比较复杂，可以考虑将其拆解，并将逻辑转换成辅助函数。

## 了解切片的使用方法

在进行切片时，不要写多余的代码；切片也不会计较是够越界；切片会产生一份全新的列表。

## 不要在单次切片操作内同时指定`start`、`end`和`stride`
## 使用列表推导来取代`map`和`filter`

```python
a = range(10)
# 列表推导
squares = [x**2 for x in a]
print(squares)

# 使用map就要创建lambda函数
squares = map(lambda x: x**2, a)

even_squares = [x**2 for x in a if x % 2 == 0]
even_squares = map(lambda x: x**2, filter(lamda x: x % 2 == 0, a))
```

## 不要使用含有两个以上的表达式的列表推导

列别推导支持多级循环，每一级循环也支持多项条件。超过两个表达式的列表推导是很难理解的。

## 用生成器表达式来改写而数据量大的列表推导

```python
# 列表推导
value = [len(x) for x in open('/tmp/tmp_file.tex')]
# 生成器
it = (len(x) for x in open('/tmp/tmp_file.tex'))
print(it), print(next(it))
```

## 尽量使用`enumerate`取代`range`
## 用`zip`同时遍历两个迭代器
## 不要在`for`和`while`循环后面写`else`块

```python
# else也会执行
for i in range(3):
    print(i)
else:
    print('else')

# else不会执行
for i in range(3):
    print(i)
    if i == 1:
        break
else:
    print('else')
    
# 直接执行else
for i in []:
    print(i)
else:
    print('else')
```

## 合理利用`try/except/else/finally`结构中的每个代码块

- `finally`块

  如果一场异常要向上传播，又要在异常发生时执行清理工作。

  ```python
  handle = open('/tmp/tmp_file.tex')
  try:
      data = handle.read()
  finally:
      handle.close()
  ```

- `else`块


  如果`try`块没有发生异常，就执行`else`块。

- 混合使用

  `try`代码来执行，`except`来处理异常，`else`来是更新异常回报，最后使用`finally`清除资源。	
