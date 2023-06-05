# 第一章 起步
- 搭建编程环境
  - python2和python3，最好使用python，2的代码可能无法准确运行在3的环境
  - 自带了一个终端窗口中运行的解释器
- 搭建环境
  - Linux
    - 检查版本 
      - python
      - 默认的版本是2.7.6
      - 退出的命令为ctrl+D，或则exit()
      - python3 可以看到新的版本呢
      - 
    - 安装文本编辑器
      - sudo apt-get install geany
    - 运行 hello world程序
      - 配置使用正确的版本
  - OS X系统中
  - windows系统中
    - 安装python
    -  启动终端会话
    -  在终端会话运行python
    -  安装文本编辑器
    -  配置
    -  运行程序
-  解决安装问题
-  从终端运行python程序

# 第二章 变量和简单数据类型

- 运行时的情况：语法，不同颜色突出
- 变量：会记录变量的最新值
  - 命名和使用
    - 只能包含字母，数字，下划线，不能数字开头
    - 不能包含空格
    - 不要用关键字或函数名
    - 慎用小写L和大写O
  - 避免命名错误
- 字符串：可以单引号或者双引号
  - 修改大小写
  - 合并（拼接）字符串： 用+
  - 使用制表符\t或则和换行符\n
  - 删除空白rstrip()，lstrip(),strip()
  - 避免语法错误
- 数字
  - 整数 **表示乘方， /会得到小数
  - 浮点数：小数
  - 使用函数str()避免类型错误，将其他类型转换为字符串
- 注释 用#
- import this

# 第三章 列表
- 是什么：用[]来表示列表，用逗号分隔元素
  - 访问列表元素[n]，用元素的位置或索引
  - 索引从0而不是1开始
  - 使用列表的各个值
- 修改，添加和删除元素：列表是动态的
  - 修改：赋值
  - 添加：：xx.append()添加到末尾，xxx.insert(index,xs) 在列表中插入元素
  - 删除：del xxx[n],xx.pop()删除尾部元素，xxx.pop(n)弹出任何位置处的元素，xxx.remove()根据值删除元素
- 组织列表
  - sort() : 永久排序
  - sorted() : 临时排序
  -  反转列表： reverse()
  -  len() 长度
-  避免索引错误

# 第四章 操作列表
- 遍历列表 

```
for a in as： 
print(a)
```

  - 多条语句，同样缩进就可以
  - print()会自动换行 
  - 循环结束，取消缩进
- 避免缩进错误
  - 忘记缩进
  - 不必要的缩进
  - 遗漏了冒号
- 创建数值列表
  - 使用函数range()
  - 使用range()生成列表`list(range(1,6))` 还可以指定步长
  - 统计计算 min,max,sum
  - 列表解析 `[value**2 for value in range(1,11)]`
- 使用列表的一部分
  - 切片:指定第一个和最后一个索引`players[0:3]`，可以返回最后三名`players[-3:]`
  - 遍历切片：for循环遍历
  - 复制列表：`friend_foods = my_food[:]`
- 元组：不可变的列表
  - 定义元组：使用圆括号而不是方括号，使用索引来范围问元素`dimensions = (200,50)   dimensions[0]`
  - 遍历元组中的值 ： 用for循环
  - 修改元组变量：可以给存储元组的变量赋值，相当于重新定义元组
- 设置代码格式
  - 格式设置指南
  - 缩进：建议4个空格
  - 行长：不超过80字符
  - 空行
  - 其他代码格式设置指南

# 第五章 if 语句
- 简单示例 
```
if car == 'bmw' 
  print(car.upper())
```
- 条件测试
  - 检查是否相等
  - 检查是否相等的时候不考虑大小写 `car.lower ==`
  - 检查是否不相等 `!=`
  - 比较数字
  - 多个条件 and ,or, in ， not in
  - 布尔表达式
- if 语句
  - 执行紧跟的，缩进后的语句
  - if-else语句
  - if-elif-else
  - else可以省略
- 使用if语句处理列表
  - 检查特殊元素
  - 检查列表是不是空的
  - 使用多个列表
- 设置if语句的格式
  - 运算符两边各添加一个空格

# 第六章 字典
- 基本的字典/对象： `alien_0 = {'color'；'green','point':5}`
- 使用字典：
  - 访问字典中的值`alien_0['color']`
  - 添加键值对`alien_0['x_position']=0`
  - 先创建一个空字典 alien_o= {}
  - 修改字典中的值
  - 删除键值对： del alien_0['points']
- 遍历字典
  - 遍历所有键值对 `for key,value in user_0.items():`
  - 遍历所有键：keys(),默认也是遍历所有的键，也可以用keys()确定元素是否在键里
  - 按顺序遍历字典中的键`for name in sorted(favorite_languages.keys())`
  - 遍历字典中的所有值 values(),剔除重复想，可以使用集合set（）
- 嵌套
  - 字典列表
  - 字典中存储列表
  - 字典中存储字典

# 第七章 用户输入和while循环
 - 函数input() 的工作原理
  - 让程序暂定运行，等待用户输入一些文本
  - 函数input() 接受一个参数
  - 使用int()来获取数值输入：将输入转换成数值显示
  - 求模运算符：%：两个数相除并返回余数
  - python2.7中：raw_input()获取输入
- while循环简介：循环不断地运行，知道指定的条件不满足位置
  - 使用while循环
  ```
  num = 1
  while num <= 5:
    print(num)
    num += 1
  ```
  - 让用户选择何时退出
  - 使用标志：active = True
  - 使用break 退出循环，if a == b: break
  - 在循环中使用continue: 返回循环开头，并根据条件测试结果决定是否继续执行循环
  - 避免无限循环
- 使用while 来处理列表和字典
  - 在列表之间移动元素：while array : 当列表不为空的时候一直运行。pop().append()
  - 删除包含特定值的所有列表元素：remove(),
  - 使用用户输入来填充字典：使用标志控制


# 第八章 函数
函数是带名字的代码块：完成具体的工作。存储在模块的独立文件中
- 定义函数: 使用def关键字来定义一个函数，指出函数名。加上冒号：文档字符串（docstring）的注释，描述了函数是做什么的。文档字符串用三引号括起
```
def greet():
	print("hello")
greet()
``` 
  - 向函数传递信息：传参
  - 实参和形参
    - 函数定义中，参数为形参：函数完成工作所需的一项信息
    - 代码中，参数为实参L调用函数时传递给函数的信息
- 传递实参
  - 位置实参：根据顺序区分多个参数。多次调用函数
  - 关键字实参：`greet(animal_type='hmaster',pet_name='harry')`,将名称和值关联起来
  - 默认值：形参定义的时候，给一个默认值
  - 等效的函数调用
  - 避免实参错误
- 返回值 ： return
  - 返回简单值：不能直接函数+的结果
  - 让实参变成可选的：给某个实参指定默认值：空字符串。并将其移到形参的末尾
  - 返回字典：可以返回任何类型的值，包括列表和字典等复杂的数据结构
  - 结合使用函数和while循环：
- 传递列表
  - 在函数中修改列表
  - 禁止函数修改列表：将列表的副本传递给函数：list_name[:]
- 传递任意数量的实参 ` def make_pizza(*toppings):`
  - 不同类型的实参，任意数量的形参需要放在最后
  - 使用任意数量的关键字实参 `def build_profile(first,last,**user_info):`
- 将函数存储在模块中
  - 导入整个模块：.py文件:import的时候，程序会将所有函数复制到这个程序中。调用：可以用模块的名称+函数名
  - 导入特定的函数：`from module_name import function_name,function_1,funciton_2`
  - 使用as 给函数指定别名 `from pizza import make_pizza as mp`
  - 使用as给模块指定别名 `import pizza as p`
  - 导入模块中所有函数 `from pizza import *`
  - 函数编写指南

# 第九章 类
面向对象编程：有通用属性和通用方法，需要实例化
- 创建和使用类
  - 创建dog类
    - 首字母大写的名称是类
    - 方法_init_():self必须，会自动传递
    ```
    class Dog(): 
      def __init__(self, name, age):  
          self.name = name 
          self.age = age 

      def sit(self): 
          print(self.name.title() + " is now sitting.") 
      def roll_over(self): 
          print(self.name.title() + " rolled over!")
    ```
  - 根据类创建实例 `mydog = Dog('whilee',6)`
    - 访问属性：句点表示 `mydog.name`
    - 调用方法：mydog.sit()
    - 创建多个实例
- 使用类和实例
  - 创建一个类
  - 给默认值
  - 修改属性的值
    - 直接修改：`mydog.name = 'bob'`
    - 通过方法修改
    - 通过方法对属性的值进行递增
- 继承
  - `class ElectricCar(Car)` , `super()._init_(make,model,year)`
  - 给子类定义属性和方法
  - 重写父类发方法
  - 将实例用作属性：可以理解为实例嵌套，句点链式访问
  - 模拟实物
- 导入类
  - `from car import Car`
  - 在一个模块中存储多个类
  - 从一个模块中导入多个类
  - 导入整个模块：通过模块+句点访问类
  - 导入模块中所有的类 *
  - 在一个模块中导入拎一个模块
  - 自定义工作流程
- python 标准库
  - collections中的一个类——OrderedDict
- 类编码风格：驼峰命名法

# 第十章 文件和异常
- 从文件中读取数据
  -  读取整个文件
    -  函数open()：打开文件，访问他
    -  关键字with在不再需要访问文件后将其关闭
    -  read() 方法：读取，到达文件末尾时返回一个空字符串
  ```
  with open('pi_digits.txt') as file_object: 
    contents = file_object.read() 
    print(contents)
  ```
  - 文件路径
    - windows 中文件路径要用反斜杠\
    - 或者用绝对路径
  - 逐行读取
    - `for line in file_object:`
    - 每行的末尾有一个看不见的换行符
  - 创建一个包含文件各行内容的列表
    - `lines = file_object.readlines()`
  - 使用文件的内容
  - 包含一百万位的大型文件
    - 可以用切片，只要系统的内存足够多，就可以处理多少数据
- 写入文件
  - 写入空文件
    - 读取模式，写入模式，附加模式，读取和写入文件的模式
  ```
  filename = 'programming.txt' 
  with open(filename, 'w') as file_object: 
      file_object.write("I love programming.")
  ```
  - 写入多行
    - write（）不会再写入的文本末尾添加换行符
    - 使用\n
  - 附加到文件：附加模式
- 异常：try-except
  - 处理ZeroDivisionError异常
  - 使用try-except代码块
  ```
  try: 
     print(5/0) 
  except ZeroDivisionError: 
     print("You can't divide by zero!")
  ```
  - 使用异常避免崩溃
  - else代码块：代码成功执行的代码
  - 处理FileNotFoundError异常
  - 分析文本
  - 使用多个文件
  - 失败时一声不吭：pass语句
  - 决定报告那些错误
- 存储数据：用json
  - 使用json.dump():存储，接受两个参数。json。load():读取数据到内存
  ```
  import json 
  numbers = [2, 3, 5, 7, 11, 13] 
     filename = 'numbers.json' 
     with open(filename, 'w') as f_obj: 
     json.dump(numbers, f_obj)
     
  import json 
  filename = 'numbers.json' 
  with open(filename) as f_obj: 
      numbers = json.load(f_obj)  
  print(numbers)
  ```
  - 保存和读取用户生成的数据
  - 重构：每个函数都执行单一而清晰的任务

# 第十一章 测试代码

使用unittest 中的工具来测试代码
- 测试函数
  - 单元测试：合适函数的某个方面没有问题，和测试用例：一组单元测试，全覆盖测试用例：一整套单元测试
  - 可通过的测试
  ```
  import unittest 
  from name_function import get_formatted_name 
  class NamesTestCase(unittest.TestCase): 
       """测试name_function.py""" 

       def test_first_last_name(self): 
       """能够正确地处理像Janis Joplin这样的姓名吗？""" 
          formatted_name = get_formatted_name('janis', 'joplin') 
          self.assertEqual(formatted_name, 'Janis Joplin') 
  unittest.main()
  ```
  - 不能通过的测试
  - 测试未通过时怎么办
  - 添加新测试
- 测试类
  - 各种断言方法
  - 一个要测试的类
  - 测试类
  ```
  import unittest 
  from survey import AnonymousSurvey 
  class TestAnonmyousSurvey(unittest.TestCase):
  """针对AnonymousSurvey类的测试""" 

      def test_store_single_response(self): 
          """测试单个答案会被妥善地存储""" 
          question = "What language did you first learn to speak?" 
          my_survey = AnonymousSurvey(question) 
          my_survey.store_response('English') 

          self.assertIn('English', my_survey.responses) 

  unittest.main()
  ```
  - 方法serUp():每个测试方法中使用他们呢，优先执行它


# 项目
# 外星人入侵





