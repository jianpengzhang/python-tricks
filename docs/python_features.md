# Python 语言特性

## 1. Python 函数参数传递

**问题：** 下⾯代码，输出分别是什么？

```python
a = 1


def fun(a):
    a = 2


fun(a)
print(a)  # 1
```

```python
a = []


def fun(a):
    a.append(1)


fun(a)
print(a)  # [1]
```

**✨ 知识点：**

在 Python 中，函数参数传递涉及到 “可变类型” 和 “不可变类型” 两个概念，理解这两个概念对理解参数传递过程中的 “值变与不变”
非常重要。

**（1）不可变类型 vs 可变类型:**

* 不可变类型: 包括 int, float, str, tuple 等，这些类型的对象一旦创建，其值就不能被修改；
* 可变类型: 包括 list, dict, set 等，这些类型的对象创建后，其内容可以被修改；

**（2）参数传递机制：**

Python 中函数参数传递本质上是 “传对象引用”，这意味着在函数内部，参数名和传递的对象指向同一个内存地址。

* 不可变类型的参数传递：  
  当将 "不可变类型" 作为参数传递给函数时，函数内部对参数的修改不会影响外部的变量。这是因为修改 "不可变类型"
  的值会创建一个新的对象，而不会修改原来的对象。

  示例：

  ```python
  a = 1
  def fun(a):
      print("func_in", id(a))  # func_in 12178984
      a = 2
      print("re-point", id(a), id(2))  # re-point 12179016 12179016
  print("func_out", id(a), id(1))  # func_out 12178984 12178984
  fun(a)
  print(a)  # 1
  ```
  解释：函数内部的 a 修改并不会影响外部的 a，因为 int 是不可变类型。
* 可变类型的参数传递：  
  当将 “可变类型” 作为参数传递给函数时，函数内部对参数内容的修改会影响外部的变量。这是因为可变类型的对象内容可以被直接修改，而不会创建新的对象。

  示例：

  ```python
  a = []
  def fun(a):
    print("func_in",id(a))  # func_in 139795471960896
    a.append(1)
  print("func_out",id(a))     # func_out 139795471960896
  fun(a)
  print(a)  # [1]
  ```
  解释：函数内部对 a 的修改会影响外部的 a，因为 list 是可变类型。

**【注意事项】**

* 如果在函数内部给参数重新赋值：
  * 对于不可变类型，这实际上会创建一个新的本地变量，而不会影响外部的变量；
  * 对于可变类型，如果不想在函数内部修改传入的对象，可以使用对象的副本进行操作，例如 "a.copy()" 或者 "a[:]"；

## 2. Python 元类(metaclass)

### 2.1 元类（Metaclass）简介

在 Python 中，一切皆为对象，包括类本身，而元类（Metaclass）是用于创建类的
“类”，即元类定义了类的行为，而类定义了实例的行为，换句话说，元类控制类的创建过程，默认情况下，Python 中所有类的元类是 "type"
，它负责创建所有的类。

**元类的作用：**

* 控制类的创建：元类可以在类创建之前修改类的属性和方法，甚至可以改变类的继承结构；
* 自定义类行为：通过元类，可以定制类的创建规则，比如自动添加某些方法、验证属性等；

**使用元类：**

可以通过在类定义中使用 metaclass 参数来指定一个元类。例如：

```python
class MyMeta(type):
    def __new__(cls, name, bases, dct):
        print(f"Creating class {name}")
        return super().__new__(cls, name, bases, dct)


class MyClass(metaclass=MyMeta):
    pass
```

运行输出："Creating class MyClass"

这里 "MyMeta" 是一个元类，它重载了 `__new__` 方法，在创建类时会输出一条信息，“MyClass” 使用这个元类，因此在它被创建时，“MyMeta”
的 `__new__` 方法被调用。

**元类的常见方法：**

* `__new__(cls, name, bases, dct)`: 在类创建之前调用，用于控制类的创建过程，“name” 是类的名称，“bases” 是类的基类，“dct”
  是类的属性和方法字典。
  ```python
  class A:
    def __init__(self):
        pass


  class MyMeta(type):
      def __new__(cls, name, bases, dct):
          print(f"Creating class name {name}")
          print(f"Creating bases {bases}")
          print(f"Creating dct {dct}")
          return super().__new__(cls, name, bases, dct)
  
  
  class MyClass(A, metaclass=MyMeta):
  
      def print_a(self):
          print("Print A")
  ```
  输出：
  ```text
  Creating class name MyClass
  Creating bases (<class '__main__.A'>,)
  Creating dct {'__module__': '__main__', '__qualname__': 'MyClass', 'print_a': <function MyClass.print_a at 0x7f54808d4d60>}
  ```
* `__init__(cls, name, bases, dct)`: 在类创建之后调用，用于初始化类，此时类已经被创建，cls 是类对象本身。
* `__call__(cls, *args, **kwargs)`: 控制类实例的创建过程，通常用于单例模式。

### 2.2 元类面试题

* 面试题 1: 解释 Python 中的元类，元类的作用是什么？  
  **解答**：  
  元类是用于创建类的 “类”，控制类的创建过程。元类可以定制类的创建规则，如自动添加方法、属性校验、修改继承关系等。Python
  中的所有类默认由 “type” 这个元类创建。元类的典型用法包括实现单例模式、注册类、自动化创建方法等。

* 面试题 2: 如何使用元类实现类的单例模式？  
  **解答**：  
  单例模式是一种设计模式，保证一个类只有一个实例。通过元类可以控制类的实例化过程，从而实现单例模式。

  ```python
  class SingletonMeta(type):
      _instances = {}
  
      def __call__(cls, *args, **kwargs):
          if cls not in cls._instances:
              cls._instances[cls] = super().__call__(*args, **kwargs)
          return cls._instances[cls]
    
    
  class SingletonClass(metaclass=SingletonMeta):
      def __init__(self, value):
          self.value = value
    
    
  obj1 = SingletonClass(1)
  obj2 = SingletonClass(2)
    
  print(f"obj1:{id(obj1)}, obj2:{id(obj2)}")
  print(obj1 is obj2)  # True
  print(obj1.value)  # 1
  print(obj2.value)  # 1
  ```
  **解释**："SingletonMeta" 是一个元类，通过重载 `__call__` 方法来确保每次实例化类时，返回的都是同一个实例。

* 面试题 3: 如何通过元类自动向类添加方法？  
  **解答**：  
  可以通过重载元类的 `__new__` 方法，在类创建时自动向其添加方法。
  ```python
  class AutoMethodMeta(type):
      def auto_method_print_a(self):
          print("print_a: Print A...")
  
      def __new__(cls, name, bases, dct):
          dct['auto_method'] = lambda self: "This is an auto-generated method."
          dct['auto_method_print_a'] = cls.auto_method_print_a
          return super().__new__(cls, name, bases, dct)
  
  
  class MyClass(metaclass=AutoMethodMeta):
      pass
  
  
  obj = MyClass()
  obj.auto_method_print_a()
  print(obj.auto_method())  # This is an auto-generated method.
  ```
  **解释**：在 "AutoMethodMeta" 中，通过 `__new__` 方法向类添加了一个名为 "auto_method"、“auto_method_print_a”
  方法，因此每个使用该元类的类都会自动拥有这两个方法。

  * 面试题 4: 如何使用元类对类属性进行验证？  
    **解答**：  
    可以在元类的 `__init__` 方法中进行属性验证，确保类在创建时符合某些规则。

  ```python
  class ValidatingMeta(type):
      def __init__(cls, name, bases, dct):
          if 'required_attr' not in dct:
              raise TypeError(f"{name} must have a 'required_attr' attribute.")
          # super().__init__(cls)
          super().__init__(name, bases, dct) 
  
  
  class MyClass(metaclass=ValidatingMeta):
      required_attr = True
  
  # 这将抛出错误，因为没有定义 `required_attr`
  # class MyOtherClass(metaclass=ValidatingMeta):
  #     pass
  ```
  **解释**：ValidatingMeta 在类初始化时检查是否定义了 required_attr 属性，如果没有则抛出 TypeError 异常。

## 3. @staticmethod和@classmethod

在 Python 中，主要有三种方法：实例方法、类方法 (classmethod) 和 静态方法 (staticmethod)，这三者的区别在于它们的调用方式以及它们可以访问的对象和数据。

### 3.1 实例方法

实例方法是定义在类中，并且在类的实例上调用的方法。它们可以访问实例属性和方法，并且默认第一个参数是 self，表示类的实例。

**特性：**

* 可以访问和修改实例属性；
* 只能通过实例对象调用；

**示例：**

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def instance_method(self):
        return f"Instance method called. Value is {self.value}"


obj = MyClass(10)
print(obj.instance_method())  # 输出: Instance method called. Value is 10
```

### 3.2 类方法 (classmethod)

类方法使用 @classmethod 装饰器来定义，默认第一个参数是 cls，表示类本身，而不是类的实例，类方法可以访问类属性和类方法，但不能直接访问实例属性。

**特性：**

* 不能访问实例属性，但可以访问类属性；
* 可以通过类或实例调用；

**示例：**

```python
class MyClass:
    class_variable = "Class level variable"

    @classmethod
    def class_method(cls):
        return f"Class method called. {cls.class_variable}"


print(MyClass.class_method())  # 通过类调用，输出: Class method called. Class level variable

obj = MyClass()
print(obj.class_method())  # 也可以通过实例调用
```

### 3.3 静态方法 (staticmethod)

静态方法使用 @staticmethod 装饰器来定义，静态方法没有默认参数，它既不需要访问实例属性，也不需要访问类属性。因此，静态方法与普通的函数基本没有区别，只是它们被包含在类的命名空间中。

**特性：**

* 不能访问实例属性或类属性；
* 可以通过类或实例调用；

**示例：**

```python
class MyClass:
    @staticmethod
    def static_method():
        return "Static method called."


print(MyClass.static_method())  # 通过类调用，输出: Static method called.

obj = MyClass()
print(obj.static_method())  # 也可以通过实例调用
```

### 3.4 区别总结

| \\                 | 实例方法                                      | 类方法                                                               | 静态方法                                                             |
|:-------------------|:------------------------------------------|:------------------------------------------------------------------|:-----------------------------------------------------------------|
| 区别                 | - 第一个参数是 self，表示实例本身；<br/> - 可以访问实例属性和方法； | - 使用 @classmethod 装饰，第一个参数是 cls，表示类本身；<br/> - 可以访问类属性，不能直接访问实例属性； | - 使用 @staticmethod 装饰，没有默认参数；<br/> - 不能访问类属性或实例属性，仅与类有逻辑关系的独立方法； |
| 调用形式：obj=MyClass() | obj.instance_method()                     | obj.class_method()                                                | obj.static_method()                                              |
| 调用形式：MyClass | 不可用                                       | MyClass.class_method()                                            | MyClass.static_method()                                          |
| 使用场景               | 用于需要访问或修改实例状态的操作；                         | 用于需要访问类属性或执行一些与类有关的操作，而不需要实例化类的场景；                                | 用于独立于实例或类状态，但与类逻辑相关的操作；                                          |

## 4. 类变量和实例变量

### 4.1 类变量（Class Variables）

类变量是在类内部声明（但在任何方法之外）定义的变量，它们在类的所有实例之间共享。换句话说，类变量对于每个实例都是相同的，修改类变量的值会影响所有实例。

**特性：**

* 属于类本身，而不是某个实例；
* 在类的所有实例之间共享；
* 可以通过类名或实例名访问，但通常使用类名访问更为直观；

**示例：**

```python
class MyClass:
    class_variable = "I am a class variable"


# 创建两个实例
obj1 = MyClass()
obj2 = MyClass()

# 通过类名访问类变量
print(MyClass.class_variable)  # 输出: I am a class variable

# 通过实例访问类变量
print(obj1.class_variable)  # 输出: I am a class variable
print(obj2.class_variable)  # 输出: I am a class variable

# 修改类变量
MyClass.class_variable = "New class value"

print(obj1.class_variable)  # 输出: New class value
print(obj2.class_variable)  # 输出: New class value
```

### 4.2 实例变量（Instance Variables）

实例变量是在类的 `__init__` 方法中，或者其他实例方法中使用 self 关键字定义的变量。实例变量的值对于每个实例来说都是独立的，不同实例的实例变量可以有不同的值。

**特性：**

* 属于类的实例，而不是类本身;
* 每个实例都有自己独立的实例变量;
* 只能通过实例访问;

**示例：**

```python
class MyClass:
    def __init__(self, value):
        self.instance_variable = value  # 定义实例变量


# 创建两个实例
obj1 = MyClass("Object 1")
obj2 = MyClass("Object 2")

# 访问实例变量
print(obj1.instance_variable)  # 输出: Object 1
print(obj2.instance_variable)  # 输出: Object 2

# 修改实例变量
obj1.instance_variable = "New value for Object 1"

print(obj1.instance_variable)  # 输出: New value for Object 1
print(obj2.instance_variable)  # 输出: Object 2
```

### 4.3 类变量与实例变量的区别

* 作用域：类变量属于类，实例变量属于实例；
* 共享性：类变量在所有实例之间共享，实例变量是独立的；
* 访问方式：类变量可以通过类名或实例名访问，但实例变量只能通过实例名访问；
* 生命周期：类变量的生命周期与类相同，实例变量的生命周期与实例相同；

### 4.4 注意事项（避免混淆）

在 Python 中，当类变量是一个可变对象（如列表、字典、集合等）时，通过实例修改类变量的行为会与不可变对象（如整数、字符串、元组等）有所不同。这是由于
Python 中对象的可变性和引用机制所导致的（如第一章节）。

### 4.4.1 类变量 —— 不可变对象

对于不可变对象（如整数、字符串、元组等），如果通过实例对类变量进行赋值（修改）操作，Python 会在实例的命名空间中创建一个新的同名实例变量，而不会修改类变量。例如：

```python
class MyClass:
    class_variable = "Class variable"


obj = MyClass()

# 通过实例赋值操作修改类变量（实际是创建了一个实例变量）
obj.class_variable = "Modified by instance"

print(obj.class_variable)  # 输出: Modified by instance
print(MyClass.class_variable)  # 输出: Class variable
```

在这个例子中，obj.class_variable = "Modified by instance" 实际上在 obj 的实例命名空间中创建了一个新的实例变量
class_variable，而类变量 MyClass.class_variable 并未受到影响。

#### 4.4.2 类变量 —— 可变对象

当类变量是一个可变对象（如列表）时，通过实例对其进行修改操作（例如添加元素、修改元素）时，由于可变对象是通过引用传递的，所有引用该类变量的实例都会看到修改后的值。这就意味着，即使是通过实例来修改类变量，它实际影响的是类变量本身，而不会在实例命名空间中创建新的变量。

示例：

```python
class MyClass:
    class_variable = []  # 类变量是一个列表

    def __init__(self, value):
        self.instance_variable = value


obj1 = MyClass("Instance 1")
obj2 = MyClass("Instance 2")

# 通过实例修改类变量（实际是修改类变量本身）
obj1.class_variable.append(1)

print(obj1.class_variable)  # 输出: [1]
print(obj2.class_variable)  # 输出: [1]
print(MyClass.class_variable)  # 输出: [1]
```

#### 4.4.3 总结

* 不可变对象：通过实例修改类变量时，会在实例命名空间中创建一个新的实例变量，类变量不会被修改；
* 可变对象：通过实例修改类变量时，由于修改的是引用对象，类变量本身会被修改，所有实例都能看到修改后的结果；

因此，在使用类变量作为可变对象时，需要小心其共享特性，以避免意外修改影响所有实例。如果希望每个实例都有自己的独立副本，通常应该在 `__init__`
方法中初始化该变量为实例变量，而不是使用类变量。同时，建议通过类名访问和修改类变量避免混淆。

## 5. Python 自省

Python 自省（Introspection）是指在运行时检查对象的能力。通过自省，Python 程序可以在运行时动态获取对象的类型、属性、方法、父类、模块等信息。自省机制使
Python 具有强大的动态性，能够根据当前对象的状态进行相应的操作。

### 5.1 自省主要功能

#### 5.1.1 获取对象类型

* `type()`：用于获取对象的类型；
* `isinstance()`：检查对象是否是某个类的实例；
* `issubclass()`：检查一个类是否是另一个类的子类；

```python
x = 42
print(type(x))  # 输出: <class 'int'>
print(isinstance(x, int))  # 输出: True


class A: pass


class B(A): pass


print(issubclass(B, A))  # 输出: True
```

#### 5.1.2 获取对象属性和方法

* `dir()`：返回对象的所有属性和方法名列表；
* `hasattr()`：检查对象是否具有某个属性或方法；
* `getattr()`：获取对象的某个属性或方法；
* `setattr()`：设置对象的某个属性或方法；
* `vars()`：返回对象的 `__dict__` 属性，即对象的属性字典；

```python
class MyClass:
    def __init__(self):
        self.value = 10

    def method(self):
        return self.value


obj = MyClass()

print(dir(obj))  # 输出: ['__class__', '__delattr__', '__dict__', '__dir__', ... 'value']
print(hasattr(obj, 'value'))  # 输出: True
print(getattr(obj, 'value'))  # 输出: 10

setattr(obj, 'value', 20)
print(obj.value)  # 输出: 20

print(vars(obj))  # 输出: {'value': 20}
```

#### 5.1.3 获取类的层次结构

* `__bases__`：用于获取类的基类（父类）；
* `__mro__`：返回类的继承层次结构；

```python
class A: pass


class B(A): pass


print(B.__bases__)  # 输出: (<class '__main__.A'>,)
print(B.__mro__)  # 输出: (<class '__main__.B'>, <class '__main__.A'>, <class 'object'>)
```

#### 5.1.4  检查模块和包的内容

* `__name__`：获取模块或包的名称；
* `__file__`：获取模块的文件路径；
* `__doc__`：获取模块、类或函数的文档字符串（docstring）；

```python
import os

print(os.__name__)  # 输出: os
print(os.__file__)  # 输出: 文件路径
print(os.__doc__)  # 输出: os 模块的文档字符串
```

### 5.2 自省应用场景

#### 5.2.1 动态导入模块

根据条件动态导入模块，并调用其中的函数或方法：

```python
module_name = 'math'
module = __import__(module_name)

print(module.sqrt(16))  # 输出: 4.0
```

#### 5.2.2 动态生成类

使用 type() 动态创建类，type() 是一个类的元类（metaclass），可以用来创建新的类：

```python
MyDynamicClass = type('MyDynamicClass', (object,), {'attribute': 100})

obj = MyDynamicClass()
print(obj.attribute)  # 输出: 100
```

### 5.3 面试示例

* 问题 1：什么是 Python 自省？请举例说明：
  答：自省是指在运行时检查对象的能力。示例：可以是使用 type() 获取对象类型，使用 dir() 列出对象的属性和方法。

* 问题 2：如何在 Python 中动态获取对象的所有属性？
  答：可以使用 dir() 函数列出对象的所有属性和方法，也可以使用 vars() 获取对象的属性字典。

* 问题 3：编写代码，动态导入模块并调用其中的函数。
  答：使用 `__import__()` 动态导入模块，并调用其中的函数，如 module.sqrt(16)。

## 6. 列表、字典、集合推导式

推导式是 Python 中非常强大的语法工具，允许我们以简洁的方式生成和操作列表、字典以及集合。

### 6.1 列表推导式（List Comprehension）

**（1）基本语法**

列表推导式的基本语法形式如下：

```text
[expression for item in iterable if condition]
```

* `expression`：对每个 item 执行的表达式，可以是任意操作，用于计算新列表的每个元素；
* `item`：从 iterable(可迭代对象) 中获取的元素；
* `iterable`：要迭代的对象，例如列表、元组、字符串等；
* `condition（可选）`：可选过滤条件，只有满足条件的元素才会被包含在生成的列表中；

**（2）示例**

```python
# 创建一个包含 0 到 9 的平方的列表
squares = [x ** 2 for x in range(10)]
print(squares)  # 输出: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 过滤出偶数的平方
even_squares = [x ** 2 for x in range(10) if x % 2 == 0]
print(even_squares)  # 输出: [0, 4, 16, 36, 64]

# 使用嵌套的推导式来处理二维或多维结构：创建一个 3x3 的矩阵
# 外层推导式负责生成矩阵的行，内层推导式生成行中的每个元素
matrix = [[i * j for j in range(3)] for i in range(3)]
print(matrix)  # 输出: [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# 使用列表推导式将一个二维列表展开为一维列表
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flatten_matrix = [x for row in matrix for x in row]
print(flatten_matrix)  # 输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 6.2 字典推导式（Dict Comprehension）

**（1）基本语法**

字典推导式的基本语法形式如下：

```text
{key_expression: value_expression for item in iterable if condition}
```

* `key_expression`：字典键的表达式；
* `value_expression`：字典值的表达式；
* `item`：从 iterable(可迭代对象) 中获取的元素；
* `iterable`：要迭代的对象，例如列表、元组等；
* `condition（可选）`：可选过滤条件，只有满足条件的元素才会被包含在生成的字典中；

**（2）示例**

```python
# 创建一个字典，键是 0 到 9，值是它们的平方
square_dict = {x: x ** 2 for x in range(10)}
print(square_dict)  # 输出: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81}

# 过滤出偶数的平方
even_square_dict = {x: x ** 2 for x in range(10) if x % 2 == 0}
print(even_square_dict)  # 输出: {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# 反转字典的键和值
original_dict = {'a': 1, 'b': 2, 'c': 3}
inverted_dict = {v: k for k, v in original_dict.items()}
print(inverted_dict)  # 输出: {1: 'a', 2: 'b', 3: 'c'}

# 从元组列表中创建一个字典
tuples = [(1, 'a'), (2, 'b'), (3, 'c')]
dictionary = {k: v for k, v in tuples}
print(dictionary)  # 输出：{1: 'a', 2: 'b', 3: 'c'}
```

### 6.3 集合推导式（Set Comprehension）

集合推导式类似于列表推导式和字典推导式，但生成的结果是一个集合。集合是无序且不允许重复元素的，因此集合推导式会自动去除重复元素。

**（1）基本语法**

集合推导式的基本语法形式如下：

```text
{expression for item in iterable if condition}
```

* `expression`：对每个 item 执行的表达式；
* `item`：从 iterable(可迭代对象) 中获取的元素；
* `iterable`：要迭代的对象，例如列表、元组、字符串等；
* `condition（可选）`：可选过滤条件，只有满足条件的元素才会被包含在生成的集合中；

**（2）示例**

```python
# 创建一个包含 0 到 9 的平方的集合
squares_set = {x ** 2 for x in range(10)}
print(squares_set)  # 输出: {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# 去除列表中的重复元素：由于集合是无序且不重复的，集合推导式非常适合用于去除重复元素
numbers = [1, 2, 2, 3, 4, 4, 5]
unique_numbers = list({x for x in numbers})
print(unique_numbers)  # 输出：[1, 2, 3, 4, 5]

# 创建一个集合，包含 0 到 9 中的偶数的平方
even_squares_set = {x ** 2 for x in range(10) if x % 2 == 0}
print(even_squares_set)  # 输出：{0, 64, 4, 36, 16}
```

### 6.4 常见面试题

```python
# 问题 1：如何使用列表推导式生成一个包含从 1 到 10 的所有偶数的列表
q_list1 = [x for x in range(1, 11) if x % 2 == 0]
print(q_list1)  # [2, 4, 6, 8, 10]

# 问题 2：如何使用字典推导式生成一个字典，其中键是 1 到 5 的数字，值是这些数字的立方？
q_dict1 = {x: x ** 3 for x in range(1, 6)}
print(q_dict1)  # {1: 1, 2: 8, 3: 27, 4: 64, 5: 125}

# 问题 3：如何使用推导式生成一个二维列表（如 3x3 的矩阵），并请解释其中的每一步操作？
q_list2 = [[i * j for i in range(3)] for j in range(3)]  # 外层推导式负责生成矩阵的行，内层推导式生成行中的每个元素。
print(q_list2)  # [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# 问题 4：如何使用集合推导式生成一个包含从 1 到 10 的所有偶数的集合？
q_set1 = {x for x in range(1, 11) if x % 2 == 0}
print(q_set1)  # {2, 4, 6, 8, 10}

# 问题 5：如何使用集合推导式从一个列表中去除重复元素，并将每个元素的平方值存储在集合中？
nums = [1, 2, 2, 3, 4, 4, 5]
q_set2 = {x ** 2 for x in nums}
print(q_set2)  # {1, 4, 9, 16, 25}
```

## 7. Python 单下划线和双下划线

在 Python 中，单下划线和双下划线有特殊的含义，通常用于变量名和方法名的前后缀，以表达不同的用途和访问权限。理解这些用法对编写
Python 代码和理解他人代码都非常重要。

### 7.1  单下划线（“_”）使用

#### 7.1.1 前缀 "_var"

用于表示某个变量、方法或属性是内部使用的或具有特定含义，但不希望在外部直接访问（实际并不会真的限制访问）。

例如：

```python
class MyClass:
    def __init__(self):
        self._protected_var = 42  # 受保护的变量

    def _protected_method(self):
        # 私有的
        print("This is a protected method.")

    def public_method(self):
        return self._protected_method()


obj = MyClass()
print(obj._protected_var)  # 仍然可以访问，输出：42
obj._protected_method()  # 仍然可以调用，输出：This is a protected method.
obj.public_method()  # 输出：This is a protected method.
```

变量 “_protected_var” 和方法 “_protected_method” 都以单下划线开头，这是一种约定，告诉其他开发人员这些成员是类内部使用的，不建议在类外部直接访问，公共方法
“public_method” 可以访问内部方法 “_protected_method”。

若模块名或包名以单下划线开头表示这个模块不应该被外部直接导入，这是为了防止模块被 `from module import *` 语句导入。

例如：

```python
# 文件名为 _internal.py
def internal_function():
    pass


# 在其他文件中
from _internal import internal_function  # 不建议这样做
```

#### 7.1.2 后缀 "var_"

单下划线后缀通常用于避免与 Python 内置名称或关键字冲突。

```python
class_ = "Python Class"  # 避免与内置的 class 关键字冲突
```

#### 7.1.3 "_" 临时变量

在循环或解包过程中，单下划线 “_” 常用作临时变量，表示该变量的值将被忽略。

例如：

```python
for _ in range(3):
    print("Hello!")  # 输出三次 "Hello!"

x, _ = (1, 2)  # 忽略第二个值
```

#### 7.1.4 "_" 交互解释器

在 Python 交互式解释器中，单下划线 “_” 用于保存最后一次表达式的结果。

例如:

```text
>>> 5 + 3
8
>>> _
8
```

### 7.2 双下划线（“__”）使用

#### 7.2.1 前缀 "__var" (名称改写机制)

双下划线作为变量名或方法名前缀，表示该变量或方法是类内的私有成员（Private）。Python 会将其名称进行 “改写”（name
mangling），将其改为 "_ClassName__var" 的形式，以避免与子类中的名称冲突。

例如：

```python
class MyClass:
    def __init__(self):
        self.__private_var = 42

    def __private_method(self):
        print("This is a private method.")


obj = MyClass()

# print(obj.__private_var)  # AttributeError: 'MyClass' object has no attribute '__private_var'
# obj.__private_method()  # AttributeError: 'MyClass' object has no attribute '__private_method'

# 通过名称改写访问私有变量和方法
print(obj._MyClass__private_var)  # 输出: 42
obj._MyClass__private_method()  # 输出: This is a private method.
```

#### 7.2.2 前后缀 "\__var\__"（魔术方法/特殊方法）

双下划线前后缀用于定义 Python 的特殊方法或魔术方法（Magic Methods），这些方法通常由 Python
解释器调用，而不是用户直接调用，一般不应为普通变量或方法随意命名为这种形式。

例如：

* `__init__`：是构造函数；
* `__str__`：通过 str(object) 以及内置函数 format() 和 print() 调用以生成一个对象的 “非正式” 或格式良好的字符串表示。返回值必须是字符串对象。
* `__repr__`：是由 repr() 内置函数调用，用来输出一个对象的“官方”字符串表示。返回值必须是字符串对象，此方法通常被用于调试。内置类型
  object 所定义的默认实现会调用 object.__repr__()。

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"MyClass with value {self.value}"


obj = MyClass(10)
print(obj)  # 输出: MyClass with value 10
```

### 7.3 面试示例

* 问题 1：_var 和 __var 有什么区别？  
  `_var`：表示受保护的变量，这是约定俗成的表示，意味着该变量不应被外部直接访问，但实际上可以访问；  
  `__var`：会触发名称改写机制，将其改为 _ClassName__var，从而避免名称冲突，并使其成为类的私有变量；


* 问题 2：如何在 Python 中定义一个类的私有方法，并确保它不会与子类中的方法冲突？  
  使用双下划线前缀（如 __private_method）来定义类的私有方法，Python 会自动将其名称改写为 _
  ClassName__private_method，从而避免与子类中的方法冲突；


* 问题 3：\__init\__ 和 \__str\__ 的作用是什么？  
  `__init__`：是类的构造函数，用于初始化对象；  
  `__str__`：是用于定义对象的字符串表示形式，当使用 print 或 str() 调用对象时会被调用；

## 8. Python \"_\_str__" vs. \"_\_repr__"

在 Python 中，`__str__` 和 `__repr__` 是两个用于对象字符串表示的特殊方法（魔术方法），它们虽然都用于返回对象的字符串表示，但在语义和用途上有所不同。

### 8.1 \"_\_str__"

通过 str(object) 以及内置函数 format() 和 print() 调用以生成一个对象的 “非正式” 或格式良好的字符串表示，返回值必须是字符串对象。

示例：

```python
class MyClass:
    def __init__(self, name, value):
        self.name = name
        self.value = value

    def __str__(self):
        # 重写 __str__ 自定义输出
        return f"MyClass(name={self.name}, value={self.value})"


obj = MyClass("example", 42)
print(obj)  # 输出: MyClass(name=example, value=42)
```

### 8.2 \"_\_repr__"

由 repr() 内置函数调用，用来输出一个对象的 “官方” 字符串表示，返回值必须是字符串对象，此方法通常被用于调试，内置类型 object
所定义的默认实现会调用 `object.__repr__()`。

示例：

```python
class MyClass:
    def __init__(self, name, value):
        self.name = name
        self.value = value

    def __repr__(self):
        # 重写 __repr__ 自定义输出
        return f"MyClass({repr(self.name)}, {repr(self.value)})"


obj = MyClass("example", 42)
print(repr(obj))  # 输出: MyClass('example', 42)
```

### 8.3 "\_\_str__" vs. "\_\_repr__"

如上所述：

* `__repr__()`: 为开发者提供对象的正式字符串表示。另一重要特性是：开发者通常可以用它来重新创建一个与原始对象相同的对象；
* `__str__()`: 为用户提供对象的非正式字符串表示。即，任何用户都能理解对象中包含的数据，对用户来说更简单、更容易阅读；

示例：

```
>>> import datetime
>>> today = datetime.datetime.now()
>>> str(today)
'2024-08-20 14:34:13.765650'
>>> repr(today)
'datetime.datetime(2024, 8, 20, 14, 34, 13, 765650)'
>>> 
>>> 
>>> new_date = datetime.datetime(2024, 8, 20, 14, 34, 13, 765650)
>>> new_date == today
True
>>> new_date = 2024-08-20 14:34:13.765650
  File "<stdin>", line 1
    new_date = 2024-08-20 14:34:13.765650
                    ^
SyntaxError: leading zeros in decimal integer literals are not permitted; use an 0o prefix for octal integers
```

**总结:**

主要目的：

* `__str__`：返回用户友好的描述性字符串;
* `__repr__`：返回开发者友好的、精确的表示（通常是对象的“官方”表示，便于调试）;

默认行为：

* 如果没有定义 `__str__`，调用 str() 或 print() 时，会尝试调用 `__repr__`;
* 如果没有定义 `__repr__`，则使用默认的 `__repr__` 实现，返回类似 <__main__.MyClass object at 0x000001> 的字符串，显示对象的类型和内存地址;

何时实现：

* `__repr__` 更适合用于调试和开发，通常尽量实现一个能返回有效 Python 表达式的 `__repr__`;
* `__str__` 更适合用于展示给最终用户，可以实现为一个简洁、友好的字符串表示;

示例对比:

```python
class MyClass:
    def __init__(self, name, value):
        self.name = name
        self.value = value

    def __str__(self):
        return f"MyClass with name {self.name} and value {self.value}"

    def __repr__(self):
        return f"MyClass({repr(self.name)}, {repr(self.value)})"


obj = MyClass("example", 42)

print(str(obj))  # 输出: MyClass with name example and value 42
print(repr(obj))  # 输出: MyClass('example', 42)

# 在交互式解释器中直接输入对象名
# >>> obj
# MyClass('example', 42)
```

## 9. 字符串格式化 & 拼接

在 Python 中，字符串格式化 & 拼接有几种常用方式。

### 9.1 使用 "%" 操作符

这是最早的字符串格式化方法（Python 2.6 以前），通过 “%” 符号替代字符串中的占位符，常用占位符有 %s（字符串）、%d（整数）、%f（浮点数）等。

示例：

```python
name = "Alice"
age = 30
formatted_string = "Name: %s, Age: %d" % (name, age)
print(formatted_string)  # 输出: Name: Alice, Age: 30

names = ["Alice", "Paddy", "Joie"]
average_age = 30
formatted_string = "Names: %s" % names
print(formatted_string)  # 输出：Names: ['Alice', 'Paddy', 'Joie']

formatted_string = "Names: %s, Average Age: %d" % (names, age)
print(formatted_string)  # 输出：Names: ['Alice', 'Paddy', 'Joie'], Average Age: 30

names = ("Alice", "Paddy", "Joie")
# formatted_string = "Names: %s" % names
# print(formatted_string)  # 输出：TypeError: not all arguments converted during string formatting
formatted_string = "Names: %s" % (names,)
print(formatted_string)  # 输出：Names: ('Alice', 'Paddy', 'Joie')
```

**优点**：简单易用，适合处理简单的字符串格式化;  
**缺点**：不支持复杂的数据结构或格式化选项;

### 9.2 使用 "str.format()" 方法

"str.format()" 提供了更强大的格式化功能，可以通过位置或关键字参数指定占位符，并且支持丰富的格式化选项。

示例：

```python
name = "Alice"
age = 30
formatted_string = "Name: {}, Age: {}".format(name, age)
print(formatted_string)  # 输出: Name: Alice, Age: 30

names = ("Alice", "Paddy", "Joie")
print('{0} {2} {1} {2} {2} {2}'.format(*names))  # Alice Joie Paddy Joie Joie Joie

# 关键字参数
formatted_string = "Name: {name}, Age: {age}".format(age=age, name=name)
print(formatted_string)  # 输出: Name: Alice, Age: 30

# 格式化选项
pi = 3.14159
formatted_string = "Pi is approximately {:.2f}".format(pi)  # 保留两位小数
print(formatted_string)  # 输出: Pi is approximately 3.14

```

**优点**：功能强大，支持复杂的格式化和嵌套数据结构；  
**缺点**：语法较繁琐，格式字符串容易变得冗长；

### 9.3 使用 "f-strings" (Python 3.6+)

"f-strings" 是 Python 3.6 引入的一种格式化字符串的方式，语法简洁、直观，直接在字符串中嵌入变量和表达式。

示例：

```python
name = "Alice"
age = 30
formatted_string = f"Name: {name}, Age: {age}"
print(formatted_string)  # 输出: Name: Alice, Age: 30

# 嵌入表达式：
pi = 3.14159
formatted_string = f"Pi is approximately {pi:.2f}"  # 保留两位小数
print(formatted_string)  # 输出: Pi is approximately 3.14
```

**优点**：简洁、直观、性能高，可嵌入表达式，支持多行字符串;  
**缺点**：仅在 Python 3.6 及以上版本可用;

### 9.4 使用 "string.Template"

"string.Template" 提供了一种安全的字符串替换方法，特别适合从不可信的数据源构建字符串。

示例：

```python
from string import Template

template = Template("Name: $name, Age: $age")
formatted_string = template.substitute(name='Alice', age=30)
print(formatted_string)  # 输出: Name: Alice, Age: 30

# 安全替换：
template = Template("Name: $name, Age: $age")
formatted_string = template.safe_substitute(name="Alice")
print(formatted_string)  # 输出: Name: Alice, Age: $age (age 没有提供值)
```

**优点**：安全性高，适用于处理用户输入数据;  
**缺点**：功能有限，不支持复杂的表达式或高级格式化选项;

### 9.5 其他简单字符串拼接

* 通过 "+" 号的形式：

  ```python
  a, b = 'Hello', ' World!'
  c = a + b
  print(a + b)  # Hello World!
  print(c)  # Hello World!
  ```

* 通过 "," 号的形式：

  ```python
  a, b = 'Hello', ' World!'
  print(a, b)  # Hello  World!
  ```

* 直接连接：

  ```python
  print('Hello'         ' World!')  # Hello World!
  print('Hello'' World!')  # Hello World!
  ```
* 使用 “join” 内置方法：

  字符串有一个内置方法 "join"，其参数是一个序列类型，例如数组或者元组等。

  ```python
  print(' '.join(['Hello', 'World!']))  # Hello World!
  print(''.join(['Hello', 'World!']))  # HelloWorld!
  ```

* 使用 "*" 操作符：

  ```python
  a = 'Hello World!'
  print(a * 3)  # Hello World!Hello World!Hello World!
  ``` 

## 10. 可迭代对象、迭代器和生成器

理清 Python 中 "容器（container）"、“可迭代对象（iterable）”、“迭代器（iterator）”、“生成器（generator）” 概念及用法。其相互之间关系如下图所示：

![python_features10_1.png](..%2F_static%2Fpython_features10_1.png)

### 10.1 容器 (container)

容器是用来存储和管理多个元素的对象，可以存放任何数据类型的元素，包括其他容器，可以用 “in”, “not in”
关键字判断元素是否包含在容器中。通常这类数据结构把所有的元素存储在内存中（也有一些特例，并不是所有的元素都放在内存，比如迭代器和生成器对象）在
Python 中，常见的容器对象有：

1. 列表 (List)：有序且可变，允许重复元素；`my_list = [1, 2, 3, 4]`
2. 元组 (Tuple)：有序且不可变，允许重复元素；`my_tuple = (1, 2, 3, 4)`
3. 字典 (Dictionary)：无序的键值对集合，键必须唯一且不可变，值可以是任何类型；`my_dict = {'a': 1, 'b': 2}`
4. 集合 (Set)：无序且不重复的元素集合，支持数学集合操作；`my_set = {1, 2, 3, 4}`
5. 字符串 (String)：不可变的字符序列；`my_string = "Hello"`

尽管绝大多数容器都提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是可迭代对象赋予了容器这种能力，当然并不是所有的容器都是可迭代的。

### 10.2 可迭代对象 (iterable)

可迭代对象是实现了 `__iter__()` 或 `__getitem__()` 方法的对象，返回一个迭代器对象供遍历。

**常见的可迭代对象：**

* 内置容器类型：如 列表、元组、字典、集合、字符串等；
* 文件对象：文件读取操作返回的对象也是可迭代的；

示例：

```python
import os
from collections.abc import Iterable, Iterator

# 判断是否是可迭代对象
print(isinstance([], Iterable))  # 输出: True 列表是可迭代的
print(isinstance({}, Iterable))  # 输出: True 字典是可迭代的
print(isinstance((), Iterable))  # 输出: True 元组是可迭代的
print(isinstance(set(), Iterable))  # 输出: True 集合是可迭代的
print(isinstance('', Iterable))  # 输出: True  字符串是可迭代的

currPath = os.path.dirname(os.path.abspath(__file__))
with open(currPath + '/demo.py') as file:
    print(isinstance(file, Iterable))  # True File 是可迭代的

# 判断是否是可迭代器
print(isinstance([], Iterator))  # 输出: False 列表不是迭代器
print(isinstance({}, Iterator))  # 输出: False 字典不是迭代器
print(isinstance((), Iterator))  # 输出: False 元组不是迭代器
print(isinstance(set(), Iterator))  # 输出: False 集合不是迭代器
print(isinstance('', Iterator))  # 输出: False  字符串不是迭代器

currPath = os.path.dirname(os.path.abspath(__file__))
with open(currPath + '/demo.py') as file:
    print(isinstance(file, Iterator))  # True File 是迭代器
```

**`__iter__()` / `__getitem__()` 方法：**

* `__iter__()` 方法：
  * 可迭代对象的核心方法之一；
  * 通过实现 `__iter__()` 方法，一个对象可以成为可迭代对象，返回一个迭代器；
  * `__iter__()` 必须返回一个带有 `__next__()` 方法的对象，例，在一个类中实现 `__iter__()` 方法，通常会返回
    self，并在该对象上定义 `__next__()` 方法；
  * 示例：

    ```python
    class MyIterable:
        def __iter__(self):
            self.current = 0
            return self

        def __next__(self):
            if self.current < 3:
                self.current += 1
                return self.current
            else:
                raise StopIteration

    my_iter = iter(MyIterable())  # iter() Python 内置函数，手动调用其 __iter__() 方法并返回一个迭代器，见下文解释
    print(next(my_iter))  # 输出: 1  next() Python 内置函数，手动调用其 __next__() 获取迭代器的下一个元素，见下文解释
    
    for i in my_iter:
        print(i)  # 输出: 1 2 3
    `````
  * for 循环遍历可迭代对象的原理：
    * 调用 `__iter__()` 方法：
      * Python 首先调用可迭代对象的 `__iter__()` 方法，这个方法返回一个迭代器对象；
      * 迭代器是一个实现了 `__next__()` 方法的对象，用于逐步返回下一个元素;
    * 获取迭代器：
      * `__iter__()` 返回的迭代器会用于遍历该可迭代对象;
    * 调用 `__next__()` 方法：
      * for 循环内部会自动调用迭代器的 `__next__()` 方法来获取下一个元素，并将其赋值给循环变量;
      * 每次调用 `__next__()` 方法时，迭代器返回一个新的元素;
    * 处理 StopIteration 异常：
      * 当迭代器的元素被遍历完后，`__next__()` 方法会抛出 StopIteration 异常，表示迭代结束;
      * for 循环会捕获这个异常，并终止循环;
      * 例如：
        ```
        x = [1, 2, 3]
        for elem in x:
        …
        ```
        原理如图所示：

        ![python_features10_2.png](..%2F_static%2Fpython_features10_2.png)

  * iter() 函数（`__iter__()` 是可迭代对象实现的内部方法，而 `iter()` 是获取迭代器的外部函数）:
    * 内置函数，用于从可迭代对象或迭代器获取迭代器，是手动调用 `__iter__()` 方法的一种方式；
    * 对于可迭代对象，iter(obj) 会调用其 `__iter__()` 方法并返回一个迭代器；
    * 如果对象没有实现 `__iter__()` 但实现了 `__getitem__()`，iter() 也可以返回一个迭代器，使用索引递增方式进行迭代；
    * 示例：
      ```
      >>> x = [1, 2, 3]
      >>> y = iter(x)
      >>> type(x)
      <class 'list'>
      >>> type(y)
      <class 'list_iterator'>
      ```
  * next() 函数：
    * Python 的内置函数，用于获取迭代器的下一个元素，是手动调用 `__next__()` 方法的一种方式；
    * 它内部会调用迭代器对象的 `__next__()` 方法；
    * 语法：next(iterator[, default])。default 是可选参数，当迭代器耗尽时返回该值，而不是抛出 StopIteration 异常；
    * 示例：
      ```
      >>> iterator = iter([1, 2, 3])
      >>> print(next(iterator))
      1
      >>> print(iterator.__next__())
      2
      ```
* `__getitem__()` 方法：
  * 如果对象实现了 `__getitem__()` 方法，它也可以被迭代；
  * `__getitem__()` 通常用于索引访问（如 obj[index]），当实现此方法时，Python 的迭代机制可以通过递增索引来迭代对象；
  * 示例：

    ```python
    class MySequence:
        def __getitem__(self, index):
            if index < 3:
                return index * 2
            else:
                raise IndexError("Index out of range")
    
    my_seq = MySequence()
    print(my_seq[2])  # 输出: 4
    # print(my_seq[3])  # 输出: IndexError: Index out of range
    for item in my_seq:
        print(item)  # 输出: 0, 2, 4
    ```

**小结：**

* `__iter__()` 是更通用的迭代方法，适用于实现迭代器的对象;
* `__getitem__()` 适用于基于索引的访问，在没有 `__iter__()` 时可以作为迭代机制的替代方法;
  > **备注:**  
  > 一个对象实现了 `__getitem__()` 方法，它就可以被视为可迭代对象，尽管它没有实现 `__iter__()` 方法。
  > 在这种情况下，Python 会通过从索引 0 开始递增地调用 `__getitem__()` 来模拟迭代，直到 `__getitem__()` 抛出 IndexError
  异常为止。
  > 但通过 `__iter__()` 方法是更标准的做法。如果仅实现 `__getitem__()`，该对象仍然是可迭代的，但通常这会用于类似于序列（如列表和元组）的对象。

### 10.3 迭代器 (iterator)

#### 10.3.1 介绍

如上描述，Python 迭代器是实现了 `__iter__()` 和 `__next__()` 方法的对象。

特征：

* `__iter__()` 返回迭代器对象本身；
* `__next__()` 返回序列中的下一个值，如果没有值，则抛出 StopIteration 异常；
* 惰性求值：延迟计算，节省内存。即，每次调用 `__next__()` 时才生成下一个元素，节省内存；
* 只能向前遍历，不能回溯，一次性使用，遍历后需重新创建；
* 适用于需要逐个访问元素且无需保存整个序列的场景，如遍历大型文件、数据流等；

性能：

* 迭代器在处理大数据集时具有明显优势，因为它按需生成元素，不需要将所有数据加载到内存中；
* 在处理流数据或处理无界数据集时，迭代器避免了一次性计算所有元素，减少了内存使用，提高了性能；

* 示例：自定义迭代器：

  ```python
  class Counter:
      def __init__(self, low, high):
          self.current = low
          self.high = high
  
      def __iter__(self):
          return self
  
      def __next__(self):
          if self.current > self.high:
              raise StopIteration
          else:
              self.current += 1
              return self.current - 1
  
  
  counter = Counter(1, 5)
  for num in counter:
      print(num)  # 输出: 1, 2, 3, 4, 5
  ```

* 处理大型日志文件：当处理非常大的日志文件时，迭代器逐行读取文件，避免将整个文件加载到内存中

  ```python
  class LogFileIterator:
      def __init__(self, filepath):
          self.file = open(filepath)
  
      def __iter__(self):
          return self
  
      def __next__(self):
          line = self.file.readline()
          if not line:
              self.file.close()
              raise StopIteration
          return line.strip()
  
  log_iterator = LogFileIterator("server.log")
  for log in log_iterator:
      print(log)
  ```

#### 10.3.2 高级用法

* 链式迭代：通过 itertools.chain 实现多个可迭代对象的无缝迭代
  ```python
  from itertools import chain

  for item in chain([1, 2], ['a', 'b']):
      print(item)  # 输出: 1, 2, a, b
  ```

* 无限迭代：itertools.cycle 用于无限循环迭代可迭代对象

  ```python
  from itertools import cycle
  counter = cycle([1, 2, 3]) # 创建一个无限循环迭代器
  for _ in range(5):
      print(next(counter))  # 输出: 1, 2, 3, 1, 2
  ```

  代码解释：  
  使用 itertools.cycle 创建一个无限循环的迭代器，cycle() 会重复遍历提供的可迭代对象，这里是列表 [1, 2, 3]，在循环中，每次调用
  next(counter) 都会返回下一个元素，循环到末尾后重新从头开始，由于 for 循环次数设置为 5，输出为 1, 2, 3, 1, 2。

  ```python
  from itertools import cycle
  from itertools import islice
  
  counter = cycle([1, 2, 3])
  limited = islice(counter, 0, 4)
  for x in limited:
      print(x) # 输出: 1, 2, 3, 1
  ```
  代码解释：
  * cycle([1, 2, 3]) 是一个迭代器，它会无限循环遍历 [1, 2, 3]；
  * islice(counter, 0, 4) 用于从 counter 中切片，返回从位置 0 开始的 4 个元素；

  由于 counter 是一个无限循环的迭代器，islice 限制了输出的数量，因此输出为 1, 2, 3, 1。循环首先遍历 [1, 2, 3]，然后再次从 1
  开始，由于限制是 4 个元素，最后返回 [1, 2, 3, 1]

#### 10.3.3 可迭代对象与迭代器区别

**可迭代对象 (Iterable)：**

* 定义：实现了 `__iter__()` 或 `__getitem__()` 方法的对象，可以返回一个迭代器（通常 `__iter__()` 会返回
  self（亦可返回一个其他迭代器），并在该对象上定义 `__next__()` 方法，即也是迭代器）；
* 范围：包括所有集合类型，如列表、元组、字典、集合和字符串。通过 for 循环可以遍历它们；
* 使用：当你想要对元素进行遍历时，使用 for 循环或者调用 iter()；

**迭代器 (Iterator)：**

* 定义：实现了 `__iter__()` 和 `__next__()` 方法的对象，提供了逐个访问元素的机制；
* 范围：迭代器可以从可迭代对象中通过 iter() 函数获得，也可以是自定义的类；
* 使用：每次调用 next() 方法时，迭代器返回下一个元素，当元素遍历完时，抛出 StopIteration 异常；

**两者区别：**

* 范围：所有迭代器都是可迭代对象（迭代器必须同时实现 `__iter__()` 方法和 `__next__()`
  方法），但并非所有可迭代对象都是迭代器。可迭代对象通过 `__iter__()` 方法生成迭代器；
* 一次性使用：迭代器只能前进，且只能遍历一次；而可迭代对象可以多次生成迭代器进行遍历；
* 实现：迭代器比可迭代对象多了 `__next__()` 方法，用于逐个返回元素；

**使用场景：**

* 可迭代对象：适用于需要多次遍历数据的场景，如列表、字典等；
* 迭代器：适用于需要懒加载数据、流式处理大数据或实现自定义迭代行为的场景；

### 10.4 生成器 (generator)

生成器 (generator) 是一种特殊的迭代器，但反之不一定成立，其通过 yield 关键字在函数中实现，可以逐个返回元素，而不需要一次性生成所有数据。

**定义：**

* 生成器是使用 yield 关键字定义的特殊迭代器函数，每次调用 yield 时返回一个值，并暂停函数执行；
* 生成器返回的是一个迭代器，可以逐个访问生成的元素；

**原理：**

* 每次调用生成器函数时，执行到 yield 停止，并返回一个值，下次从此处继续执行；
* 生成器在运行时动态生成数据，而非预先生成所有数据；

**yield 关键字：**

* yield 是生成器函数的核心，它暂停函数执行并返回一个值，下次函数从暂停的地方继续执行；
* 每次调用 yield，函数状态（包括局部变量）都会被保存，允许函数在下次调用时继续从中断的地方继续执行；
* yield 与 return 的区别：（1）yield 可以多次返回值，并保持函数状态；（2）return 会终止函数并返回单一值，函数状态不会保留；

**生成器创建方法：**

生产器的创建方法有两种：一是生成器表达式；二是生成器函数 (yield) 关键字。

* 生成器表达式
  生成器表达式类似列表推导式，但使用圆括号 "()" 而非方括号 "[]"。

  ```python
  from collections.abc import Generator

  gen = (x * x for x in range(10))
  print(isinstance(gen, Generator))  # 输出：True 是生成器
  print(type(gen))  # 输出：<class 'generator'>
  # print(list(gen))  # 输出：[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  print(next(gen))  # 输出：0
  for ℹ in gen:
      print(i)  # 输出：1 4 9 16 25 36 49 64 81
  ```

* 生成器函数

  如果一个函数定义中包含 yield 关键字，那么这个函数就不再是一个普通的函数，而是一个 generator。

  示例1： 倒序打印

  ```
  def countdown(n):
      while n > 0:
          yield n
          n -= 1

  for num in countdown(5):
      print(num)  # 输出: 5, 4, 3, 2, 1
  ```

  示例2： 斐波那契数列，生成器可以用于生成无限序列，如斐波那契数列，避免一次性生成所有数据

  ```python
  def fib():
      prev, curr = 0, 1
      while True:
          yield curr
          prev, curr = curr, curr + prev


  # 创建一个斐波那契数列生成器对象
  f = fib()

  # 打印前10个斐波那契数列元素
  for _ in range(10):
      print(next(f))  # 输出：1 1 2 3 5 8 13 21 34 55
  ```

  当执行 f=fib() 时返回的是一个生成器对象，此时函数体中的代码不会执行，只有显示或隐式调用 next() 的时候才会真正执行里面的代码。
  在每次调用 next() 的时候执行，遇到 yield 语句返回，再次执行时，从上次返回的 yield 语句处继续执行。

  示例3（高级）：生成器委托，使用 yield from 委托子生成器，简化嵌套生成器的调用
  ```python
  def sub_generator():
      yield 1
      yield 5

  def main_generator():
      yield from sub_generator()
      yield 3
  
  for item in main_generator():
      print(item)  # 输出: 1, 5, 3
  ```

**性能：**

* 生成器与迭代器一样，通过延迟计算和按需生成数据，节省了内存并提高了性能；
* 生成器的性能优势在于处理大型或无限数据集时的内存效率，特别是在流式数据处理或大数据分析中非常有用；
* 生成器在处理复杂逻辑时还能保持代码简洁，提高可读性；

### 10.5 面试示例

* 如何判断一个对象是否是可迭代对象？
  使用 isinstance(obj, Iterable)，Iterable 在 collections.abc 模块中；

* 什么是迭代器协议？
  迭代器协议包括 `__iter__()` 和 `__next__()` 方法；

* 生成器与迭代器的区别是什么？
  生成器是迭代器的一种，它通过 yield 关键字实现，并且能够在执行中暂停和恢复。

* 生成器在什么情况下比列表更有优势？
  当处理大数据集时，生成器可以节省内存，因为它们不会将所有元素一次性加载到内存中，而是按需生成；

* 将列表生成式中 [] 改成 () 之后数据结构是否改变？
  是，从列表变为生成器。
  ```
  >>> L = [x*x for x in range(10)]
  >>> L
  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  >>> g = (x*x for x in range(10))
  >>> g
  <generator object <genexpr> at 0x7fad1f8cc520>
  ```

  通过列表生成式，可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且，创建一个包含百万元素的列表，不仅是占用很大的内存空间，如：我们只需要访问前面的几个元素，后面大部分元素所占的空间都是浪费的。因此，没有必要创建完整的列表（节省大量内存空间）。在Python中，我们可以采用生成器：边循环，边计算的机制 —>
  生成器 (generator)

## 11. *args and **kwargs

在 Python 中，`*args` 和 `**kwargs` 是两种用于函数定义时处理可变数量参数的机制，分别用于处理位置参数（*args）和关键字参数（**
kwargs）。这两个机制提高了函数的灵活性和可重用性。

* 位置参数：按位置顺序传递给函数的参数，必须按顺序提供，调用时参数的顺序决定其赋值；
  ```python
  def greet(name: str, age: int):
      print(f"Name: {name}, Age: {age}")
  
  
  greet("Alice", 30)  # 输出: Name: Alice, Age: 30
  ```
* 关键字参数：在调用函数时，通过名称（键）传递的参数，可以任意顺序传递，调用时明确指定每个参数的名称。
  ```python
  def greet(name: str, age: int = 10):
    print(f"Name: {name}, Age: {age}")

  greet(age=30, name="Alice")  # 输出: Name: Alice, Age: 30
  ```

两者区别：  
（1）位置参数依赖顺序，关键字参数依赖名称；  
（2）使用关键字参数可以提高代码的可读性和灵活性，尤其是在参数较多或顺序不固定时；

`*args` 和 `**kwargs` 允许将多个不确定参数或关键字参数传递给函数。

### 11.1 *args

`*args` 用于函数中接收任意数量的位置参数，返回一个元组。在函数需要处理不确定数量的输入时使用。

示例：

```python
def add_numbers(*args):
    print(type(args))  # 输出：<class 'tuple'>
    print(args[0])  # 输出：1
    args[0] = 9  # 输出：TypeError: 'tuple' object does not support item assignment
    return sum(args)


# 传递三个不同的位置参数
print(add_numbers(1, 2, 3))  # 输出: 6
nums = [1, 2, 3]
# * 解包可迭代对象
print(add_numbers(*nums))  # 输出: 6

# 解包列表并将参数传递给函数时，就好像单独传递了每个参数一样。这意味着可以使用多个解包运算符从多个列表中获取值，并将它们全部传递给单个函数
nums1 = [1, 2, 3]
nums2 = [4, 5, 6]
nums3 = [7, 8, 9]
print(add_numbers(*nums1, *nums2, *nums3))  # 输出: 45
```

代码解释：“add_numbers” 函数使用了 *args 参数来接收任意数量的位置参数，并将它们相加返回总和。可以向函数传递任意数量的参数，不限制参数的个数。

注意：

* 调用函数 "\*nums" ：这里 “*” 表示为解包运算符，用于解包列表、元组等可迭代对象，将其元素传递给函数或构造新的列表、元组等；
* 函数参数 “\*args”：在函数参数定义中使用 “*” 将多个位置参数打包成一个元组 (tuple)，元组是一个不可变的对象，其值在赋值后不能更改；
* “\*args”："args" 只是一个名称，可以选择其他名称（符合命名规范）， 但 “*” 必须，例如：“*argsintegers”；

### 11.2 **kwargs

`**kwargs` 用于函数中接收任意数量的关键字参数，返回一个字典。在函数需要处理不确定数量的命名参数时使用。

```python
def greet(**kwargs):
    print(type(kwargs))  # 输出：<class 'dict'>
    for key, value in kwargs.items():
        print(f"{key}: {value}")


greet(name="Alice", age=30)  # 输出: name: Alice, age: 30
user = {"name": "Alice", "age": 30, "address": "china"}
# ** 解包字典，将键值对作为关键字参数传递给函数
greet(**user)  # 输出: name: Alice, age: 30 address: china
# 同上述 *args 同理
info = {"phone": "****", "email": "email@xxx.com"}
greet(**user, **info)  # 输出: name: Alice, age: 30 address: china phone: **** email: email@xxx.com
```

代码解释：“greet” 函数使用了 "**kwargs" 参数来接收任意数量的关键字参数，并将它们打印输出。可以向函数传递任意数量的参数，不限制参数的个数。

注意：

* 调用函数 "\**user" ：解包字典，“**” 运算符用于解包字典，将键值对作为关键字参数传递给函数；
* 函数参数 “\**kwargs”：在函数参数定义中使用 “**”，将多个关键字参数打包成一个字典 (dict)；
* “\**kwargs”：”kwargs“ 只是一个名称，可以选择其他名称（符合命名规范），但 “**”（星号）必须，例如：“**words”；

### 11.3 混合使用

```python
def func(*args, **kwargs):
    print("args:", args)
    print("kwargs:", kwargs)


func(1, 2, 3, name="Alice", age=30)
# 输出:
# args: (1, 2, 3)
# kwargs: {'name': 'Alice', 'age': 30}
```

* "\*args" 只能传递位置参数，而 "**kwargs" 只能传递关键字参数；
* 函数定义中，“*args” 必须在 “**kwargs” 之前；

### 11.4 函数参数顺序

在 Python 3 中，函数参数的顺序有固定规则，必须遵循以下顺序：

* 位置参数（Positional Arguments）：函数调用时按顺序提供的参数
  ```python
  def func(a, b):
      return a + b
  print(func(1, 2))  # 输出: 3
  ```

* 默认参数（Default Arguments）：为参数提供默认值

  ```python
  def func(a, b=2):
      return a + b
  print(func(1))  # 输出: 3
  ```

* 可变位置参数（*args）：将多个位置参数打包为元组
  ```python
  def func(a, b=2, *args):
      return sum(args)
  print(func(1, 2, 3, 4))  # 输出(a:1,b:2,args(3,4) 相加): 7
  ```

* 关键字参数（Keyword Arguments）：使用 key=value 形式的参数

  ```python
  def func(a, b=2, *args, c):
      return a + b + c
  print(func(1, c=3))  # 输出: 6
  ```

* 可变关键字参数（**kwargs）：将多个关键字参数打包为字典
  ```python
  def func(a, b=2, *args, c, **kwargs):
      return kwargs
  print(func(1, 2, 4, 5, c=10, name="Paddy", age=12))  # 输出: {'name': 'Paddy', 'age': 12}
  ```

**总结**：在定义函数时，参数的顺序依次为：位置参数、默认参数、*args、关键字参数、**kwargs。

### 11.5 "*" 和 "**" 用法

* `“*”` 用法  
  **位置参数打包**：在函数定义中，“*args” 用于接收任意数量的位置参数，打包为一个元组；

  ```python
  def func(*args):
      print(args)
  
  func(1, 2, 3)  # 输出: (1, 2, 3)
  ```

  **解包可迭代对象**：在函数调用中，* 用于将列表、元组等可迭代对象解包为单独的参数；
  ```python
  def add(a, b, c):
      return a + b + c
  
  numbers = [1, 2, 3]
  print(add(*numbers))  # 输出: 6
  ```

* `“**”` 用法

  **关键字参数打包**：在函数定义中，**kwargs 用于接收任意数量的关键字参数，打包为一个字典;
  ```python
  def func(**kwargs):
      print(kwargs)
  
  func(a=1, b=2)  # 输出: {'a': 1, 'b': 2}
  ```

  **解包字典**：在函数调用中，** 用于将字典解包为关键字参数;
  ```python
  def greet(name, age):
    print(f"Name: {name}, Age: {age}")

  info = {"name": "Alice", "age": 30}
  greet(**info)  # 输出: Name: Alice, Age: 30
  ```

* 示例
  * 使用运算符解压缩列表并将参数传递给函数时，就好像单独传递了每个参数一样。这意味着可以使用多个解包运算符从多个列表中获取值，并将它们全部传递给单个函数：
  ```python
  def my_sum(*args):
      result = 0
      for x in args:
          result += x
      return result
    
  list1 = [1, 2, 3]
  list2 = [4, 5]
  list3 = [6, 7, 8, 9]
    
  print(my_sum(*list1, *list2, *list3)) # 输出：45
  ```
  * 列表拆分为三个不同的部分：显示第一个值、最后一个值以及介于两者之间的所有值

  ```python
  my_list = [1, 2, 3, 4, 5, 6]
  a, *b, c = my_list
  print(a)  # 输出：1
  print(b)  # 输出：[2, 3, 4, 5]
  print(c)  # 输出：6
  ```
  * 解包运算符可以拆分任何可迭代对象的项：合并两个列表

  ```python
  my_first_list = [1, 2, 3]
  my_second_list = [4, 5, 6]
  my_merged_list = [*my_first_list, *my_second_list]
  
  print(my_merged_list) # 输出：[1, 2, 3, 4, 5, 6]
  ```
  * 解包运算符可以拆分任何可迭代对象的项：合并两个字典

  ```python
  my_first_dict = {"A": 1, "B": 2}
  my_second_dict = {"C": 3, "D": 4}
  my_merged_dict = {**my_first_dict, **my_second_dict}
  
  print(my_merged_dict)  # 输出：{'A': 1, 'B': 2, 'C': 3, 'D': 4}
  ```
  * 解包字符串

  ```python
  a = [*"HelloWorld!"]
  print(a)  # 输出：['H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd', '!']
  # 或者：
  *a, = "HelloWorld!"
  print(a)  # 输出：['H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd', '!']
  # 或者：
  *a, b = "HelloWorld!"
  print(a)  # 输出：['H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd']  type(a) => <class 'list'>
  print(b)  # 输出：！ type(b) => <class 'str'>
  ```

**注意：** 使用解包运算符需要考虑代码可读性。

## 12. 面向切面编程 (AOP) 和装饰器

### 12.1 面向切面编程 (AOP)

面向切面编程（AOP）是一种编程范式，旨在将横切关注点（如日志记录、安全检查、事务管理等）与核心业务逻辑分离，它允许在不修改代码的情况下将这些行为
“切入” 到程序的执行流程中，从而提高代码的模块化程度。在 Python 中，AOP 主要通过装饰器和元类来实现。

* 使用装饰器实现 AOP
  装饰器是 Python 实现 AOP 的主要工具。通过装饰器，可以在函数执行前后添加额外的行为，如日志记录、性能监控等。

  ```python
  def log_execution(func):
      def wrapper(*args, **kwargs):
          print(f"Executing {func.__name__}...")
          result = func(*args, **kwargs)
          print(f"Finished executing {func.__name__}.")
          return result
  
      return wrapper
  
  @log_execution
  def process_data():
      print("Processing data...")
  
  process_data()
  ```

  输出：

  ```
  Executing process_data...
  Processing data...
  Finished executing process_data.
  ```

  在这个示例中，"log_execution" 装饰器为 "process_data" 函数添加了执行日志的功能。

* 使用元类实现 AOP

  元类（metaclass）可以用于更复杂的 AOP 实现，尤其是在需要控制类的创建或修改类的行为时。

  ```python
  def log_execution(func):
      def wrapper(*args, **kwargs):
          print(f"Executing {func.__name__}...")
          result = func(*args, **kwargs)
          print(f"Finished executing {func.__name__}.")
          return result
  
      return wrapper
  
  class LoggingMeta(type):
      """
      元类：可以在类创建之前修改类的属性和方法，甚至可以改变类的继承结构；
      目地：在所有方法前后添加日志输出
      """
  
      def __new__(cls, name, bases, dct):
          """
          在类创建之前调用，用于控制类的创建过程
          :param name: 类的名称
          :param bases: 类的基类
          :param dct: 类的属性和方法字典
          """
          for key, value in dct.items():
              if callable(value):  # Python callable () 函数，用于检查对象是否可以像函数一样被调用
                  dct[key] = log_execution(value)  # 可调用对象前后增加日志输出逻辑
          return super().__new__(cls, name, bases, dct)
  
  class MyClass(metaclass=LoggingMeta):
      def method1(self):
          print("method1 execution")
  
      def method2(self):
          print("method2 execution")
  
  obj = MyClass()
  obj.method1()
  obj.method2()
  
  ```

  输出：
  ```
  Executing method1...
  method1 execution
  Finished executing method1.
  
  Executing method2...
  method2 execution
  Finished executing method2.
  ```

  在这个例子中，LoggingMeta 元类自动为 MyClass 中的所有方法添加日志功能。

**应用场景**

* 日志记录：在函数或方法执行时自动记录日志;
* 权限控制：在方法调用前自动检查用户权限;
* 事务管理：在业务操作前后自动管理数据库事务;

**AOP 的优点**

* 提高模块化：分离关注点，减少代码重复;
* 易于维护：通过集中管理跨领域逻辑，简化代码维护;
* 通过 AOP，开发者可以将日志、权限等横切关注点从业务逻辑中解耦，使代码更加清晰、可维护;

### 12.2 装饰器

装饰器是一种函数或类，用于在不改变目标函数或方法的基础上，动态地添加功能。它在 AOP
中经常被用作实现横切关注点的工具。通过在函数或方法前加上 "@decorator_name" 应用装饰器。

**原理：**

在 Python 中，当你为函数添加装饰器（通过 @decorator 语法）时，Python 解释器会自动将装饰器函数应用到目标函数上。这是 Python
的一个语言特性，在函数定义时就已经生效。

Python 解释器的处理逻辑：

* 解析函数定义：当 Python 解释器遇到一个使用 “@decorator” 语法的函数定义时，它会立即应用装饰器，而不是等到函数被调用时才应用；
* 调用装饰器函数：装饰器函数会在函数定义的过程中被调用，它接收被装饰的函数作为参数，并返回一个新的函数或可调用对象；
* 替换原函数：返回的函数或对象会取代原始函数，这意味着当你调用 “say_hello()” 时，实际上调用的是装饰器返回的包装函数，而不是原函数本身；

如：

```python
@decorator
def say_hello():
    print("Hello!")


# 等效于:
say_hello = decorator(say_hello)
```

在 Python 源码中，这种处理是在函数定义阶段完成的。具体来说，Python 在解释函数定义时会检查是否有 @
标记，如果有，解释器就会调用相应的装饰器函数，并将结果绑定到函数名上。

**示例：**

* 示例1：基本用法

  ```python
  def my_decorator(func):
      def wrapper(*args, **kwargs):
          print("Something before the function")
          result = func(*args, **kwargs)
          print("Something after the function")
          return result
  
      return wrapper
  
  @my_decorator
  def say_hello():
      print("Hello!")
  
  say_hello()
  ```

  输出：

  ``` 
  Something before the function
  Hello!
  Something after the function
  ```

  此例中，"my_decorator" 装饰器在 "say_hello" 函数前后添加了日志信息。

* 示例2：带参数的装饰器

  装饰器本身也可以接受参数。这种情况下，装饰器会返回另一个装饰器。
  ```python
  def repeat(times):
      # repeat 是一个工厂函数，它返回一个装饰器
      def decorator(func):
          # decorator 是实际的装饰器函数，它接受一个函数作为参数
          def wrapper(*args, **kwargs):
              #  wrapper 是装饰器内部的包装函数
              for _ in range(times):
                  # 使用一个循环，来调用被装饰的函数，循环的次数由 `times` 决定
                  func(*args, **kwargs)
  
          return wrapper
  
      return decorator
  
  
  @repeat(times=3)
  def say_hello():
      print("Hello!")
  
  
  say_hello()
  ```

  输出：

  ```
  Hello!
  Hello!
  Hello!
  ```

  在这个例子中，“@repeat(times=3)” 是一个带参数的装饰器，其实现原理如下：

  * 调用外层函数 repeat(times)：
    * 当解释器遇到 @repeat(times=3) 时，它首先调用 repeat(3)，此时 times 被设定为 3；
    * repeat 函数返回一个装饰器函数 decorator；
  * 调用中间层函数 decorator(func)：
    * decorator 接收被装饰的函数 say_hello 作为参数，并返回一个包装函数 wrapper；
  * 调用内层函数 wrapper(*args, **kwargs)：
    * 每次调用 say_hello("Hello!") 时，实际上调用的是 wrapper 函数；
    * wrapper 函数内部执行了一个循环，将 say_hello("Hello!") 调用 3 次；
  * 输出结果：
    * 最终，say_hello("Hello!") 会输出 3 次 "Hello!"；

* 示例3：类装饰器

  ```python
  class MyDecorator:
      def __init__(self, func):
          self.func = func
  
      def __call__(self, *args, **kwargs):
          """
          当定义 “__call__” 方法，类的实例（对象）可以像普通函数那样被调用，例 MyDecorator() 即可调用 __call__()
          :param args:
          :param kwargs:
          :return:
          """
          print("Before function execution")
          result = self.func(*args, **kwargs)
          print("After function execution")
          return result
  
  
  @MyDecorator
  def say_hello():
      print("Hello!")
  ```

  输出：

  ```
  Before function execution
  Hello!
  After function execution
  ```

* 示例4：嵌套装饰器 (Multiple Decorators)

  嵌套装饰器指的是在一个函数上同时应用多个装饰器。多个装饰器会从内到外依次应用，这意味着最内层的装饰器最后执行。

  ```python
  def decorator_one(func):
      def wrapper(*args, **kwargs):
          print("Decorator One")
          return func(*args, **kwargs)
  
      return wrapper
  
  
  def decorator_two(func):
      def wrapper(*args, **kwargs):
          print("Decorator Two")
          return func(*args, **kwargs)
  
      return wrapper
  
  
  @decorator_one
  @decorator_two
  def say_hello():
      print("Hello!")
  
  
  say_hello()
  ```

  输出：

  ```
  Decorator One
  Decorator Two
  Hello!
  ```

  "decorator_two" 装饰器先作用于 “say_hello”，然后 “decorator_one” 装饰器再作用于被 “decorator_two”
  装饰的函数，装饰器执行则从最外往里推进执行："decorator_one" -> "decorator_two"；

* 示例5：装饰器用于类实例方法、类方法和静态方法

  ```python
  def timing_decorator(func):
      import time
      def wrapper(*args, **kwargs):
          start_time = time.time()
          result = func(*args, **kwargs)
          end_time = time.time()
          print(f"{func.__name__} took {end_time - start_time} seconds")
          return result
  
      return wrapper
  
  
  class MyClass:
      @timing_decorator
      def instance_method(self):
          print("Instance method")
  
      @classmethod
      @timing_decorator
      def class_method(cls):
          print("Class method")
  
      @staticmethod
      @timing_decorator
      def static_method():
          print("Static method")
  
  
  obj = MyClass()
  obj.instance_method()
  obj.class_method()
  obj.static_method()
  ```

  输出：

  ```
  Instance method
  instance_method took 2.384185791015625e-06 seconds
  Class method
  class_method took 1.9073486328125e-06 seconds
  Static method
  static_method took 1.430511474609375e-06 seconds
  ```

  装饰器不仅能装饰普通函数，还可以用于类实例方法、类方法和静态方法，装饰器 “timing_decorator” 记录了函数执行所花费的时间。

**总结：**  装饰器的实现原理基于闭包（closure 后面会详细介绍）将额外功能嵌入到原函数的调用过程中，利用函数的可传递性和可返回性（first-class
functions 函数作为一等公民），实现代码复用和逻辑的分离。

## 13. 闭包 (closure)

闭包和函数紧密联系在一起，介绍闭包前先介绍下：变量的作用域、嵌套函数概念。

### 13.1 函数作用域

在 Python 中，作用域（Scope）决定了变量的可见性和生命周期。主要分为以下四种作用域，遵循 LEGB
（Local、Enclosing、Global、Built-in） 规则查找变量：

* 局部作用域（Local Scope）：  
  **定义：** 在函数内部定义的变量，作用范围仅限于该函数内部；  
  **特点：** 函数调用结束后，局部变量被销毁；  
  **示例：**
  ```python
  def my_function():
    x = 10  # 局部变量 x
    print(x)
  my_function()  # 输出: 10
  ```
* 嵌套作用域（Enclosing Scope）：    
  **定义：** 在嵌套函数中，内部函数可以访问外部函数的变量；    
  **特点：** 外部函数的变量在内部函数中继续存在，形成闭包；  
  **示例：**
  ```python
  def outer_function():
      x = 5
      def inner_function():
          print(x)  # 可以访问外部函数的变量
      return inner_function
  inner = outer_function()
  inner()  # 输出: 5 
  ```
* 全局作用域（Global Scope）：  
  **定义：** 在函数外部定义的变量，可以在整个模块中访问；    
  **特点：** 全局变量在模块的任何地方都可以访问，甚至可以通过 global 关键字在函数内部修改；  
  **示例：**
  ```python
  x = 20  # 全局变量
  def my_function():
      global x
      x = 30  # 修改全局变量
  my_function()
  print(x)  # 输出: 30
  ```
* 内置作用域（Built-in Scope）：  
  **定义：** Python 解释器提供的内置命名空间，包括内置函数、异常等；     
  **示例：**  使用 Python 提供的 len()、print() 函数；
  ```python
  print(len("hello"))  # 输出: 5
  ```

**LEGB 规则**

Python 查找变量时按照 LEGB 顺序进行查找：

`Local`：局部作用域，函数内部变量；  
`Enclosing`：嵌套函数的外部函数中的变量；  
`Global`：全局作用域，模块级别变量；  
`Built-in`：内置作用域，Python 预定义的名称；

### 13.2 嵌套函数

嵌套函数是指在一个函数内部定义的另一个函数。内部函数可以访问外部函数的变量，但外部函数不能访问内部函数的变量。

示例：嵌套函数可以将相关的功能逻辑封装在一起，使代码更结构化和清晰。例如，外部函数可以定义一个通用的处理逻辑，而内部函数可以处理特定的细节。

```python
def foo():
    # foo是外围函数 
    a = 1

    # printer是嵌套函数 
    def printer():
        print(a)

    printer()


foo()  # 1
```

示例中的变量 a 可以被嵌套函数 printer 正常访问。

嵌套函数与闭包密切相关，闭包是指内部函数保留了对外部函数局部变量的引用，即使外部函数执行完毕后，内部函数仍然可以访问这些变量。闭包广泛用于函数工厂、装饰器等场景中。

### 13.3 闭包

#### 13.3.1 闭包

闭包是指在一个函数内部定义的函数，这个内部函数可以访问外部函数的变量，即使外部函数已经返回。这种机制可以保持外部函数的变量在内部函数中的状态。
同时，闭包是函数式编程的重要语法结构，也是一种代码组织结构，提高了代码的可重复使用性。

创建一个闭包必须满足以下几点:

* 嵌套函数：必须有一个外部函数和一个内部函数。内部函数在外部函数内部定义；
* 访问外部变量：内部函数必须引用外部函数的变量（即外部函数的局部变量）；
* 外部函数返回内部函数：外部函数必须返回内部函数，这样外部函数的作用域就会通过内部函数“延续”；
* 外部函数已执行结束：外部函数执行结束后，返回的内部函数依然可以访问外部函数中的局部变量；

原理：

闭包的原理在于函数的作用域链和变量的生命周期。当一个函数在另一个函数内部定义时，内部函数可以访问外部函数的局部变量。如果外部函数返回了内部函数，并且该内部函数在外部被调用，Python
解释器会将这些外部变量保存在内部函数的环境中，即使外部函数的执行已经结束。这种机制让内部函数能够“记住”外部函数中的变量值。

关键点:

* 作用域链：内部函数能够访问外部函数的局部变量；
* 延迟绑定：变量值在函数调用时确定；
* 持久化环境：即使外部函数结束，内部函数仍能访问外部变量；

示例1：

```python
def outer_function(x):
    def inner_function(y):
        return x + y  # 这里 x 是外部函数的变量

    return inner_function


add_five = outer_function(5)
print(add_five(3))  # 输出: 8
```

当 outer_function 返回 inner_function 时，x 被保存在 inner_function 的环境中。 当调用 add_five(3) 时，x 的值已经确定，但仍可以在闭包中被访问。

示例2：局部变量在函数外被访问

```python
def foo():
    # foo是外围函数 
    a = 1

    # printer是嵌套函数 
    def printer():
        print(a)

    return printer


x = foo()
x()  # 1
```

函数中的局部变量仅在函数的执行期间可用，一旦 foo() 执行过后，我们会认为变量 a 将不再可用。然而，在这里我们发现 foo 执行完之后，在调用
x 的时候 a 变量的值正常输出了，这就是闭包的作用，闭包使得局部变量在函数外被访问成为可能。

#### 13.3.2 使用场景

* 保持状态（例如：计数器）
  闭包可以用来保持函数的内部状态，在不使用类的情况下实现类似对象的行为。
  ```python
  def counter():
    count = 0

    def increment():
        # 使用 `nonlocal` 关键字，可以在内部函数中修改外部函数的变量
        nonlocal count
        count += 1
        return count

    return increment

  counter_func1 = counter()
  print(counter_func1())  # 输出: 1
  print(counter_func1())  # 输出: 2
  print(counter_func1())  # 输出: 3
  counter_func2 = counter()
  print(counter_func2())  # 输出: 1
  ```
* 延迟计算： 闭包可以将某些计算延迟到函数实际调用时再进行。
  延迟计算可以用来优化性能，避免不必要的计算，只有在需要时才进行计算。以下是一个延迟计算的示例：
  ```python
  def lazy_addition(a, b):
      def compute():
          print("计算开始...")
          return a + b
      return compute

  result = lazy_addition(10, 20)

  # 延迟计算，不调用则不计算
  print(result())  # 输出: 计算开始... 30
  ```
  compute 是一个闭包，它保存了 a 和 b 的值，并在需要时才进行计算。

* 回调函数： 使用闭包实现回调函数
  假设我们有一个异步操作（比如模拟的网络请求），当操作完成时，我们需要调用一个回调函数处理结果。
  ```python
  import time
  
  
  def async_operation(callback):
      print("开始异步操作...")
      time.sleep(2)  # 模拟耗时操作
      result = "数据处理完成"
      callback(result)
  
  
  def callback_factory(user):
      def callback(result):
          print(f"{user} 接收到异步操作的结果: {result}")
  
      return callback
  
  
  # 为不同用户生成不同的回调函数
  callback_for_alice = callback_factory("Alice")
  callback_for_bob = callback_factory("Bob")
  
  # 进行异步操作并传入回调函数
  async_operation(callback_for_alice)  # Alice 接收到异步操作的结果: 数据处理完成
  async_operation(callback_for_bob)  # Bob 接收到异步操作的结果: 数据处理完成
  ```

  * callback_factory 是一个闭包工厂函数，根据 user 参数生成不同的回调函数；
  * callback 函数被传递给 async_operation，用于在异步操作完成后处理结果；
  * 由于闭包的特性，每个回调函数都记住了 user 的值，即使在异步操作执行过程中 user 的值不会改变；

  **使用场景：**
  * 异步编程：在异步操作中，闭包可以保留状态，并在操作完成时传递给回调函数；
  * 事件驱动编程：闭包可以用于定义不同的事件处理函数，并根据不同的上下文信息生成回调函数；

* 函数工厂  
  闭包可以用来创建带参数的函数工厂，根据不同的参数生成不同的函数。
  ```python
  def power_factory(exponent):
      def power(base):
          return base ** exponent
  
      return power
  
  
  # 创建平方函数
  square = power_factory(2)
  print(square(4))  # 输出: 16
  
  # 创建立方函数
  cube = power_factory(3)
  print(cube(2))  # 输出: 8
  ```
  * power_factory 是一个外部函数，它接受一个 exponent 参数；
  * power 是一个内部函数，计算 base 的 exponent 次幂；
  * power_factory 返回 power 函数，创建一个带有特定指数的函数（如平方或立方）；

* 延迟计算的函数工厂
  ```python
  def delayed_computation_factory(operation, x, y):
      def computation():
          print("开始计算...")
          result = operation(x, y)
          return result
      return computation
  
  # 定义一些操作函数
  def add(a, b):
      return a + b
  
  def multiply(a, b):
      return a * b
  
  # 创建延迟计算的闭包
  delayed_add = delayed_computation_factory(add, 10, 20)
  delayed_multiply = delayed_computation_factory(multiply, 5, 6)
  
  # 执行计算
  print(delayed_add())       # 输出: 开始计算... 30
  print(delayed_multiply())  # 输出: 开始计算... 30
  ```

  * 函数工厂：delayed_computation_factory 是一个函数工厂，它接受一个操作函数 operation 以及两个操作数 x 和 y，并返回一个闭包
    computation；
  * 闭包：computation 是一个闭包，保存了外部函数的参数和操作逻辑，只有在调用时才执行计算；
  * 延迟计算：计算过程不会在创建闭包时执行，而是等到调用闭包时才执行；

#### 13.3.3 面试示例（闭包、嵌套函数、作用域）

* 闭包
  * 什么是闭包？请解释它的工作原理？  
    闭包是指在一个函数内部定义另一个函数，并且内部函数引用了外部函数中的变量。即使外部函数已执行完毕并返回，内部函数仍能访问这些变量。闭包通常用于工厂函数、延迟计算、回调函数等场景。
  * 闭包的优点是什么？它如何帮助你在函数外部保存状态？  
    闭包允许在函数外部保存状态，避免使用全局变量，使代码更模块化和安全。通过闭包，函数可以捕获并“记住”外部环境中的变量，并在将来调用时使用这些变量。
  * 闭包与全局变量相比有什么优势？   
    闭包提供了更好的封装性，可以避免全局命名空间污染和意外修改，同时保持状态。
* 嵌套函数
  * 什么是嵌套函数？在什么情况下使用嵌套函数？  
    嵌套函数是定义在另一个函数内部的函数。它用于封装逻辑、实现闭包或访问外部函数的变量。
  * 请解释嵌套函数如何访问外部函数的变量?  
    嵌套函数通过其作用域链可以访问外部函数的变量。这些变量被保存在闭包中，即使外部函数返回后，嵌套函数仍能访问它们。
  * 如何通过嵌套函数实现信息隐藏或封装？  
    将功能逻辑封装在嵌套函数中，只暴露外部函数。外部无法直接访问嵌套函数，从而实现信息隐藏。
* 作用域
  * 请解释 Python 中的作用域和 LEGB（Local, Enclosing, Global, Built-in）规则？  
    Python 通过 LEGB 规则解析变量：首先在本地作用域 (Local)，然后在嵌套的外部作用域 (Enclosing)，接着在全局作用域 (
    Global)，最后在内置作用域 (Built-in) 中查找变量。
  * 什么是 nonlocal 关键字？它的作用是什么？  
    nonlocal 用于在嵌套函数中声明外部函数中的变量，使得该变量在内层函数中可修改，而非局部变量。
  * 请解释 global 和 nonlocal 关键字的区别及使用场景？  
    global 使函数内部的变量绑定到全局作用域中的同名变量，适用于全局变量的修改。  
    nonlocal 则用于绑定到嵌套函数的外部作用域中的变量，适用于嵌套函数间的数据共享。

## 14. 函数是一等公民 (First-Class Citizens)

在 Python 中，函数是一等公民（First-Class Citizens），这意味着函数在语言中可以像其他数据类型一样进行传递、返回、赋值等操作。函数一等公民的概念赋予了
Python 极大的灵活性，使其能够支持高级编程模式，如函数式编程、闭包、装饰器等。

* 示例1：函数作为变量传递
  ```python
  def greet(name):
      return f"Hello, {name}!"
  
  
  # 将函数赋值给变量
  say_hello = greet
  
  # 使用该变量调用函数
  print(say_hello("Alice"))  # 输出: Hello, Alice!
  ```

* 示例2：将函数作为参数传递给另一个函数
  ```python
  def add(x, y):
      return x + y
  
  def subtract(x, y):
      return x - y
  
  def operate(func, x, y):
      return func(x, y)
  
  # 将 add 函数作为参数传递
  result = operate(add, 10, 5)
  print(result)  # 输出: 15
  
  # 将 subtract 函数作为参数传递
  result = operate(subtract, 10, 5)
  print(result)  # 输出: 5
  ```

* 示例3：函数作为返回值
  ```python
  def make_multiplier(factor):
      def multiplier(x):
          return x * factor
      return multiplier
  
  times_2 = make_multiplier(2)
  print(times_2(5))  # 输出: 10
  ```

* 示例4：在容器中存储函数
  函数可以像其他对象一样存储在列表、元组、字典等容器中。
  ```python
  def square(x):
      return x * x
  
  def cube(x):
      return x * x * x
  
  functions = [square, cube]
  
  for func in functions:
      print(func(3))  # 输出: 9, 27
  ```

* 示例5：项目实际应用  
  在一个事件驱动的系统中，不同的事件会触发不同的回调函数。
  ```python
  def on_connect():
      print("Connected to the server!")
  
  def on_disconnect():
      print("Disconnected from the server!")
  
  def on_message(message):
      print(f"Received message: {message}")
  
  # 事件到回调函数的映射
  event_handlers = {
      "connect": on_connect,
      "disconnect": on_disconnect,
      "message": on_message
  }
  
  # 模拟事件处理系统
  def handle_event(event_type, *args):
      handler = event_handlers.get(event_type)
      if handler:
          handler(*args)
  
  # 触发不同的事件
  handle_event("connect")
  handle_event("message", "Hello, World!")
  handle_event("disconnect")
  ```
  本例中，event_handlers 字典将事件类型映射到对应的回调函数，并在事件触发时调用相应的函数。

## 15. 鸭子类型 (Duck typing)

### 15.1 鸭子类型

鸭子类型（Duck Typing）是一种动态类型语言中的编程概念，它源于 “如果一只鸟走起来像鸭子，游起来像鸭子，叫起来也像鸭子，那么它就是一只鸭子”的哲学。
在鸭子类型中，对象的实际类型并不重要，只要它实现了所需的方法或行为，就可以被认为是所需的类型。这意味着不需要明确的接口或继承关系，只要对象
“表现” 得像某种类型，它就可以被使用。

**应用场景：**

* 多态性  
  在鸭子类型中，函数或方法可以接受任何类型的对象，只要该对象实现了必要的方法。例如，在一个函数中传入任意对象，只要该对象有
  run() 方法，函数就可以正常执行。
* 灵活性  
  鸭子类型使代码更灵活，因为它不依赖于对象的具体类型或类层次结构。你可以传入不同类型的对象，只要它们有所需的行为。
* 简化代码  
  通过鸭子类型，代码不再需要大量的类型检查或复杂的继承关系，代码变得更简洁易读。

**示例说明：**

```python
class Dog:
    def speak(self):
        return "Woof!"


class Cat:
    def speak(self):
        return "Meow!"


def animal_sound(animal):
    return animal.speak()


dog = Dog()
cat = Cat()

print(animal_sound(dog))  # 输出: Woof!
print(animal_sound(cat))  # 输出: Meow!
```

上述示例，animal_sound 函数不关心传入的对象是 Dog 还是 Cat，只要对象有 speak() 方法，它就可以工作。

**优势：**

* 灵活性高：代码可以处理更多不同类型的对象；
* 降低耦合：减少了对特定类或接口的依赖；
* 提高可扩展性：更容易扩展代码，只需确保新对象实现必要的方法即可；

**面试示例：**

* 什么是鸭子类型？与静态类型检查有何区别？  
  鸭子类型是一种动态类型检查方式，不关心对象的实际类型，只要对象实现了所需的方法或行为即可使用。与静态类型检查不同，鸭子类型在运行时确定对象是否符合预期，而静态类型检查在编译时通过类型声明来确定。
* 鸭子类型的优点和缺点是什么？  
  优点：灵活性高，代码更易于扩展和复用，降低了对特定类或接口的依赖；  
  缺点：可能导致运行时错误，代码可读性和维护性降低，缺乏编译时的类型安全性；

### 15.2 使用场景用例

* 场景1：处理不同类型的日志记录器

  假设你有多个不同类型的日志记录器，如文件日志记录器和数据库日志记录器，它们都有一个 log
  方法。你可以使用鸭子类型来处理这些不同的记录器，而不需要担心它们的具体类型。

  ```python
  class FileLogger:
      def log(self, message):
          print(f"Logging to a file: {message}")
    
  class DatabaseLogger:
      def log(self, message):
          print(f"Logging to a database: {message}")
    
  def log_message(logger, message):
      logger.log(message)
    
  file_logger = FileLogger()
  db_logger = DatabaseLogger()
    
  log_message(file_logger, "This is a file log") # 输出：Logging to a file: This is a file log
  log_message(db_logger, "This is a database log") # 输出：Logging to a database: This is a database log
  ```

* 场景2：处理不同格式的数据

  假设你有不同格式的数据处理器，如 CSV 和 JSON 数据处理器。只要它们实现了相同的 process 方法，你就可以使用鸭子类型来统一处理。

  ```python
  class CSVProcessor:
      def process(self, data):
          print("Processing CSV data")
    
  class JSONProcessor:
      def process(self, data):
          print("Processing JSON data")
    
  def process_data(processor, data):
      processor.process(data)
    
  csv_processor = CSVProcessor()
  json_processor = JSONProcessor()
    
  process_data(csv_processor, "csv_data")
  process_data(json_processor, "json_data")
    
  ```

* 场景3：数据转换器的使用

  假设你有一个项目中需要将数据从一种格式转换为另一种格式，有不同的转换器实现了相同的 convert 方法。

  ```python
    class JSONConverter:
        def convert(self, data):
            print("Converting data to JSON")
    
    class XMLConverter:
        def convert(self, data):
            print("Converting data to XML")
    
    def convert_data(converter, data):
        converter.convert(data)
    
    json_converter = JSONConverter()
    xml_converter = XMLConverter()
    
    convert_data(json_converter, "some_data")
    convert_data(xml_converter, "some_data")
  ```

综上：原理一致，鸭子类型使得代码可以更加灵活和易于扩展，因为它依赖于对象的行为而非具体类型。这种编程风格尤其适用于需要处理多种类型对象但只关心它们所需行为的场景。

## 16. Python 中重载

函数重载是指在同一个作用域内定义多个同名函数，但它们的参数数量或类型不同，函数重载允许在调用时根据传入的参数自动选择合适的函数版本。
其主要是为了解决两个问题：

* 可变参数类型；
* 可变参数个数；

**Python 中的重载**

Python 不支持传统意义上的方法重载，即根据参数类型或数量来定义同名函数，如果在同一作用域内定义两个同名函数，后定义的函数会覆盖之前的定义。
对于重载解决的问题，Python 提供了一些机制可以实现类似于重载的功能：

* 可变参数类型：Python 作为动态类型语言，能够接受任意类型的参数，因此不需要为不同类型参数定义多个函数；
  ```python
  # 使用可变参数 (*args 和 **kwargs) 可以处理任意数量的参数，从而模拟函数重载的行为
  def add(*args):
    return sum(args)

  print(add(1, 2))             # 输出: 3
  print(add(1, 2, 3, 4))       # 输出: 10
  ```
* 可变参数个数：对于参数个数不同的情况，Python 通过默认参数（即缺省参数）来实现，因此也不需要函数重载机制；
  ```python
  # 通过使用默认参，模拟函数重载机制
  def greet(name, message="Hello"):
      return f"{message}, {name}!"
    
  print(greet("Alice"))        # 输出: Hello, Alice!
  print(greet("Bob", "Hi"))    # 输出: Hi, Bob!
  ```
* 【不推荐】基于类型的条件判断：通过使用条件判断，可以根据参数类型实现不同的行为。
  ```python
  from functools import singledispatch

  @singledispatch
  def process(data):
      print("Default processing", data)
    
  @process.register(int)
  def _(data):
      print("Processing integer", data)
    
  @process.register(list)
  def _(data):
      print("Processing list", data)
    
  process(10)  # 输出: Processing integer 10
  process([1, 2, 3])  # 输出: Processing list [1, 2, 3]
  process("hello")  # 输出: Default processing hello
  ``` 
* 【推荐】`functools.singledispatch`：基于参数类型的函数重载。  
  `functools.singledispatch` 是 Python 3.4
  引入的一个装饰器，用于实现基于参数类型的函数重载。它允许你定义一个通用函数，然后根据传递的第一个参数类型自动调用相应的特定版本。这种单分派机制在需要对不同类型的输入进行不同处理时非常有用。

  **使用方法：**
  * 使用 @singledispatch 装饰器定义一个通用函数；
  * 使用 @<generic_function>.register 为不同的类型定义特化版本；

  **主要功能：**
  * 单一分派：基于第一个参数的类型调用不同的函数；
  * 扩展性：可以通过 @<base_function>.register(type) 来注册新的类型处理函数，而不需要修改原函数；
  * 默认行为：未注册的类型会调用默认的基函数；

  **示例：**
  ```python
  from functools import singledispatch
    
  @singledispatch
  def process(data):
      return data
    
  @process.register(list)
  def _(data):
      return [x * 2 for x in data]
    
  @process.register(dict)
  def _(data):
      return {k: v * 2 for k, v in data.items()}
    
  print(process([1, 2, 3]))    # 输出: [2, 4, 6]
  print(process({"a": 1}))     # 输出: {'a': 2}
  ``` 
  应用场景：
  * 处理多种输入类型：如上例的 process 函数，可以根据输入类型执行不同的逻辑，常用于数据处理函数中；
  * API 设计：在设计 API 时，可以允许函数接受不同形式的输入，提供更灵活的接口；
  * 面向对象编程：在类的设计中，可以使用多态性来实现方法的重载行为；
  * 简化代码结构，避免多个 if-elif 语句；

**面试示例**

* Python 中是否支持方法重载？如何模拟重载？  
  Python 不支持传统的重载，但可以通过默认参数、*args、**kwargs 或 functools.singledispatch 来模拟重载行为。

* 解释 functools.singledispatch 的用途及其工作原理  
  singledispatch 是 Python 3.4+ 提供的一个装饰器，用于创建基于参数类型的泛型函数。通过注册不同类型的处理函数，可以实现基于类型的多态行为。

## 17. 新式类和旧式类

在 Python 中，类（Class）是一种用于创建对象的模板。Python 类主要分为旧式类（Classic Class）和新式类（New-Style
Class），两者主要区别在于它们在类的继承和方法解析方面的行为。

**【注意】：** 在 Python 3 中，旧式类已经完全被废弃。因此，了解旧式类的存在和基本区别对理解历史上下文有帮助，但在实际应用中不再需要处理旧式类，只需关注新式类即可。

### 17.1 新式类、旧式类区别

* 旧式类（Classic Class）：
  * 定义：在 Python 2.x 中，不显式继承自 object 的类被称为旧式类；
  * 继承顺序：使用深度优先搜索（DFS）来解析方法继承；
  * 方法解析顺序（MRO）：没有引入统一的 MRO；

  ```python
  # 旧式类定义
  class OldClass:
      pass
  ```
* 新式类（New-Style Class）：
  * 定义：在 Python 2.x 中，继承自 object 的类，以及在 Python 3.x 中的所有类，都是新式类；
  * 继承顺序：使用广度优先搜索（BFS）和 C3 线性化算法来解析方法继承；
  * 方法解析顺序（MRO）：引入了统一的 MRO，可以通过 __mro__ 属性查看类的解析顺序；
    例如：可以使用 类名.\__mro__ 或 类名.mro() 方法查看类的 MRO 顺序，如，MyClass.\__mro__ 会返回类的继承顺序：
    ```python
    class A:
        def show(self):
            print("From A")
    
    
    class B(A):
        def show(self):
            print("From B")
    
    
    class C(A):
        def show(self):
            print("From C")
    
    
    class D(B, C):
        pass
    
    
    print(D.mro()) # 输出：[<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
    print(D.__mro__) # 输出：(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
    ```
  * 元类：支持元类编程，允许通过自定义元类来控制类的创建过程；

  ```python
  # 新式类定义（Python 2 中需继承 object）
  class NewClass(object):
      pass
  # 或者 Python3
  class NewClass:
      pass
  ```
* 主要区别
  * 继承机制：旧式类使用深度优先，而新式类使用广度优先搜索（BFS）和 C3 线性化算法；
  * super() 函数：在新式类中，super() 可以正确地根据 MRO 调用父类方法；
  * 属性解析：新式类支持描述符协议（如 __get__、__set__、__delete__）和更复杂的属性解析机制；
  * 兼容性：新式类可以利用更多的 Python 特性，如装饰器和元类编程；
* 实际应用
  * 在 Python 3 中，所有类默认是新式类，因此通常无需显式继承 object；
  * 在 Python 2 中，建议使用新式类来编写兼容性更强和行为一致的代码；

### 17.2 Python 类继承：广度优先搜索（Python3.x/新式类）、深度优先搜索（Python2.x/旧式类）

* 广度优先搜索（BFS）在类继承中的应用  
  Python 3 中的 MRO 采用 C3 线性化算法，它基于广度优先搜索。这意味着在解析继承链时，优先访问兄弟类，再访问父类。C3
  线性化确保了继承路径的顺序和一致性。
  示例：
     ```python
     class A:
         def greet(self):
             print("Hello from A")
       
     class B(A):
         def greet(self):
             print("Hello from B")
       
     class C(A):
         def greet(self):
             print("Hello from C")
       
     class D(B, C):
         pass
       
     d = D()
     d.greet()  # 输出: Hello from B
     print(D.__mro__)  # 输出: (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
     ```
  本例中，D 类继承自 B 和 C，根据 MRO 规则，它首先检查 B，然后是 C，再是 A，最后是 object。

* 深度优先搜索（DFS）在类继承中的应用

  在 Python 2 中，MRO 采用的是深度优先搜索。这意味着在解析继承链时，会先递归访问父类，而不是平级类。Python 3
  采用广度优先搜索，解决了深度优先搜索可能带来的菱形继承问题。

* 区别

  * 搜索顺序：BFS 优先处理平级类，DFS 递归处理父类；
  * 应用：Python 3 中采用广度优先搜索（C3 线性化），确保了继承关系的线性化和无歧义性；

### 17.3 super() 使用

super() 在 Python 中用于调用父类的一个方法，通常在继承链中用于访问超类的初始化方法或其他方法。正确的调用方式如下：

* 单继承情况下
  ```python
  class Parent:
      def __init__(self):
          print("Parent init")

  class Child(Parent):
      def __init__(self):
          super().__init__()  # 调用父类的 __init__ 方法
          print("Child init")

  child = Child()  # 输出：Parent init Child init
  ```

* 多继承情况下
  ```python
  class A:
      def __init__(self):
          print("A init")
  
  
  class B(A):
      def __init__(self):
          super().__init__()  # 调用下一个类的 __init__ 方法，遵循 MRO 顺序
          print("B init")
  
  
  class C(A):
      def __init__(self):
          super().__init__()
          print("C init")
  
  
  class D(B, C):
      def __init__(self):
          super().__init__()  # 依次调用 B, C, A 的 __init__ 方法
          print("D init")
  
  
  d = D()
  
  # 输出：
  # A init
  # C init
  # B init
  # D init
  ```
  在以上示例中，super().__init__() 调用遵循类的 MRO（Method Resolution Order），即方法解析顺序。MRO
  确定了继承体系中各类的调用顺序。  
  MRO 调用链说明：
  * D 类的 \__init__() 方法首先调用 super().\__init__()；
  * 由于 D 继承自 B 和 C，所以 super() 首先调用 B.\__init__()；
  * 在 B.\__init__() 中，super().\__init__() 调用了 C.\__init__()；
  * C.\__init__() 调用 super().\__init__() 最终调用到 A.\__init__()；

  调用顺序是：D -> B -> C -> A，打印结果依次为：
  ```text
    A init
    C init
    B init
    D init
  ```

## 18. __new__和__init__区别

在 Python 中，`__new__` 和 `__init__` 是类对象两个重要方法，用于对象的创建和初始化，但两者有不同的功能和调用时机。

### 18.1 `__new__`和`__init__`

* `__new__` 方法：
  * 功能: 用于创建类实例，它是一个静态方法，会在对象实例化之前被调用；
  * 参数: 通常接收类本身 (cls) 作为第一个参数，并返回该类的一个新实例；
  * 应用场景: 通常用在需要控制类实例创建过程的场景，例如单例模式、元类的自定义等；
  * 示例：
    通常用于控制实例的创建过程。一个典型的应用场景是实现单例模式，即确保一个类只有一个实例：
    ```python
    class Singleton:
        _instance = None
    
        def __new__(cls, *args, **kwargs):
            if not cls._instance:
                cls._instance = super().__new__(cls)
            return cls._instance
    
        def __init__(self, name):
            self.name = name
    
    s1 = Singleton("Instance1")
    s2 = Singleton("Instance2")
    
    print(s1.name)  # 输出: Instance1
    print(s2.name)  # 输出: Instance1
    print(s1 is s2)  # 输出: True
    ```
  * 注意：在 Python 中，`__new__`
    方法通常不需要被覆盖，除非你在子类化一个不可变类型（如str、int、tuple等），这是因为不可变类型一旦创建，它们的内容就无法被改变，因此在创建实例时，必须通过 `__new__`
    方法控制实例的创建过程。  
    **理解：**
    * 不可变类型：对象创建后不能更改其内容，如：字符串和元组；
    * `__new__`：负责创建实例，返回新对象，对于不可变类型，覆盖 `__new__` 可以精确控制实例化过程，确保对象在创建时就具有正确的状态；
      **例如：**
    ```python
    class MyStr(str):
        def __new__(cls, content):
            return super().__new__(cls, content.upper())
    
    s = MyStr("hello")
    print(s)  # 输出: HELLO
    ```
    在这个示例中，MyStr 继承自 str，并覆盖了 `__new__` 方法，使得在创建 MyStr 实例时，字符串内容自动转换为大写。

* `__init__` 方法：
  * 功能: 用于初始化一个已经创建的实例，它是一个实例方法，会在对象实例创建之后被调用；
  * 参数: 通常接收实例本身 (self) 作为第一个参数，用于设置实例属性或执行初始化操作；
  * 应用场景: 在实例被创建后进行进一步的初始化，如设置初始状态或属性；
  * 示例：`__init__` 用于为实例 obj 设置初始值 value；
    ```python
    class MyClass:
        def __init__(self, value):
            self.value = value
  
    obj = MyClass(10)
    print(obj.value)  # 输出: 10
    ```

* 区别：
  * `__new__` 是类级别的方法，负责创建和返回一个新实例，而 `__init__` 是实例级别的方法，用于初始化这个实例；
  * `__new__` 必须返回一个实例，而 `__init__` 不返回值；

* 代码示例
  ```python
  class MyClass:
      def __new__(cls, *args, **kwargs):
          print("Creating instance")
          instance = super(MyClass, cls).__new__(cls)
          return instance
  
      def __init__(self, value):
          print("Initializing instance")
          self.value = value
  
  obj = MyClass(42)
  # 输出：
  # Creating instance
  # Initializing instance
  ```
  在这个例子中，`__new__` 在 `__init__` 之前被调用，用于创建实例，而 `__init__` 用于对创建的实例进行初始化。

### 18.2 `__new__`和`__init__` 与元类区别

`__new__`、`__init__` 和元类是 Python 中对象创建和类行为自定义的核心概念，但它们的作用和应用场景不同。

* `__new__` 和 `__init__`
  * `__new__`: 是类级别的方法，负责创建一个新的实例，它在实例化之前被调用，主要用于控制实例的创建过程；
  * `__init__`: 是实例级别的方法，负责初始化已经创建的实例，它在实例化之后被调用，主要用于设置实例的初始状态；

* 元类
  * 元类（Metaclass）: 是创建类的类，元类控制类的创建和行为，可以通过定义 `__new__` 和 `__init__` 方法自定义类的构造过程。元类最常见的应用是通过继承
    type 来创建新类；

* 区别
  * `__new__` 和 `__init__` 都是用于实例的创建和初始化，`__new__` 负责创建实例，`__init__` 负责初始化实例；
  * 元类用于控制类的创建过程，而不是实例的创建，元类允许你在类创建时定制行为，比如动态修改类属性或方法；

* 应用场景
  * `__new__`: 当需要控制对象实例化过程时，如实现单例模式；
  * 元类: 当需要动态生成类或修改类行为时，如框架中的自动化代码生成；

## 19. 设计模式

### 19.1 单例模式（Singleton Pattern）

同步可参考个人另一博文：[《Python：用于有效对象管理的单例模式》](https://mp.weixin.qq.com/s/1DkoeUAgEoU2AlhfZdvD4A)

单例模式确保一个类只有一个实例，并提供一个全局访问点。它适用于需要唯一对象的场景，如：数据库连接、配置文件管理等。

Python 中实现单例模式的方式有多种，以下是几种常见的实现方法：

#### 19.1.1 使用 `__new__` 方法

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super().__new__(cls)
        return cls._instance


# 示例
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # 输出: True
```

或者：

```python
class Singleton:
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super().__new__(cls)
        return cls._instance


class MyClass(Singleton):
    pass


my1 = MyClass()
my2 = MyClass()
print(my1 is my2)  # 输出: True
```

#### 19.1.2 使用装饰器

```python
def singleton(cls):
    instances = {}

    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance


@singleton
class Singleton:
    pass


# 示例
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # 输出: True
```

#### 19.1.3 共享属性

共享属性单例的原理基于 Borg 模式，该模式通过共享状态（即共享属性）来实现单例效果。具体来说，每个实例对象的属性字典（__dict__
）都指向同一个共享状态字典。这意味着，无论创建多少实例，这些实例的属性都共享相同的状态。

**工作原理：**

* 类属性共享状态：类中定义一个共享的字典来保存状态；
* 重写 `__new__` 方法：在 `__new__` 方法中，将每个新实例的 `__dict__` 指向共享状态字典；
* 实例共享属性：所有实例共享相同的属性和值，修改一个实例的属性会影响所有实例；

实现了状态共享，不同于传统单例模式，该模式允许多个实例存在，但它们共享状态。如，示例：

```python
class Borg:
    _shared_state = {}  # 类属性，共享所有实例的状态

    def __new__(cls, *args, **kwargs):
        obj = super().__new__(cls)
        obj.__dict__ = cls._shared_state  # 使所有实例的 __dict__ 指向相同的字典
        return obj


class MyClass(Borg):
    def __init__(self, name):
        self.name = name


a = MyClass("Alice")
b = MyClass("Bob")

print(a.name)  # 输出: Bob
print(b.name)  # 输出: Bob
print(a is b)  # 输出: False
```

在这个例子中，a 和 b 虽然是不同的实例，但由于它们共享同一个 `__dict__`，因此属性 name 的修改在所有实例中都会反映出来。

#### 19.1.4 使用元类（MetaClass）

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class Singleton(metaclass=SingletonMeta):
    pass


# 示例
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # 输出: True
```

#### 19.1.5 使用模块（import方法）

模块在 Python 中天然就是单例的，因为模块在第一次导入时会被缓存，后续的导入都是直接从缓存中获取。

```python
# singleton_module.py
class Singleton:
    pass


singleton = Singleton()

# 示例
from singleton_module import singleton

s1 = singleton
s2 = singleton
print(s1 is s2)  # 输出: True
```

#### 19.1.6 线程安全单例

线程安全的单例模式用于在多线程环境下确保单例对象的唯一性。如果多个线程同时尝试创建单例实例，可能会导致多个实例被创建，这违反了单例模式的设计初衷。通过确保线程安全，可以避免数据不一致、资源浪费等问题，确保系统的稳定性和正确性。

线程安全的单例可以通过以下方法实现：

* 使用锁（Lock）：在创建实例时加锁，确保只有一个线程能创建实例；
* 双重检查锁（Double-Checked Locking）：先检查实例是否存在，如果不存在则加锁创建；
* 模块级别单例：Python模块本身是线程安全的，导入时只会执行一次初始化；

使用锁适用于需要严格控制访问的场景，双重检查锁更高效，而模块级别单例则简单直接。

**线程安全：示例**

* 使用锁（Lock）

```python
import threading


class Singleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        with cls._lock:
            if not cls._instance:
                # cls._instance = super(Singleton, cls).__new__(cls)
                cls._instance = super().__new__(cls)
        return cls._instance


singleton1 = Singleton()
singleton2 = Singleton()

print(singleton1 is singleton2)  # 输出: True
```

**说明：** 使用 threading.Lock() 来确保在实例化时只有一个线程可以进入创建实例的代码块。

> 【备注】：
> `super(Singleton, cls).__new__(cls)` 可以简化为 `super().__new__(cls)`
> 在 Python 3 中，super() 不需要显式传递类和实例对象。它会自动解析为当前类的父类并返回该父类的 `__new__`
方法。因此，两者效果相同，但 `super().__new__(cls)` 更简洁。

* 双重检查锁（Double-Checked Locking）

```python
import threading


class Singleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            with cls._lock:
                if not cls._instance:
                    cls._instance = super().__new__(cls)
        return cls._instance


singleton1 = Singleton()
singleton2 = Singleton()

print(singleton1 is singleton2)  # 输出: True
```

**说明：** 双重检查锁先检查 _instance 是否存在，避免了不必要的加锁开销，提升性能。

* 模块级别单例

```python
# singleton.py
class Singleton:
    pass


singleton = Singleton()
```

```python
# main.py
from singleton import singleton

singleton1 = singleton
singleton2 = singleton

print(singleton1 is singleton2)  # 输出: True
```

**说明：** 在 Python 中，模块本身是线程安全的，导入时只会执行一次初始化，后续导入不会再次初始化。

### 19.2 工厂模式（Factory Pattern）

同步可参考个人另一博文：[《Python： 开始使用工厂模式设计》](https://mp.weixin.qq.com/s/iqsJTjJOWsn4nrC-L7SU3g)

工厂模式是一种创建型设计模式，用于定义一个接口或抽象类来创建对象，但将实例化推迟到子类中。其核心思想是通过工厂方法来处理对象的创建，而不是在代码中直接调用构造函数。这使得代码更加灵活和可扩展，尤其是在需要通过不同条件创建不同类型对象时。
以下是常见工厂实现方式：

* 简单工厂模式（Simple Factory）
  * 概念: 由一个工厂类根据传入的参数决定创建哪种具体产品；
  * 优点: 客户端与产品的创建分离，客户端不需要知道产品创建的逻辑，只需要消费该产品即可;
  * 缺点:
    工厂类集成了所有产品的创建逻辑，当工厂类出现问题，所有产品都会出现问题；还有当新增加产品都会修改工厂类，违背开闭原则 ；
  * 应用场景: 创建单一产品时；

* 工厂方法模式（Factory Method）
  * 概念: 定义一个创建对象的接口，由子类决定实例化哪个类;
  * 优点: 遵循开放/封闭原则，更容易扩展;
  * 缺点: 需要为每种产品都创建一个具体工厂类，类的数量会增加;
  * 应用场景: 当有多个产品类型，需要根据具体子类来确定产品实例时;

* 抽象工厂模式（Abstract Factory）
  * 概念: 提供一个接口，创建一系列相关或相互依赖的对象，而无需指定具体类。
  * 优点: 可以创建一系列相关的产品，确保产品之间的一致性。
  * 缺点: 复杂度较高，增加新产品族时需要扩展抽象工厂。
  * 应用场景: 需要创建多个相关产品对象时，如跨平台开发。

* 总结
  * 简单工厂: 适合单一产品创建。
  * 工厂方法: 适合多个产品类型创建，扩展性更好。
  * 抽象工厂: 适合创建多个相关产品族，确保一致性。

#### 19.2.1 简单工厂模式

简单工厂模式通常包含一个工厂类，该类具有一个创建方法，根据传入的参数决定实例化哪个具体类。

![python_features19.2.1.png](..%2F_static%2Fpython_features19.2.1.png)

```python
from abc import ABC, abstractmethod


class Phone(ABC):
    @abstractmethod
    def make(self):
        pass


class Huawei(Phone):
    def make(self):
        return "Manufacture of Huawei mobile phones!"


class Iphone(Phone):
    def make(self):
        return "Manufacture of iPhone mobile phones!"


class PhoneFactory:
    @staticmethod
    def create_phones(phone_type):
        # Python 3.10+：match-case
        # match phone_type:
        #     case "huawei":
        #         return Huawei()
        #     case "iphone":
        #         return Iphone()
        #     case _:
        #         print("No Phone Type.")
        if phone_type == "huawei":
            return Huawei()
        elif phone_type == "iphone":
            return Iphone()
        else:
            print("No Phone Type.")


# 使用简单工厂创建对象
huawei = PhoneFactory.create_phones("huawei")
print(huawei.make())  # 输出: Manufacture of Huawei mobile phones!

iphone = PhoneFactory.create_phones("iphone")
print(iphone.make())  # 输出: Manufacture of iPhone mobile phones!
```

**说明：** PhoneFactory 类提供了一个静态方法 create_phones，根据传入的 phone_type 参数，返回相应的 Huawei 或 Iphone
实例。客户端只需要调用工厂方法，无需关心具体的 Huawei 或 Iphone 类。

**实际生产用例：日志处理系统中的简单工厂模式**

在实际生产中，简单工厂模式可以用于创建不同类型的日志处理器（如控制台日志、文件日志、数据库日志等），以便根据配置或运行时条件灵活地选择日志记录方式。

示例代码：

```python
class Logger:
    def log(self, message):
        pass


class ConsoleLogger(Logger):
    def log(self, message):
        print(f"ConsoleLogger: {message}")


class FileLogger(Logger):
    def log(self, message):
        with open("logfile.txt", "a") as file:
            file.write(f"FileLogger: {message}\n")


class DatabaseLogger(Logger):
    def log(self, message):
        # 模拟数据库写入
        print(f"DatabaseLogger: {message} (Written to database)")


class LoggerFactory:
    @staticmethod
    def create_logger(logger_type):
        if logger_type == "console":
            return ConsoleLogger()
        elif logger_type == "file":
            return FileLogger()
        elif logger_type == "database":
            return DatabaseLogger()
        else:
            raise ValueError("Unknown logger type")


# 使用简单工厂根据配置创建日志处理器
logger_type = "file"  # 可以从配置文件或运行时动态确定
logger = LoggerFactory.create_logger(logger_type)
logger.log("This is a test log message.")
```

#### 19.2.2 工厂方法模式

定义一个用于创建对象的工厂接口，但让子类决定实例化哪个类，工厂方法模式使得类的实例化推迟到子类。

![python_features19.2.2.png](..%2F_static%2Fpython_features19.2.2.png)

```python
from abc import ABC, abstractmethod


class Phone(ABC):
    @abstractmethod
    def make(self):
        pass


class Huawei(Phone):
    def make(self):
        return "Manufacture of Huawei mobile phones!"


class Iphone(Phone):
    def make(self):
        return "Manufacture of iPhone mobile phones!"


class PhoneFactory(ABC):
    @abstractmethod
    def create_phones(self):
        pass


class HuaweiFactory(PhoneFactory):
    def create_phones(self):
        return Huawei()


class IphoneFactory(PhoneFactory):
    def create_phones(self):
        return Iphone()


factory = HuaweiFactory()
phone = factory.create_phones()
print(phone.make())  # 输出: Manufacture of Huawei mobile phones!
```

说明：PhoneFactory 工厂接口，由子类决定实例化对象，如：HuaweiFactory 实例化 Huawei、IphoneFactory 实例化 Iphone。

**实际生产用例：数据库连接器**  
在一个大型系统中，需要根据不同的数据库类型（如 MySQL、PostgreSQL、SQLite）创建对应的数据库连接器。工厂方法模式可以使代码灵活地支持多种数据库，而无需在主代码中硬编码数据库类型。

示例代码：

```python
from abc import ABC, abstractmethod


class DatabaseConnector(ABC):
    @abstractmethod
    def connect(self):
        pass


class MySQLConnector(DatabaseConnector):
    def connect(self):
        print("Connecting to MySQL")


class PostgreSQLConnector(DatabaseConnector):
    def connect(self):
        print("Connecting to PostgreSQL")


class SQLiteConnector(DatabaseConnector):
    def connect(self):
        print("Connecting to SQLite")


class DatabaseFactory(ABC):
    @abstractmethod
    def create_connector(self):
        pass


class MySQLFactory(DatabaseFactory):
    def create_connector(self):
        return MySQLConnector()


class PostgreSQLFactory(DatabaseFactory):
    def create_connector(self):
        return PostgreSQLConnector()


class SQLiteFactory(DatabaseFactory):
    def create_connector(self):
        return SQLiteConnector()


# 使用工厂方法模式根据配置创建数据库连接器
config = "PostgreSQL"  # 可以从配置文件或运行时动态确定
if config == "MySQL":
    factory = MySQLFactory()
elif config == "PostgreSQL":
    factory = PostgreSQLFactory()
elif config == "SQLite":
    factory = SQLiteFactory()

connector = factory.create_connector()
connector.connect()  # 输出：Connecting to PostgreSQL
```

* 代码解耦：工厂方法模式将对象的创建与使用分离，使得代码更为灵活，增加了系统的可扩展性。添加新的类型时，只需添加新的工厂类，不需要修改现有代码；
* 符合开闭原则：代码对扩展开放，对修改封闭。新对象的创建逻辑可以通过扩展新的工厂类来实现，而不影响现有代码的稳定性；
* 提高代码可读性：使用工厂方法模式后，创建对象的代码更简洁清晰，更容易维护；

#### 19.2.3 抽象工厂模式

提供一个创建一系列相关或相互依赖对象的接口，而无需指定具体类，由其子类工厂实现创建相关对象。

```python
from abc import ABC, abstractmethod


# 抽象产品
class Button(ABC):
    # 按钮
    @abstractmethod
    def click(self):
        pass


class Checkbox(ABC):
    # 复选框
    @abstractmethod
    def toggle(self):
        pass


# 具体产品
class WindowsButton(Button):
    def click(self):
        return "Windows Button Clicked"


class MacOSButton(Button):
    def click(self):
        return "MacOS Button Clicked"


class WindowsCheckbox(Checkbox):
    def toggle(self):
        return "Windows Checkbox Toggled"


class MacOSCheckbox(Checkbox):
    def toggle(self):
        return "MacOS Checkbox Toggled"


# 抽象工厂
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self):
        pass

    @abstractmethod
    def create_checkbox(self):
        pass


# 具体工厂
class WindowsFactory(GUIFactory):
    def create_button(self):
        return WindowsButton()

    def create_checkbox(self):
        return WindowsCheckbox()


class MacOSFactory(GUIFactory):
    def create_button(self):
        return MacOSButton()

    def create_checkbox(self):
        return MacOSCheckbox()


# 客户端代码
def create_ui(factory: GUIFactory):
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    print(button.click())
    print(checkbox.toggle())


# 实际使用
factory = WindowsFactory()
create_ui(factory)
# 输出：
# Windows Button Clicked
# Windows Checkbox Toggled

factory = MacOSFactory()
create_ui(factory)
# 输出：
# MacOS Button Clicked
# MacOS Checkbox Toggled
```

说明：GUIFactory 抽象工厂接口定义了创建一组相关或相互依赖对象的接口，不指定具体类，由具体子类工厂实现创建相关的对象（WindowsFactory、MacOSFactory）。

**使用场景**

* 跨平台应用：如，可以根据不同平台创建不同的界面组件、封装不同共有云 API 接口（华为、阿里等）；
* 产品族的创建：如果需要创建一组相关的对象，而不是单一对象，可以使用抽象工厂；

工厂方法专注于创建单个产品，抽象工厂则是用于创建一系列相关产品。两者都遵循依赖倒置原则和开放/封闭原则，但抽象工厂模式更为复杂，适用于需要生成一组关联对象的场景。

#### 19.2.4 单例模式与工厂模式结合

确保工厂类只存在一个实例：

```python
class SingletonFactory:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super().__new__(cls, *args, **kwargs)
        return cls._instance


class DogFactory(SingletonFactory):
    def create_animal(self):
        return Dog()


factory1 = DogFactory()
factory2 = DogFactory()
print(factory1 is factory2)  # 输出: True
```

### 19.3 观察者模式（Observer Pattern）

观察者模式是一种行为设计模式，它定义了一种一对多的依赖关系。当一个对象的状态发生变化时，所有依赖于它的对象都会收到通知并自动更新。

**组成部分：**

* Subject（主题）：维护一组观察者，并提供方法来添加、删除和通知观察者；
* Observer（观察者）：定义一个接口，用于接收主题更新的通知；
* ConcreteSubject（具体主题）：实现主题接口，并在状态变化时通知所有观察者；
* ConcreteObserver（具体观察者）：实现观察者接口，并在接收到通知后更新自身状态；

```python
from abc import ABC, abstractmethod


class Subject:
    # Subject 类：代表观察者模式中的“主题”或“被观察对象”。它维护一个 _observers 列表，用于存储所有已注册的观察者
    # 在设计上,也可以考虑抽象一层,例如: 抽象主题 Subject(): 定义 __init__(), attach(), detach(), notify() ,具体主题ConcreteSubject(Subject),示例比较简单没有抽象
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        # attach(self, observer)：将一个观察者添加到 _observers 列表中
        self._observers.append(observer)

    def detach(self, observer):
        # detach(self, observer)：从 _observers 列表中移除指定的观察者
        self._observers.remove(observer)

    def notify(self, message):
        # notify(self, message)：遍历 _observers 列表，调用每个观察者的 update 方法，将 message 传递给他们
        for observer in self._observers:
            observer.update(message)


class Observer(ABC):
    # Observer 类：定义了观察者的接口。它包含一个 update(self, message) 方法，所有具体的观察者都必须实现这个方法以处理来自主题的通知
    @abstractmethod
    def update(self, message):
        pass


class ConcreteObserver(Observer):
    # ConcreteObserver 类：继承自 Observer，是一个具体的观察者实现
    def __init__(self, name):
        # __init__(self, name)：初始化观察者，并给它分配一个 name 属性
        self.name = name

    def update(self, message):
        # update(self, message)：实现了 Observer 的 update 方法，当收到来自主题的通知时，打印出收到的消息
        print(f"{self.name} received message: {message}")


# 使用示例
subject = Subject()  # 创建一个主题

# 创建2个观察者
observer1 = ConcreteObserver("Observer 1")
observer2 = ConcreteObserver("Observer 2")

# 将观察者注册到对应主题
subject.attach(observer1)
subject.attach(observer2)

# 通知观察者，传递消息
subject.notify("New event occurred!")

# 输出：
# Observer 1 received message: New event occurred!
# Observer 2 received message: New event occurred!
```

> **【扩展知识点】：**
>
>在 Python 中，当你定义一个抽象接口（即使用 abc.ABC 和 abc.abstractmethod 时），抽象方法通常需要定义 self 参数，特别是在实例方法中。self
是指向实例本身的引用，允许访问实例属性和方法。抽象方法和普通实例方法一样，需要传递 self 作为第一个参数。
>
>```python
>from abc import ABC, abstractmethod
>
>class MyAbstractClass(ABC):
>    @abstractmethod
>    def my_method(self, param):
>        pass
>```
>
>本例中，my_method 是一个抽象方法，self 需要作为第一个参数。

### 19.4 责任链模式（Chain of Responsibility Pattern）

#### 19.4.1 介绍

责任链模式（Chain of Responsibility
Pattern）是一种行为型设计模式，它允许多个对象有机会处理请求，避免了请求发送者与接收者之间的紧耦合；通过将多个处理对象串联成一条链，请求会沿着这条链传递，直到有对象处理它或者到达链末尾。可以将责任链模式理解为一个“接力赛跑”，每个对象都有机会处理请求，如果不能处理，就将请求传递给下一个对象。

**模式结构：**  
责任链模式主要包含以下角色：

* Handler（抽象处理者）：抽象基类，定义处理请求的方法，通常提供将请求传递给下一个处理者的接口；
* ConcreteHandler（具体处理者）：继承 Handler，实现抽象处理者的处理逻辑，如果不能处理，则将请求传递给下一个处理者；
* Client（客户端）：创建具体处理者对象并设置它们的职责链；

**原理：**  
责任链模式的核心在于链式调用，每个处理者持有对下一个处理者的引用，当一个处理者无法处理请求时，将请求转发给下一个处理者。这种方式使得请求的发送者无需知道请求将由哪个对象处理，增加了系统的灵活性和可扩展性。

**优势：**

* 降低耦合：请求的发送者与处理者之间不需要显式地知道对方，增加了系统的灵活性；
* 增强灵活性：可以动态地添加或移除处理者，轻松改变处理顺序；
* 遵循单一职责原则：每个处理者只关注自身的处理逻辑，职责清晰；

**缺点：**

* 可能无法保证请求被处理：如果链中没有处理者能够处理请求，可能导致请求未被处理；
* 调试困难：链中的处理者可能较多，跟踪请求的处理路径可能较为复杂；
* 性能问题：如果责任链过长，可能导致处理请求的时间增加；

**常见应用场景：**

* 订单审批流程：在企业中，订单的审批通常需要经过多个层级，例如经理、主管、总监等，根据订单金额的不同，订单需要不同层级的审批，责任链模式可以动态地构建审批流程；
* 表单验证：在用户提交表单时，可能需要进行多个验证步骤，如字段格式验证、逻辑验证、权限验证等。每个验证步骤可以作为责任链中的一个处理者，逐步验证输入数据的合法性；
* 日志处理系统：日志系统可能需要根据日志级别（如
  DEBUG、INFO、WARN、ERROR）将日志信息发送到不同的输出目标（如控制台、文件、远程服务器）。责任链模式可以动态地配置不同级别日志的处理逻辑；
* 数据处理流水线：在数据处理过程中，可能需要经过多个处理步骤，如数据清洗、转换、验证、存储等。每个处理步骤可以作为责任链中的一个处理者，依次处理数据；

#### 19.4.2 代码示例

* 示例1：订单审批流程  
  **需求描述：**
  订单金额不同，需要不同级别的审批人员进行审批：
  * 金额 < 1000：由经理审批
  * 1000 ≤ 金额 < 5000：由主管审批
  * 金额 ≥ 5000：由总监审批

  ```python
  from __future__ import annotations
  
  from abc import ABC, abstractmethod
  from dataclasses import dataclass
  from typing import Optional, Any
  
  
  @dataclass
  class Order:
      amount: float  # 金额
      description: str  # 描述
  
  # 处理程序接口
  class Handler(ABC):
      @abstractmethod
      def set_next(self, handler: Handler) -> Handler:
          # 声明一个用于构建处理程序链的方法
          pass
  
      @abstractmethod
      def handle(self, data) -> Optional[str]:
          # 声明一个用于执行处理方法
          pass
  
  # 基础类，定义默认处理程序行为
  class AbstractHandler(Handler):
      _next_handler: Handler = None
  
      def set_next(self, handler: Handler) -> Handler:
          self._next_handler = handler
          return handler
  
      @abstractmethod
      def handle(self, data: Any) -> str:
          if self._next_handler:
              return self._next_handler.handle(data)
          return ""
  
  
  # 具体处理者：经理
  class ManagerHandler(AbstractHandler):
      def handle(self, data: Order) -> str:
          if data.amount < 1000:
              return f"Manager approved order: {data.description} for ${data.amount}"
          else:
              return super().handle(data)
  
  
  # 具体处理者：主管
  class SupervisorHandler(AbstractHandler):
      def handle(self, data: Order) -> str:
          if 1000 <= data.amount < 5000:
              return f"Supervisor approved order: {data.description} for ${data.amount}"
          else:
              return super().handle(data)
  
  
  # 具体处理者：总监
  class DirectorHandler(AbstractHandler):
      def handle(self, data: Order) -> str:
          if data.amount >= 5000:
              return f"Director approved order: {data.description} for ${data.amount}"
          else:
              return super().handle(data)
  
  
  if __name__ == '__main__':
      # 创建订单
      orders = [
          Order(500, "Office Supplies"),
          Order(2000, "New Computers"),
          Order(7000, "Office Renovation")
      ]
  
      # 处理订单
      manager = ManagerHandler()
      supervisor = SupervisorHandler()
      director = DirectorHandler()
      manager.set_next(supervisor).set_next(director)
      for order in orders:
          print(manager.handle(order))
  
  # 输出：
  # Manager approved order: Office Supplies for $500
  # Supervisor approved order: New Computers for $2000
  # Director approved order: Office Renovation for $7000
  ```

  **代码解释：**
  * Order 类：使用 @dataclass 装饰器定义订单对象，包含 amount 和 description 两个属性；
  * AbstractHandler 基础类：实现了默认责任链模式行为，包含：默认处理行为及 _next_handler 对下一个处理者（审批者）的引用；
  * 具体处理者（ManagerHandler, SupervisorHandler, DirectorHandler）：实现 handle 方法，根据订单金额决定是否处理或转发给下一个处理者；
  * 客户端代码：
    * 构建责任链：manager -> supervisor -> director；
    * 创建多个订单并通过责任链进行审批；

* 示例2：表单验证  
  **需求描述：**  
  用户提交表单时，需要经过多个验证步骤：
  * 非空验证；
  * 格式验证（如邮箱格式）；
  * 逻辑验证（如密码匹配）；
  ```python
  from abc import ABC, abstractmethod
  
  # 请求对象
  @dataclass
  class FormData:
      username: str
      email: str
      password: str
      confirm_password: str
  
  # 抽象处理者
  class Validator(ABC):
      def __init__(self, successor=None):
          self._successor = successor
  
      @abstractmethod
      def validate(self, data: FormData):
          pass
  
  # 具体处理者：非空验证
  class NotEmptyValidator(Validator):
      def validate(self, data: FormData):
          if not all([data.username, data.email, data.password, data.confirm_password]):
              raise ValueError("All fields must be filled out.")
          if self._successor:
              self._successor.validate(data)
  
  # 具体处理者：格式验证
  import re
  
  class EmailFormatValidator(Validator):
      def validate(self, data: FormData):
          email_regex = r"[^@]+@[^@]+\.[^@]+"
          if not re.match(email_regex, data.email):
              raise ValueError("Invalid email format.")
          if self._successor:
              self._successor.validate(data)
  
  # 具体处理者：逻辑验证
  class PasswordMatchValidator(Validator):
      def validate(self, data: FormData):
          if data.password != data.confirm_password:
              raise ValueError("Passwords do not match.")
          if self._successor:
              self._successor.validate(data)
  
  # 客户端代码
  if __name__ == "__main__":
      # 构建验证链
      # password_validator = PasswordMatchValidator()
      # email_validator = EmailFormatValidator(password_validator)
      # not_empty_validator = NotEmptyValidator(email_validator)
      not_empty_validator = NotEmptyValidator(EmailFormatValidator(PasswordMatchValidator()))
      # 创建表单数据
      form_data = [
          FormData("john_doe", "john@example.com", "password123", "password123"),
          FormData("", "jane@example.com", "password123", "password123"),
          FormData("jane_doe", "janeexample.com", "password123", "password123"),
          FormData("jane_doe", "jane@example.com", "password123", "password321")
      ]
  
      # 验证表单数据
      for data in form_data:
          try:
              not_empty_validator.validate(data)
              print(f"Form data for {data.username} is valid.")
          except ValueError as e:
              print(f"Validation error for {data.username}: {e}")
  # 输出：
  # Form data for john_doe is valid.
  # Validation error for : All fields must be filled out.
  # Validation error for jane_doe: Invalid email format.
  # Validation error for jane_doe: Passwords do not match.
  ```

  **代码解释：**
  * FormData 类：定义表单数据结构，包含 username, email, password, confirm_password 四个字段；
  * Validator 抽象类：定义了验证器的接口，包含 validate 方法和对下一个验证器的引用 _successor；
  * 具体验证器（NotEmptyValidator, EmailFormatValidator, PasswordMatchValidator）：
    * NotEmptyValidator：检查所有字段是否非空；
    * EmailFormatValidator：检查邮箱格式是否正确；
    * PasswordMatchValidator：检查密码和确认密码是否匹配；
  * 客户端代码：
    * 构建验证链：NotEmptyValidator -> EmailFormatValidator -> PasswordMatchValidator；
    * 创建多个表单数据并进行验证；

* 示例3：基于责任链模式的数据格式校验示例代码，假设我们需要校验数据中的多个字段，如字符串的长度、数值的范围等；

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional


@dataclass
class Data:
    # Data 数据类：封装了待校验的数据属性，如字符串、分数和日期
    string_value: str
    score: int
    date: str


class _CheckInterface(ABC):
    # _CheckInterface 接口：定义了责任链的基础接口，包括设置下一个检查器(set_next)和执行检查(check)。
    @abstractmethod
    def set_next(self, checker: '_CheckInterface') -> '_CheckInterface':
        pass

    @abstractmethod
    def check(self, data: Data, **kwargs) -> Optional[str]:
        pass


class DataChecker(_CheckInterface):
    # DataChecker 抽象类：实现了责任链的基本逻辑，
    # 通过 set_next 方法构建链条，
    # check 方法逐级调用链条上的检查器，
    # raise_error 方法则抛出校验失败的异常
    _next_checker: Optional[_CheckInterface] = None

    def set_next(self, checker: _CheckInterface) -> _CheckInterface:
        self._next_checker = checker
        return checker

    def check(self, data: Data, **kwargs) -> Optional[str]:
        if self._next_checker:
            return self._next_checker.check(data, **kwargs)
        return None

    def raise_error(self, message: str):
        raise ValueError(f'校验失败: {message}')

    @classmethod
    def check_list(cls) -> 'DataChecker':

        # return CheckStringLength().set_next(CheckScoreRange()).set_next(CheckDateFormat())
        # 或者
        _checker = CheckStringLength()
        tmp = _checker
        for clazz in [CheckScoreRange, CheckDateFormat]:
            tmp = tmp.set_next(clazz())
        return _checker


class CheckStringLength(DataChecker):
    # 具体的检查器：校验字符串长度
    MIN = 5
    MAX = 20

    def check(self, data: Data, **kwargs) -> Optional[str]:
        if not (self.MIN <= len(data.string_value) <= self.MAX):
            self.raise_error(f'字符串长度需要在 {self.MIN}-{self.MAX} 之间')
        return super().check(data, **kwargs)


class CheckScoreRange(DataChecker):
    # 具体的检查器：分数范围
    MIN = 0
    MAX = 100

    def check(self, data: Data, **kwargs) -> Optional[str]:
        if not (self.MIN <= data.score <= self.MAX):
            self.raise_error(f'分值需要在 {self.MIN}-{self.MAX} 之间')
        return super().check(data, **kwargs)


class CheckDateFormat(DataChecker):
    # 具体的检查器：日期格式
    def check(self, data: Data, **kwargs) -> Optional[str]:
        try:
            year, month, day = map(int, data.date.split('-'))
            if len(data.date) != 10:
                raise ValueError
        except ValueError:
            self.raise_error('日期格式不正确，应为 YYYY-MM-DD')
        return super().check(data, **kwargs)


# 示例数据
data = Data(string_value='HelloWorld', score=85, date='2024-08-30')

try:
    checker = DataChecker.check_list()
    checker.check(data)
    print("数据校验通过")
except ValueError as e:
    print(str(e))
```

**扩展与重用：**

扩展其他校验时，可以通过继承 DataChecker 类并实现 check 方法来定义新的检查器。例如，如果需要增加电子邮件格式校验，只需创建一个新的检查类并加入到责任链中：

```python
class CheckEmailFormat(DataChecker):
    def check(self, data: Data, **kwargs) -> Optional[str]:
        if '@' not in data.string_value:
            self.raise_error('无效的电子邮件格式')
        return super().check(data, **kwargs)


# 在责任链中增加新的检查器
def check_list_with_email() -> DataChecker:
    return CheckStringLength().set_next(CheckEmailFormat()).set_next(CheckScoreRange()).set_next(CheckDateFormat())
```

通过责任链模式，新的校验逻辑可以轻松集成到现有系统中，且不影响已有的校验器，实现了代码的高度复用和扩展性。

> **【扩展知识点】：`from __future__ import annotations`**  
> `from __future__ import annotations` 是用于延迟类型注解的计算。它让 Python
在运行时才去解析类型注解，而不是在定义时立即解析。这样做可以避免在代码中由于前向引用或循环导入导致的问题。
> 从 Python 3.10 开始，类型注解默认是延迟计算的，因此在 Python 3.10
及以上版本中不再需要手动引入 `from __future__ import annotations`。但是，在 Python 3.9 及以下版本中，仍然需要使用该语句来启用这一功能。
> * 在 Python 3.10 及更高版本中，默认启用了 PEP
    563，这使得类型注解被推迟到运行时进行求值。这一特性让你不必再手动导入 `from __future__ import annotations`。但如果你希望在更早的
    Python 版本中使用此特性，仍然需要手动导入；
> * 在 Python 3.11 中，PEP 649 被引入，并将替换 PEP 563 的实现，因此无需再手动使用 `from __future__ import annotations`；

### 19.5 策略模式（Strategy Pattern）

策略模式（Strategy Pattern）是一种行为设计模式，它定义了一系列算法，并将每个算法封装到独立的类中，使得它们可以互相替换，在
Python 代码中很常见，经常在各种框架中使用，能在不扩展类的情况下向用户提供改变其行为的方式。

**关键点：**

* Context：使用策略对象的类，维护指向具体策略的引用，且仅通过策略接口与该对象进行交流；
* Strategy：策略接口或抽象类，定义了算法的共同行为；
* Concrete Strategy：具体策略类，实现了不同的算法；

![python_features19.5.1.png](..%2F_static%2Fpython_features19.5.1.png)

**示例：对 Lits 数据提供降序/升序排列算法**

```python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import List


class Strategy(ABC):
    """
    策略接口声明了某个算法各个不同版本间所共有的操作。Context 会使用该接口来调用有具体策略定义的算法
    """

    @abstractmethod
    def do_algorithm(self, data: List):
        pass


class Context:
    """
    持有一个 Strategy 对象的引用。Context 类通过 strategy 属性调用策略的算法
    """

    def __init__(self, strategy: Strategy) -> None:
        self._strategy = strategy

    @property
    def strategy(self) -> Strategy:
        return self._strategy

    @strategy.setter
    def strategy(self, strategy: Strategy) -> None:
        self._strategy = strategy

    def do_some_business_logic(self, data: List) -> None:
        # 展示如何使用策略对象来执行算法逻辑。在此例中，它使用策略对数据进行排序
        print("Context: Sorting data using the strategy (not sure how it'll do it)")
        result = self._strategy.do_algorithm(data)
        print(",".join(result))


class ConcreteStrategyA(Strategy):
    """
     实现按升序排序的算法
    """

    def do_algorithm(self, data: List) -> List:
        return sorted(data)


class ConcreteStrategyB(Strategy):
    """
     实现按降序排序的算法
    """

    def do_algorithm(self, data: List) -> List:
        return reversed(sorted(data))


if __name__ == "__main__":
    data = ['a', 'b', 'c', 'd', 'e', 'f']
    context = Context(ConcreteStrategyA())
    print("Client: Strategy is set to normal sorting.")
    context.do_some_business_logic(data)
    print()
    # 通过改变 Context 的 strategy，可以动态地切换使用不同的算法
    print("Client: Strategy is set to reverse sorting.")
    context.strategy = ConcreteStrategyB()
    context.do_some_business_logic(data)

# 输出：
# Client: Strategy is set to normal sorting.
# Context: Sorting data using the strategy (not sure how it'll do it)
# a,b,c,d,e,f
# 
# Client: Strategy is set to reverse sorting.
# Context: Sorting data using the strategy (not sure how it'll do it)
# f,e,d,c,b,a
```

**优点：**

* 灵活性：可以在运行时动态改变策略；
* 可扩展性：增加新策略不影响现有系统；

**应用场景：**  
策略模式适合场景：当系统有多种算法或行为，并且希望在运行时灵活选择其中一种时。

* 支付方式选择：根据用户选择的支付方式（如信用卡、PayPal），应用不同的支付策略；
* 路径规划：地图应用中可以根据不同的策略（最短路径、避开高速、避开收费）进行路径规划；

### 19.6 命令模式（Command Pattern）

命令模式（Command Pattern）是一种行为设计模式，它可将请求转换为一个包含与请求相关的所有信息的独立对象。该转换让你能根据不同的请求将方法参数化、延迟请求执行或将其放入队列中，且能实现可撤销操作。

**命令模式结构：**

* 命令对象（Command）：封装了一个具体的操作和它的参数；
* 调用者（Invoker）：持有命令对象并在某个时间点调用命令；
* 接收者（Receiver）：实际执行命令操作的对象；

![python_features19.6.1.png](..%2F_static%2Fpython_features19.6.1.png)

**示例：**

```python
from __future__ import annotations

from abc import ABC, abstractmethod


class Command(ABC):
    """
    Command 接口声明了一个执行命令的方法
    """

    @abstractmethod
    def execute(self) -> None:
        pass


class SimpleCommand(Command):
    """
    具体命令类：实现了简单打印任务
    """

    def __init__(self, payload: str) -> None:
        self._payload = payload

    def execute(self) -> None:
        print(f"SimpleCommand: See, I can do simple things like printing"
              f"({self._payload})")


class ComplexCommand(Command):
    """
    具体命令类：处理更复杂的操作，接受 Receiver 对象和一些参数，在 execute 中将任务委托给接收者。
    """

    def __init__(self, receiver: Receiver, a: str, b: str) -> None:
        """
        复杂命令可以通过构造函数接受一个或多个接收对象（receivers）以及任何上下文数据
        """
        self._receiver = receiver
        self._a = a
        self._b = b

    def execute(self) -> None:
        """
        命令执行委托给接收者（receivers）方法
        """
        print("ComplexCommand: Complex stuff should be done by a receiver object", end="")
        self._receiver.do_something(self._a)
        self._receiver.do_something_else(self._b)


class Receiver:
    """
    包含了执行实际任务的业务逻辑
    """

    def do_something(self, a: str) -> None:
        print(f"\nReceiver: Working on ({a}.)", end="")

    def do_something_else(self, b: str) -> None:
        print(f"\nReceiver: Also working on ({b}.)", end="")


class Invoker:
    """
    保存命令对象并在合适的时机调用它们。它通过 set_on_start 和 set_on_finish 方法设置在任务开始前和结束后的命令
    """
    _on_start = None
    _on_finish = None

    def set_on_start(self, command: Command):
        self._on_start = command

    def set_on_finish(self, command: Command):
        self._on_finish = command

    def do_something_important(self) -> None:
        """
        Invoker 不依赖于具体的命令或接收器类。Invoker 通过执行命令间接将请求传递给接收器
        """
        print("Invoker: Does anybody want something done before I begin?")
        if isinstance(self._on_start, Command):
            self._on_start.execute()

        print("Invoker: ...doing something really important...")

        print("Invoker: Does anybody want something done after I finish?")
        if isinstance(self._on_finish, Command):
            self._on_finish.execute()


if __name__ == "__main__":
    """
    通过设置不同的命令和接收者，展示了命令模式的灵活性
    """

    invoker = Invoker()
    invoker.set_on_start(SimpleCommand("Say Hi!"))
    receiver = Receiver()
    invoker.set_on_finish(ComplexCommand(
        receiver, "Send email", "Save report"))

    invoker.do_something_important()

# 输出：
# Invoker: Does anybody want something done before I begin?
# SimpleCommand: See, I can do simple things like printing(Say Hi!)
# Invoker: ...doing something really important...
# Invoker: Does anybody want something done after I finish?
# ComplexCommand: Complex stuff should be done by a receiver object
# Receiver: Working on (Send email.)
# Receiver: Also working on (Save report.)
```

**扩展性：**

* 新的命令可以通过继承 Command 类轻松扩展，无需修改现有代码；
* Invoker 类可以调用任意命令对象，使得命令链和任务序列化非常灵活；

**使用场景：**

* 任务撤销、重做功能；
* 事务脚本的执行（如数据库操作）；
* 宏命令（多个命令的组合）；

### 19.7 适配器模式（Adapter Pattern）

适配器模式（Adapter Pattern）是一种结构型设计模式，它允许将一个类的接口转换为客户希望的另一个接口，使得原本不兼容的类可以协同工作。
适配器模式在 Python 代码中很常见。 基于一些遗留代码的系统常常会使用该模式。 在这种情况下， 适配器让遗留代码与当前类得以相互合作。

**示例：** 假设你有一个旧的类 Adaptee，它的接口不符合新的系统 Target 所要求的接口。通过使用适配器模式，可以在不改变旧系统代码的情况下使其兼容新系统。

```python
class Adaptee:
    """
    表示旧系统，有自己的接口 specific_request
    """

    def specific_request(self) -> str:
        return "OldSystem's specific request"


class Target:
    """
    新系统所期望的接口
    """

    def request(self) -> str:
        return "NewSystem's request"


class Adapter(Target, Adaptee):
    """
    适配器类（Adapter），通过多重继承使得 Adaptee（旧系统） 的接口和 Target（新系统） 的接口兼容
    """

    def request(self) -> str:
        return f"Adapter: (TRANSLATED) {self.specific_request()}"


# 新系统中使用统一的接口
def client_code(target: Target) -> None:
    """
    客户端代码，直接与 TargetInterface 交互，无需关心其具体实现
    """
    print(target.request(), end="")


if __name__ == "__main__":
    # Target（新系统） 调用
    target = Target()
    client_code(target)
    print("\n")

    # 使用适配器来适应旧系统
    adapter = Adapter()
    client_code(adapter)

# 输出：
# NewSystem's request
# Adapter: (TRANSLATED) OldSystem's specific request
```

### 19.8 模板方法模式（Template Method Pattern）

模板方法模式（Template Method Pattern）是一种行为设计模式，它定义了一个算法的骨架，并允许子类在不改变算法结构的情况下重定义算法的某些步骤。此模式将代码的复用性和灵活性很好地结合在一起。

**示例场景：**  
假设你要实现一组操作步骤，每个步骤的细节可能会有所不同，但总体的操作流程是相同的。模板方法模式可以将这些步骤的框架定义在一个基类中，而具体的实现则交由子类完成。

* 当多个类有相似的逻辑结构时，可以使用模板方法模式将相同的逻辑部分提取到基类中，而不同的细节部分则由子类实现；
* 避免代码重复，提升代码的可维护性和扩展性；

**示例代码：**

```python
from abc import ABC, abstractmethod


class DataProcessor(ABC):
    """
    抽象类，定义了处理数据 process 方法，其中包含读取、处理和保存数据的步骤，具体由子类实现，但模板方法 process 本身应保持不变，定义操作调用组成及顺序。
    """

    def process(self):
        # 模板方法 process：在基类中定义，不可更改，但可以通过继承来实现具体的细节
        self.to_start()
        self.read_data()
        self.process_data()
        self.save_data()
        self.to_finish()

    def to_start(self) -> None:
        print("Start processing data")

    def to_finish(self) -> None:
        print("End of data processing")

    @abstractmethod
    def read_data(self) -> None:
        pass

    @abstractmethod
    def process_data(self) -> None:
        pass

    @abstractmethod
    def save_data(self) -> None:
        pass


class CSVProcessor(DataProcessor):
    """
    具体子类: CSV 数据的读取、处理和保存步骤
    """

    def read_data(self):
        print("Reading CSV data")

    def process_data(self):
        print("Processing CSV data")

    def save_data(self):
        print("Saving CSV data")


class JSONProcessor(DataProcessor):
    """
    具体子类: JSON 数据的读取、处理和保存步骤
    """

    def read_data(self):
        print("Reading JSON data")

    def process_data(self):
        print("Processing JSON data")

    def save_data(self):
        print("Saving JSON data")


def client_code(data_processor: DataProcessor) -> None:
    data_processor.process()


if __name__ == "__main__":
    client_code(CSVProcessor())
    print("\n")
    client_code(JSONProcessor())

# 输出：
# Start processing data
# Reading CSV data
# Processing CSV data
# Saving CSV data
# End of data processing
#
#
# Start processing data
# Reading JSON data
# Processing JSON data
# Saving JSON data
# End of data processing
```

### 19.9 组合模式（Composite Pattern）

组合模式（Composite Pattern）是一种结构型设计模式，它允许你将对象组合成树形结构来表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

**示例场景：** 假设你在开发一个文件系统，其中有文件和文件夹。文件夹可以包含文件或其他文件夹。你希望用户能够对文件和文件夹进行统一的操作，如获取大小、添加或删除内容等。

**示例代码：**

```python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import List


# 组件接口
class Component(ABC):
    """
    声明了组合中简单对象和复杂对象的通用操作
    """

    # 可选（parent）：基础组件可以声明一个接口，用于设置和访问树结构中的父级，并提供一些默认实现
    @property
    def parent(self) -> Component:
        return self._parent

    @parent.setter
    def parent(self, component: Component):
        self._parent = component

    def add(self, component: Component) -> None:
        pass

    def remove(self, component: Component) -> None:
        pass

    def is_composite(self) -> bool:
        return False

    @abstractmethod
    def operation(self) -> str:
        # 是所有组件必须实现的方法，定义了组件的具体操作行为
        pass


# 叶子节点
class Leaf(Component):
    """
    Leaf 类表示组合的最终对象（树结构中的叶子节点），叶子节点不能包含其他组件。
    通常，Leaf 对象会执行实际工作，而 Composite 对象只会委托给其子组件。
    """

    def operation(self) -> str:
        # 具体实现 operation() 方法，表示叶子节点的操作
        return "Leaf"


# 组合节点
class Composite(Component):
    """
    组合节点可以包含其他 Component 对象（既可以是叶子节点，也可以是其他组合节点）
    """

    def __init__(self) -> None:
        self._children: List[Component] = []

    # 实现了 add() 和 remove() 方法，用于管理子节点
    def add(self, component: Component) -> None:
        self._children.append(component)
        component.parent = self

    def remove(self, component: Component) -> None:
        self._children.remove(component)
        component.parent = None

    def is_composite(self) -> bool:
        return True

    def operation(self) -> str:
        # operation() 方法递归调用子节点的 operation() 方法，并返回组合的结果
        results = []
        for child in self._children:
            results.append(child.operation())
        return f"Branch({'+'.join(results)})"


def client_code(component: Component) -> None:
    # 演示如何在不知道组件类型的情况下执行操作。这展示了组合模式的强大之处，可以对叶子节点和组合节点进行统一处理
    print(f"RESULT: {component.operation()}", end="")


def client_code2(component1: Component, component2: Component) -> None:
    # 进一步展示了组合模式的灵活性，允许在运行时动态地将子组件添加到组合组件中
    if component1.is_composite():
        component1.add(component2)

    print(f"RESULT: {component1.operation()}", end="")


if __name__ == "__main__":
    # 单个叶子节点: 打印 Leaf，表示这是一个单独的叶子节点
    simple = Leaf()
    print("Client: I've got a simple component:")
    client_code(simple)
    print("\n")
    # 组合树结构: 打印 Branch(Branch(Leaf + Leaf) + Branch(Leaf))，表示一个组合结构，包含多个叶子节点
    tree = Composite()

    branch1 = Composite()
    branch1.add(Leaf())
    branch1.add(Leaf())

    branch2 = Composite()
    branch2.add(Leaf())

    tree.add(branch1)
    tree.add(branch2)

    print("Client: Now I've got a composite tree:")
    client_code(tree)
    print("\n")
    # 动态添加叶子节点: 打印 Branch(Branch(Leaf + Leaf) + Branch(Leaf) + Leaf)，展示如何在树结构中动态添加节点，并统一处理
    print("Client: I don't need to check the components classes even when managing the tree:")
    client_code2(tree, simple)

# 输出：
# Client: I've got a simple component:
# RESULT: Leaf
#
# Client: Now I've got a composite tree:
# RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))
#
# Client: I don't need to check the components classes even when managing the tree:
# RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)
```

### 19.10 外观模式（Facade Pattern）

外观模式（Facade Pattern）是一种结构型设计模式，旨在为复杂的子系统提供一个简单的接口，使客户端能够更容易地与子系统交互。外观模式通过封装复杂系统的内部实现，为客户端提供一个统一的接口，从而减少客户端与系统之间的耦合。

**外观模式的主要作用：**

* 简化接口：隐藏子系统的复杂性，提供简单易用的接口；
* 减少依赖：通过引入外观类，客户端不需要直接依赖于复杂系统的各个部分，从而减少了系统的耦合度；

**代码示例：**

```python
from __future__ import annotations


class Subsystem1:
    # 子系统 1：表示复杂系统中的子系统
    def operation1(self) -> str:
        return "Subsystem1: Ready!"

    # ... ...

    def operation_n(self) -> str:
        return "Subsystem1: Go!"


class Subsystem2:
    # 子系统 2：表示复杂系统中的子系统
    def operation1(self) -> str:
        return "Subsystem2: Ready!"

    # ... ...

    def operation_z(self) -> str:
        return "Subsystem2: Go!"


class Facade:
    """
    Facade 类封装了 Subsystem1 和 Subsystem2 的操作，对外提供一个统一的接口 operation，简化了子系统的操作流程
    """

    def __init__(self, subsystem1: Subsystem1, subsystem2: Subsystem2) -> None:
        self._subsystem1 = subsystem1 or Subsystem1()
        self._subsystem2 = subsystem2 or Subsystem2()

    def operation(self) -> str:
        # operation，简化了子系统的操作流程
        results = []
        results.append("Facade initializes subsystems:")
        results.append(self._subsystem1.operation1())
        results.append(self._subsystem2.operation1())
        results.append("Facade orders subsystems to perform the action:")
        results.append(self._subsystem1.operation_n())
        results.append(self._subsystem2.operation_z())
        return "\n".join(results)


def client_code(facade: Facade) -> None:
    """
    客户端代码通过 Facade 提供的简单接口与复杂的子系统协同工作。
    当 Facade 管理子系统的生命周期时，客户端可能根本不知道子系统的存在。 这种方法可以让复杂性得到控制。
    """
    print(facade.operation(), end="")


if __name__ == "__main__":
    subsystem1 = Subsystem1()
    subsystem2 = Subsystem2()
    facade = Facade(subsystem1, subsystem2)
    client_code(facade)

# 输出：
# Facade initializes subsystems:
# Subsystem1: Ready!
# Subsystem2: Ready!
# Facade orders subsystems to perform the action:
# Subsystem1: Go!
# Subsystem2: Go!
```

**扩展性：** 当子系统发生变化时，客户端不需要修改，只需调整 Facade 类的实现，从而有效地隔离了客户端与复杂系统的内部结构；

**外观模式 与 模板方法区别：**

* 外观模式：
  * 目的: 为复杂子系统提供一个统一的接口，简化客户端的使用；
  * 实现: 外观类封装子系统的细节，对外提供简化的接口；
  * 使用场景: 当系统过于复杂，需要对外提供一个简单接口时；
* 模板方法：
  * 目的: 定义算法的骨架，将一些步骤的实现延迟到子类；
  * 实现: 在抽象类中定义模板方法，在子类中实现具体步骤；
  * 使用场景: 多个类有相同的算法结构，但具体实现不同；
* 区别：
  * 使用意图: 外观模式简化接口，模板方法模式定义算法骨架；
  * 实现方式: 外观模式侧重封装子系统，模板方法模式侧重算法步骤的控制；

### 19.11 原型模式（Prototype Pattern）

原型模式（Prototype Pattern） 是一种创建型设计模式，它通过复制现有对象来创建新对象，而不是通过构造函数重新创建。这可以避免昂贵的对象创建操作，尤其在初始化代价高昂的情况下使用。

**核心思想：**

* 原型模式要求对象实现一个 clone() 方法，以便在不暴露复杂逻辑的情况下复制自身；
* 当需要大量相似对象时，可以使用原型模式减少创建开销；

**代码示例1：**

```python
import copy
from abc import ABC, abstractmethod


# 定义原型接口
class Prototype(ABC):

    @abstractmethod
    def clone(self):
        pass


# 具体原型类
class ConcretePrototype(Prototype):
    def __init__(self, value: str):
        self.value = value

    def clone(self):
        return copy.deepcopy(self)


# 客户端代码
if __name__ == "__main__":
    prototype1 = ConcretePrototype("prototype1")
    clone1 = prototype1.clone()
    print(f"Original: {prototype1.value}")
    print(f"Clone: {clone1.value}")
    clone1.value = "clone1"
    print(f"Original: {prototype1.value}")
    print(f"Clone: {clone1.value}")

# 输出：
# Original: prototype1
# Clone: prototype1
# Original: prototype1
# Clone: clone1
```

**说明：**

* 原型 （Prototype） 接口将对克隆方法进行声明。 在绝大多数情况下， 其中只会有一个名为 clone 克隆的方法；
* 具体原型 （Concrete Prototype） 类将实现克隆（clone）方法，除了将原始对象的数据复制到克隆体中之外，该方法有时还需处理克隆过程中的极端情况，例如克隆关联对象和梳理递归依赖等等。这里采用
  deepcopy 来确保深层次的复制；
* 客户端 （Client） 可以复制实现了原型接口的任何对象；

> **【扩展知识点：copy.deepcopy】**  
> deepcopy 是 Python 中 copy 模块提供的一个函数，它用于深度复制对象。与浅拷贝（copy.copy()
）不同，深拷贝会递归地复制对象及其包含的所有子对象。因此，原始对象和深拷贝对象完全独立，修改深拷贝对象不会影响原始对象。
>
> **浅拷贝 vs 深拷贝：**
> * 浅拷贝：只复制对象的引用，嵌套的子对象仍然引用同一块内存;
> * 深拷贝：不仅复制对象本身，还递归复制所有子对象;  
    > **示例：**
> ```python
> import copy
> 
> original = [1, [2, 3]]
> shallow_copy = copy.copy(original)
> deep_copy = copy.deepcopy(original)
> 
> # 修改嵌套列表
> original[1][0] = 100
> 
> print(original)      # [1, [100, 3]]
> print(shallow_copy)  # [1, [100, 3]] 受影响
> print(deep_copy)     # [1, [2, 3]] 不受影响
> 
>  ```

**代码示例2：**

这段示例代码展示了 Python 中如何通过 copy 模块实现自定义的浅拷贝和深拷贝。

```python
import copy


class SelfReferencingEntity:
    """
    表示一个包含父对象的类，可以设置和引用其他对象。
    """

    def __init__(self):
        self.parent = None

    def set_parent(self, parent):
        """
        设置父对象为传入的对象实例。
        """
        self.parent = parent


class SomeComponent:
    """
    一个复杂对象，包含整型值、对象列表和循环引用。
    该类通过自定义的 `__copy__` 和 `__deepcopy__` 来实现浅拷贝和深拷贝。
    """

    def __init__(self, some_int, some_list_of_objects, some_circular_ref):
        self.some_int = some_int  # 一个整型值
        self.some_list_of_objects = some_list_of_objects  # 一个包含对象的列表
        self.some_circular_ref = some_circular_ref  # 一个循环引用对象

    def __copy__(self):
        """
        实现浅拷贝：只复制对象的引用。
        当调用 copy.copy() 时，会调用此方法。
        """
        # 对嵌套的对象进行浅拷贝
        some_list_of_objects = copy.copy(self.some_list_of_objects)
        some_circular_ref = copy.copy(self.some_circular_ref)

        # 克隆对象并使用准备好的嵌套对象副本
        new = self.__class__(
            self.some_int, some_list_of_objects, some_circular_ref
        )
        new.__dict__.update(self.__dict__)  # 更新新对象的属性

        return new

    def __deepcopy__(self, memo=None):
        """
        实现深拷贝：递归复制对象及其嵌套对象。
        当调用 copy.deepcopy() 时，会调用此方法。
        memo 是一个防止循环引用的字典。
        """
        if memo is None:
            memo = {}

        # 对嵌套的对象进行深拷贝
        some_list_of_objects = copy.deepcopy(self.some_list_of_objects, memo)
        some_circular_ref = copy.deepcopy(self.some_circular_ref, memo)

        # 克隆对象并使用准备好的嵌套对象副本
        new = self.__class__(
            self.some_int, some_list_of_objects, some_circular_ref
        )
        new.__dict__ = copy.deepcopy(self.__dict__, memo)

        return new


if __name__ == "__main__":

    # 初始化对象及循环引用
    list_of_objects = [1, {1, 2, 3}, [1, 2, 3]]
    circular_ref = SelfReferencingEntity()
    component = SomeComponent(23, list_of_objects, circular_ref)
    circular_ref.set_parent(component)

    # 浅拷贝对象
    shallow_copied_component = copy.copy(component)

    # 修改浅拷贝中的列表，检查原对象是否受影响
    shallow_copied_component.some_list_of_objects.append("another object")
    if component.some_list_of_objects[-1] == "another object":
        print("浅拷贝的修改影响了原对象")
    else:
        print("浅拷贝的修改未影响原对象")

    # 修改集合，检查浅拷贝是否受影响
    component.some_list_of_objects[1].add(4)
    if 4 in shallow_copied_component.some_list_of_objects[1]:
        print("修改原对象集合影响了浅拷贝")
    else:
        print("修改原对象集合未影响浅拷贝")

    # 深拷贝对象
    deep_copied_component = copy.deepcopy(component)

    # 修改深拷贝中的列表，检查原对象是否受影响
    deep_copied_component.some_list_of_objects.append("one more object")
    if component.some_list_of_objects[-1] == "one more object":
        print("深拷贝的修改影响了原对象")
    else:
        print("深拷贝的修改未影响原对象")

    # 修改集合，检查深拷贝是否受影响
    component.some_list_of_objects[1].add(10)
    if 10 in deep_copied_component.some_list_of_objects[1]:
        print("修改原对象集合影响了深拷贝")
    else:
        print("修改原对象集合未影响深拷贝")

    # 打印循环引用的 ID，证明深拷贝处理了循环引用
    print(
        f"id(deep_copied_component.some_circular_ref.parent): "
        f"{id(deep_copied_component.some_circular_ref.parent)}"
    )
    print(
        f"id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent): "
        f"{id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent)}"
    )
    print("^^ 深拷贝处理了循环引用，并没有重复克隆。")

# 输出：
# 浅拷贝的修改影响了原对象
# 修改原对象集合影响了浅拷贝
# 深拷贝的修改未影响原对象
# 修改原对象集合未影响深拷贝
# id(deep_copied_component.some_circular_ref.parent): 140576636458848
# id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent): 140576636458848
# ^^ 深拷贝处理了循环引用，并没有重复克隆。
```

**代码解释：**

* SelfReferencingEntity 类  
  这个类的主要目的是展示循环引用问题，它拥有一个 parent 属性，表示该对象的父级。
  * set_parent 方法：设置当前对象的父级；

* SomeComponent 类  
  此类表示一个带有复杂结构的组件，其中包含一个整数、一个对象列表以及一个循环引用。此类通过实现 `__copy__` 和 `__deepcopy__`
  方法，展示如何自定义浅拷贝和深拷贝的行为。
  * 构造函数：接受一个整数、一个对象列表和一个循环引用对象作为参数。
  * `__copy__` 方法：实现浅拷贝，使用 copy.copy 对象的嵌套属性来创建新实例。
    * copy.copy(self.some_list_of_objects) 创建了对象列表的浅拷贝;
    * 最终返回的 new 是浅拷贝对象，且其属性是对原对象嵌套对象的引用;
  * `__deepcopy__` 方法：实现深拷贝，通过 copy.deepcopy 递归地复制嵌套对象，并通过 memo 参数避免循环引用问题。
    * memo 是一个字典，记录已经复制过的对象，防止无限递归；

* 测试部分  
  在 `__main__` 部分中，创建了一个嵌套对象 component 和 circular_ref，并分别展示了浅拷贝和深拷贝的行为差异。
  * 浅拷贝测试：
    * 修改 shallow_copied_component.some_list_of_objects 会影响原始对象 component，因为它们共享同一个引用；
    * 修改 component.some_list_of_objects[1] 中的集合也会影响浅拷贝对象，因为这只是浅层引用；
  * 深拷贝测试：
    * 修改 deep_copied_component.some_list_of_objects 不会影响原对象 component，因为深拷贝创建了完全独立的副本；
    * 循环引用测试：deep_copied_component 的 parent 属性中的引用保持不变，避免了无限循环；

**主要原理：**

* 浅拷贝 (__copy__)：
  * 拷贝对象本身，但嵌套的对象（如列表、集合等）仍与原对象共享；
  * 修改嵌套对象会影响拷贝和原对象；
* 深拷贝 (__deepcopy__)：
  * 递归复制对象及其嵌套的对象，确保它们独立；
  * 使用 memo 参数避免循环引用导致的无限递归；
* 循环引用：
  * 深拷贝处理了循环引用的情况，通过 memo 确保不会无限递归；
  * 此代码展示了原型模式如何通过浅拷贝和深拷贝来复制复杂对象，同时处理嵌套对象和循环引用问题；

### 19.12 状态模式（State Pattern）

状态模式是一种行为设计模式，允许对象在内部状态改变时改变其行为。这种模式对于对象在不同状态下有不同行为的场景非常有用。它通过将状态的行为分离到独立的类中，从而简化了状态的管理，避免了大量的条件分支。

**核心概念：**

* Context: 状态的管理对象，维护当前状态并将行为委托给状态对象；
* State: 表示对象的状态，定义状态下的行为；
* Concrete States: 具体状态的实现类；

**代码示例：**

```python
from abc import ABC, abstractmethod


class State(ABC):
    """
    定义状态类接口，所有具体状态类需要实现此接口的行为
    """

    @abstractmethod
    def handle(self, context):
        pass


class ConcreteStateReady(State):
    """
    具体状态 Ready 的行为实现
    """

    def handle(self, context):
        print("State Ready: 处理请求，切换到 State Start")
        context.set_state(ConcreteStateStart())


class ConcreteStateStart(State):
    """
    具体状态 Start 的行为实现
    """

    def handle(self, context):
        print("State Start: 处理请求，切换到 State End")
        context.set_state(ConcreteStateEnd())


class ConcreteStateEnd(State):
    """
    具体状态 End 的行为实现
    """

    def handle(self, context):
        print("State End: 处理请求，切换到 State Ready")
        context.set_state(ConcreteStateReady())


class Context:
    """
    上下文类，持有状态，并将请求委托给当前状态处理
    """

    def __init__(self, state: State):
        self._state = state  # 初始化时设置初始状态

    def set_state(self, state: State):
        """
        改变当前状态
        """
        self._state = state

    def request(self):
        """
        将请求委托给当前状态对象处理
        """
        self._state.handle(self)


# 使用状态模式
if __name__ == "__main__":
    context = Context(ConcreteStateReady())  # 设置初始状态为 State Ready
    context.request()  # State Ready: 切换到 State Start
    context.request()  # State Start: 切换到 State End
    context.request()  # State End: 切换到 State Ready

# 输出：
# State Ready: 处理请求，切换到 State Start
# State Start: 处理请求，切换到 State End
# State End: 处理请求，切换到 State Ready
```

**解释：**

* Context 类负责维护当前状态，并在需要时切换状态；
* State 类是状态的抽象基类，具体状态类继承它并实现各自的 handle() 方法；
* ConcreteStateA 和 ConcreteStateB 是两个具体状态，它们的 handle() 方法执行特定操作并切换到另一个状态；

通过这种设计，状态的变化是动态的，避免了复杂的 if-else 或 switch 语句，并且每个状态的行为都被封装在对应的类中，使得代码更加清晰和可扩展。

### 19.x 更多设计模式

本站包含各种设计模式及用例（创建新模式、结构性模式、行为模式），后续慢慢补充：https://refactoringguru.cn/design-patterns/python

## 20. 进程（Process）、线程（Thread）、协程（Coroutine）

### 20.1 进程（Process）

进程是操作系统分配资源的基本单位，每个进程拥有独立的内存空间。一个程序运行后至少会有一个主进程，主进程可以派生出多个子进程。

参考：https://bbs.huaweicloud.com/blogs/289316

**特点：**

* 独立性：每个进程有自己独立的内存空间，不会与其他进程共享数据；
* 多核并行：可以充分利用多核 CPU 进行真正的并行计算；
* 隔离性好：进程之间互相隔离，崩溃的进程不会影响其他进程；

**缺点：**

* 创建开销大：进程创建的开销比线程和协程更大，因为需要分配独立的内存空间；
* 进程间通信复杂：由于进程间不共享内存，需要通过 IPC (Inter-Process Communication) 进行通信，比如管道 (Pipe)、消息队列 (
  Queue)、共享内存 (Shared Memory) 等；

**使用场景：**

* CPU 密集型任务，如图像处理、视频编码等需要多核并行计算的任务；

**代码示例：**

```python
import multiprocessing


def worker(num):
    print(f'Worker: {num}')


if __name__ == '__main__':
    processes = []
    for i in range(5):
        p = multiprocessing.Process(target=worker, args=(i,))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()
```

在 Python 3 中，multiprocessing 模块提供了丰富的 API 来进行进程管理和进程间通信。以下是常用的进程使用用例，包括进程池 (
Pool)、子进程 (Process) 以及进程间通信 (Queue, Pipe, Manager)。

官方
3.12.6：[https://docs.python.org/zh-cn/3/library/multiprocessing.html](https://docs.python.org/zh-cn/3/library/multiprocessing.html)

#### 20.1.1 子进程 (Process)

在 Python 中，当你运行一个脚本时，该脚本的执行进程就是所谓的“主进程”。当主进程使用 multiprocessing
模块创建子进程时，子进程会作为主进程的子进程运行。这种父子关系意味着主进程负责管理和控制子进程的生命周期。

**主进程与子进程的关系：**

* 主进程：这是运行脚本时操作系统分配的最初进程。主进程负责创建和管理子进程，并等待子进程完成或终止它们；
  * 等待子进程完成 (join())，终止子进程 (terminate())，也可以不等待子进程而继续执行其他任务（不使用 join()）
* 子进程：由主进程创建的进程，子进程在独立的内存空间中运行，并且可以执行与主进程不同的代码片段。子进程运行在自己的上下文中，有自己独立的进程
  ID；

**Process 对象配置参数：**  
在 multiprocessing 模块中，Process 类用于创建子进程。创建子进程时，可以通过以下参数配置子进程的行为：

* target：这是子进程要执行的目标函数。在创建子进程时，target 参数指定函数的引用，子进程启动后将执行该函数的内容；
* args：这是传递给目标函数的参数。它需要是一个元组，即使目标函数只有一个参数，也需要在参数后加一个逗号，例如 args=(i,)；
* kwargs：可以传递给目标函数的关键字参数，使用字典格式；
* name：为子进程指定一个名字。默认情况下，进程会自动分配一个名字，如 Process-1；
* daemon：设定是否为守护进程。守护进程会在主进程结束时自动终止。daemon=True 将进程设为守护进程；

**子进程方法：**

* start()：启动子进程，调用 start() 方法后，子进程会在后台运行，执行指定的 target 函数。
* join()：阻塞主进程，直到调用该方法的子进程结束。join() 方法常用于确保主进程等待所有子进程完成后再继续执行。join() 可以设置
  timeout 参数来指定等待时间。
* is_alive()：检查子进程是否还在运行，返回 True 或 False。
* terminate()：立即终止子进程。这通常用于在异常情况下强制停止子进程。

**示例说明：**

以下是一个使用 multiprocessing 创建和管理子进程的简单示例，包含对上述方法的应用：

```python
import multiprocessing
import os
import time


def worker(num):
    print(
        f'Worker {num} started, Process ID: {multiprocessing.current_process().pid}, Name:{multiprocessing.current_process().name}')
    time.sleep(2)
    print(f'Worker {num} finished')


if __name__ == '__main__':
    print(f'Main Process, Process ID:{os.getpid()}, Name:{multiprocessing.current_process().name}')
    num_processes = multiprocessing.cpu_count()  # 获取CPU核心数量
    processes = []
    for i in range(num_processes):
        # 创建子进程，每个子进程执行 worker 函数
        # p = multiprocessing.Process(target=worker, args=(i,), daemon=True)
        p = multiprocessing.Process(target=worker, args=(i,))
        processes.append(p)
        # 使用 p.start() 方法启动子进程，子进程开始在后台运行
        p.start()

    for p in processes:
        # p.join() 方法阻塞主进程，直到对应的子进程执行完毕。这确保了主进程在所有子进程结束后再继续运行后续代码。
        p.join()  # 主进程将在此处等待所有子进程完成
    # 所有子进程执行完后，主进程输出 "All processes finished"
    print('All processes finished')
```

输出：主进程将等待子进程结束后输出 “All processes finished”

```text
Main Process, Process ID:43716, Name:MainProcess
Worker 0 started, Process ID: 43717, Name:Process-1
Worker 1 started, Process ID: 43718, Name:Process-2
Worker 2 started, Process ID: 43719, Name:Process-3
Worker 0 finished
Worker 1 finished
Worker 2 finished
All processes finished
```

控制台进程查看：可以看到上述输出进程 PID 与终端输出一致，包括主进程 & 子进程总共运行 4 个进程。

```text
jpzhang     43716  0.4  0.0  23520 13488 ?        S    10:01   0:00 /home/jpzhang/workspace/py-env/py3.12-dev-env/bin/python3.12 /home/jpzhang/workspace/examples/python-tricks/src/process_demo20_01.py
jpzhang     43717  0.0  0.0  23520  9668 ?        S    10:01   0:00 /home/jpzhang/workspace/py-env/py3.12-dev-env/bin/python3.12 /home/jpzhang/workspace/examples/python-tricks/src/process_demo20_01.py
jpzhang     43718  0.0  0.0  23520  9668 ?        S    10:01   0:00 /home/jpzhang/workspace/py-env/py3.12-dev-env/bin/python3.12 /home/jpzhang/workspace/examples/python-tricks/src/process_demo20_01.py
jpzhang     43719  0.0  0.0  23520  9668 ?        S    10:01   0:00 /home/jpzhang/workspace/py-env/py3.12-dev-env/bin/python3.12 /home/jpzhang/workspace/examples/python-tricks/src/process_demo20_01.py
```

若注释 p.join() 相关代码，主进程将不会等待子进程结束，而是继续运行，输出类似如下：

```text
Main Process, Process ID:44231, Name:MainProcess
All processes finished
Worker 0 started, Process ID: 44232, Name:Process-1
Worker 1 started, Process ID: 44233, Name:Process-2
Worker 2 started, Process ID: 44234, Name:Process-3
Worker 0 finished
Worker 1 finished
Worker 2 finished
```

若将子进程设置为守护进程：multiprocessing.Process(target=worker, args=(i,), daemon=True)
，则主进程结束时子进程自动终止（前提是 "注释 p.join() 相关代码"，避免主进程等待子进程结束）。

```text
Main Process, Process ID:44367, Name:MainProcess
Worker 0 started, Process ID: 44368, Name:Process-1
All processes finished

进程已结束，退出代码为 0
```

主进程结束，守护进程自动终止。

#### 20.1.2 进程池 (Pool)

#### 20.1.2.1 进程池 (multiprocessing.Pool)

在 Python 中，multiprocessing.Pool 提供了一种简单的方式来管理进程池，以并发地执行任务。进程池允许你预先创建一组工作进程，并通过这些进程来执行多个任务，避免频繁地创建和销毁进程所带来的开销。Pool
对象支持多种方法来分发任务，包括同步和异步方式。

**示例：**

```python
import multiprocessing
import os


def worker(num):
    print(f'Worker: {num}, PID: {os.getpid()}')
    return num * num


if __name__ == '__main__':
    # 创建一个进程池，最多允许 3 个进程同时执行
    with multiprocessing.Pool(processes=3) as pool:
        # map(worker, range(5)):将任务分配给进程池中的工作进程并行执行，map 方法返回一个列表，其中包含 worker 函数的返回值。
        results = pool.map(worker, range(5))
    print(results)
```

以下通过示例分别介绍 Pool 多种分发任务方式。

#### 20.1.2.1.1 apply & apply_async

创建进程池:

```python
from multiprocessing import Pool

# 创建一个进程池，指定池中的进程数量
pool = Pool(processes=4)
```

【注意】processes 参数指定了进程池中的进程数。如果省略，默认会使用 os.cpu_count() 来确定。

* apply 和 apply_async
  * apply(func, args=(), kwargs={})：同步执行指定的函数 func，并传入 args 和 kwargs。执行完毕后，返回函数的结果。这个方法类似于直接调用函数，但是在进程池中执行；
  * apply_async(func, args=(), kwargs={}, callback=None, error_callback=None)：异步执行指定的函数 func。它立即返回一个
    ApplyResult 对象，可以通过 get() 方法来获取结果。callback 是一个可选的回调函数，用于处理结果，error_callback 用于处理异常。

**示例代码：**

```python
import multiprocessing
import time


def square(x):
    time.sleep(20)
    return x * x


# __name__ == '__main__' 确保只有在直接运行脚本时才会执行主程序块。这是 Python 中防止在 Windows 平台上重复创建进程的必要条件
if __name__ == '__main__':
    # 创建一个包含 4 个工作进程的进程池（Pool 对象），进程池将管理和分发任务到这些进程中。
    pool = multiprocessing.Pool(processes=4)

    # 同步调用：
    # apply 方法将任务 square(10) 分派给一个进程，主进程会等待任务完成后再继续执行
    # 由于 square 函数内部有 time.sleep(20)，因此主进程会等待 20 秒
    # result 保存 square(10) 的返回值，也就是 100
    result = pool.apply(square, args=(10,))
    print(f'Synchronous result: {result}')

    # 异步调用：
    # apply_async 方法将任务 square(10) 分派给一个进程，但不会阻塞主进程，主进程可以继续执行后续代码
    # async_result 是一个 ApplyResult 对象，表示异步调用的返回值
    async_result = pool.apply_async(square, args=(10,))
    # async_result.get() 会阻塞主进程，直到异步任务完成并返回结果
    # 因为 square(10) 需要 20 秒来完成，所以主进程在这行代码上会等待任务完成并获取结果 100
    print(f'Asynchronous result: {async_result.get()}')

    # 调用 join 之前，先调用 close 函数，否则会出错。执行完 close 后不会有新的进程加入到 pool,join 函数等待所有子进程结束
    # 关闭进程池，表示不再接受新的任务：当进程池 close 的时候并未关闭进程池，只是会把状态改为不可再插入元素的状态
    pool.close()
    # 等待进程池中所有任务完成。在此之后，主进程才会继续执行后面的代码（如果有的话）
    pool.join()

# 输出：
# Synchronous result: 100
# Asynchronous result: 100
```

本示例，异步调用不会阻塞主进程，但在 get() 方法调用时还是会等待任务完成。如果希望在等待结果的同时执行其他任务，可以在调用
get() 之前执行更多的代码。

**apply_async 与 apply 示例如下：**

`apply_async(func, args=(), kwargs={}, callback=None, error_callback=None)：`

```python
import multiprocessing
import time


def func(msg):
    print("msg:", msg)
    time.sleep(3)
    print("End Process")


if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=3)
    for i in range(4):
        msg = f"Hello {i}"
        pool.apply_async(func, (msg,))
    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    pool.join()
    print("Sub-process(es) done.")
# 输出：
# Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
# msg: Hello 0
# msg: Hello 1
# msg: Hello 2
# End Process
# msg: Hello 3
# End Process
# End Process
# End Process
# Sub-process(es) done.
```

apply_async 函数允许你传递 callback 和 error_callback 参数。callback 参数用于指定任务成功完成后的回调函数，而
error_callback 参数用于处理任务执行过程中出现的异常。下面是一个使用 apply_async 的 callback 和 error_callback 参数的示例。

```python
import multiprocessing


def safe_divide(x, y):
    # 这个函数接收两个参数 x 和 y，并返回 x / y 的结果。如果 y 为 0，则抛出 ValueError 异常
    if y == 0:
        raise ValueError("Division by zero!")
    return x / y


def on_success(result):
    # 当 apply_async 成功完成任务时，on_success 回调函数会被调用，并接收任务的结果作为参数
    print(f'Success! Result: {result}')


def on_error(e):
    # 当 apply_async 在任务执行过程中遇到异常时，on_error 回调函数会被调用，并接收异常信息作为参数
    print(f'Error: {e}')


if __name__ == '__main__':
    pool = multiprocessing.Pool(processes=4)

    # 使用 apply_async 调用函数，传递成功回调和错误回调
    pool.apply_async(safe_divide, args=(10, 2), callback=on_success,
                     error_callback=on_error)  # 传递 (10, 2) 作为参数，任务成功后，on_success 会被调用
    pool.apply_async(safe_divide, args=(10, 0), callback=on_success,
                     error_callback=on_error)  # 传递 (10, 0) 作为参数，由于 y 为 0，safe_divide 函数会抛出异常，on_error 回调函数会被调用

    pool.close()
    pool.join()

    print("Main process finished.")

# 输出：
# Success! Result: 5.0
# Error: Division by zero!
# Main process finished.
```

* `callback`：当任务成功完成时，callback 函数会被调用，并接收任务的返回值;
* `error_callback`：当任务执行过程中出现异常时，error_callback 函数会被调用，并接收异常对象;

`apply(func, args=(), kwargs={})：`

```python
import multiprocessing
import time


def func(msg):
    print("Msg：", msg)
    time.sleep(3)
    print("End Process")


if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=3)
    for i in range(4):
        msg = f"Hello, {i}"
        # #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去
        pool.apply(func, (msg,))
    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    pool.join()  # 调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    print("Sub-process(es) done.")

# 输出：
# Msg： Hello, 0
# End Process
# Msg： Hello, 1
# End Process
# Msg： Hello, 2
# End Process
# Msg： Hello, 3
# End Process
# Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
# Sub-process(es) done.
```

**总结：**

* 同步调用：主进程在 apply 方法上会阻塞，直到任务完成，适用于不需要并行的简单任务；
* 异步调用：主进程不会在 apply_async 方法上阻塞，而是继续执行后续代码，适用于需要并行处理的复杂任务；

#### 20.1.2.1.2 map & map_async

* `map(func, iterable, chunksize=None)`：同步调用，将 iterable 中的每一个元素作为参数，依次传递给函数
  func，以并行的方式计算，并返回结果列表。map 是阻塞的，即主进程会等待所有子进程完成。
  * func：要应用到每个元素的函数；
  * iterable：要迭代的对象，每个元素都会作为参数传递给 func；
  * chunksize（可选）：将 iterable 切分为更小的块来分发给进程池中的进程，有助于优化性能；
* `map_async(func, iterable, chunksize=None, callback=None, error_callback=None)`：map 的异步版本。立即返回 ApplyResult
  对象，结果可以通过 get() 获取。
  * func：要应用到每个元素的函数；
  * iterable：要迭代的对象；
  * chunksize（可选）：将 iterable 切分为更小的块；
  * callback（可选）：任务完成时的回调函数，该函数接受一个包含结果的列表作为参数；
  * error_callback（可选）：任务失败时的回调函数，该函数接受一个异常对象作为参数；

**示例代码：**

```python
import multiprocessing
import time


def square(param):
    time.sleep(3)
    print(f"My name is: {param['name']}")
    return param['mark'] * param['mark']


if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=3)
    # 使用 map：同步调用，将 square 函数应用到列表 [{"name":"Stars","mark":1},{"name":"Active","mark":2},{"name":"Absurd","mark":3},{"name":"Fairy","mark":4}] 上
    results = pool.map(square,
                       [{"name": "Stars", "mark": 1},
                        {"name": "Active", "mark": 2},
                        {"name": "Absurd", "mark": 3},
                        {"name": "Fairy", "mark": 4}])
    print(f'Synchronous map result: {results}')  # 输出: [1, 4, 9, 16]

    # 使用 map_async：异步调用
    async_results = pool.map_async(square, [{"name": "Stars", "mark": 1},
                                            {"name": "Active", "mark": 2},
                                            {"name": "Absurd", "mark": 3},
                                            {"name": "Fairy", "mark": 4}])
    print(f'Asynchronous map result: {async_results.get()}')  # 输出: [1, 4, 9, 16]
    pool.close()
    pool.join()

# 输出：
# My name is: Stars
# My name is: Active
# My name is: Absurd
# My name is: Fairy
# Synchronous map result: [1, 4, 9, 16]
# My name is: Stars
# My name is: Active
# My name is: Absurd
# My name is: Fairy
# Asynchronous map result: [1, 4, 9, 16]
```

* `map`：同步调用，square
  函数接受参数 [{"name":"Stars","mark":1},{"name":"Active","mark":2},{"name":"Absurd","mark":3},{"name":"Fairy","mark":4}]
  ，返回 [1, 4, 9, 16]；
* `map_async`：异步调用，结果同样是 [1, 4, 9, 16]，但可以通过 get() 方法获取结果；

map_async 的 callback 参数允许你指定一个回调函数，该函数会在所有异步任务完成后自动调用，并且会接收到任务的结果列表作为参数。下面是一个使用
map_async 的回调函数的示例。

```python
import multiprocessing
import time


def square(x):
    # 模拟一个耗时操作
    time.sleep(3)
    return x * x


def collect_result(result):
    # 回调函数
    # 当 map_async 的所有任务完成后，collect_result 函数会自动被调用，并接收 map_async 的结果作为参数
    print(f"Results collected: {result}")


if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=3)
    # 异步调用 map_async，传递回调函数 collect_result,当所有任务完成时，collect_result 回调函数会被调用，并打印出所有结果
    async_results = pool.map_async(square, [1, 2, 3, 4], callback=collect_result)
    # 在这里可以做其他的事情, 因为 map_async 是异步的，这个 print 可能在 collect_result 之前或之后执行
    print("Doing other work while waiting for results...")
    # 等待所有的子进程完成
    pool.close()
    pool.join()
    print("Main process finished.")

# 由于 map_async 是异步调用，print("Doing other work while waiting for results...") 可能会在回调函数执行前打印，说明主进程并不会因为等待 map_async 的结果而被阻塞。
# 例如，注释 square 函数: time.sleep(3), time.sleep(3) 添加到 async_results = pool.map_async() 之后，模拟主进程处理其他任务
# 输出：
# Doing other work while waiting for results...
# Results collected: [1, 4, 9, 16]
# Main process finished.
```

#### 20.1.2.1.3 starmap & starmap_async

* `starmap(func, iterable, chunksize=None)`：同步调用,类似于 map，但 iterable 中的元素是元组，它们会被拆包并作为多个参数传递给函数
  func;
* `starmap_async(func, iterable, chunksize=None, callback=None, error_callback=None)`：starmap 的异步版本。返回
  ApplyResult 对象，结果可以通过 get() 方法获取;

**starmap 示例**

`starmap` 是同步的，阻塞主进程直到所有任务完成，它将每个参数元组解包，然后传递给目标函数。

```python
import multiprocessing


def add(x, y):
    return x + y


if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=3)
    # starmap 将参数元组列表 (1, 2)、(3, 4) 和 (5, 6) 传递给 add 函数，返回 [3, 7, 11]
    results = pool.starmap(add, [(1, 2), (3, 4), (5, 6)])
    pool.close()
    pool.join()
    print(f'Synchronous starmap results: {results}')  # 输出：Synchronous starmap results: [3, 7, 11]
```

**starmap_async 示例**

```python
import multiprocessing


def safe_divide(x, y):
    # 尝试除法运算，并在除数为零时抛出 ValueError
    if y == 0:
        raise ValueError("Division by zero!")
    return x / y


def on_success(results):
    # 当所有任务都成功完成时，这个回调函数会被调用，接收结果列表
    print(f'Success! Results: {results}')


def on_error(e):
    # 当任何任务发生异常时，错误回调函数会被调用，并接收异常对象
    print(f'Error: {e}')


if __name__ == '__main__':
    pool = multiprocessing.Pool(processes=4)

    # 使用 starmap_async 将参数元组 (10, 2)、(8, 4)、(5, 0) 和 (7, 1) 异步传递给 safe_divide 函数
    async_result = pool.starmap_async(safe_divide, [(10, 2), (8, 4), (5, 0), (7, 1)],
                                      callback=on_success,
                                      error_callback=on_error)

    pool.close()
    pool.join()

    print("Main process finished.")

# 输出：
# Error: Division by zero!
# Main process finished.
```

【重要提示】：callback 仅在所有任务成功完成时才会调用，而 error_callback 在任一任务失败时触发。因此，不能同时期望 callback 和
error_callback 在同一批任务中都被调用。

#### 20.1.2.1.4 总结

* 同步 vs 异步：
  * apply, map, starmap 是同步的，主进程会等待任务完成；
  * apply_async, map_async, starmap_async 是异步的，主进程可以继续其他任务；

* 单个 vs 多个参数：
  * apply, apply_async 处理单个参数；
  * map, map_async 处理单个参数的 iterable；
  * starmap, starmap_async 处理多个参数的 iterable，每个元素是参数元组；

回调函数：

* 异步方法可以接受 callback 和 error_callback，处理任务成功或失败后的操作；

#### 20.1.2.2 进程池 (concurrent.futures.ProcessPoolExecutor)
  TODO

#### 20.1.3 进程间通信（Inter-Process Communication, IPC）

进程间通信（Inter-Process Communication, IPC）是指不同进程之间交换数据的机制。在 Python 的 multiprocessing 模块中，常用的
IPC 方式包括 Queue、Pipe 和 Manager，它们可以帮助不同进程之间安全、有效地传递数据。下面详细介绍它们的使用和特点。

#### 20.1.3.1 队列 (Queue)

Queue 是一种线程和进程安全的队列，用于在线程和进程之间传递数据。它基于先进先出（FIFO）的原则工作，支持多个生产者和多个消费者。
在进程模式下，Python 提供了 multiprocessing.Queue，它是通过底层的管道（Pipe）和锁（Lock）机制实现的安全队列，可以在不同的进程间传递数据。

**特点：**

* 线程（queue.Queue，下文介绍线程在展开说明）和进程（multiprocessing.Queue）安全：Queue 是线程和进程安全的，支持多生产者和多消费者模式；
* 阻塞与非阻塞操作：Queue 的 get() 和 put() 方法可以设置为阻塞或非阻塞模式；
* 容量限制：可以设置 Queue 的最大容量，默认无限制；

**示例：**

```python
from multiprocessing import Process, Queue
import time


# 定义一个函数，向队列中添加数据
def producer(queue):
    for i in range(10):
        item = f'Item {i}'
        print(f'生产者正在生产: {item}')
        queue.put(item)  # 将数据放入队列, 如果队列已满，将阻塞
        time.sleep(1)  # 模拟生产过程中的延迟


# 定义一个函数，从队列中取出数据
def consumer(queue):
    while True:
        item = queue.get()  # 从队列中获取数据
        if item is None:
            break  # 如果获取到 None，则退出循环
        print(f'消费者正在消费: {item}')
        time.sleep(2)  # 模拟消费过程中的延迟


if __name__ == '__main__':
    # 创建一个进程间通信且最大长度为 3 的队列
    queue = Queue(maxsize=3)

    # 创建生产者进程
    producer_process = Process(target=producer, args=(queue,))

    # 创建消费者进程
    consumer_process = Process(target=consumer, args=(queue,))

    # 启动进程
    producer_process.start()
    # 等待生产者产生数据，模拟队列阻塞
    time.sleep(5)
    consumer_process.start()

    # 等待生产者进程结束
    producer_process.join()

    # 生产结束后向队列中放入 None，用于通知消费者结束
    queue.put(None)

    # 等待消费者进程结束
    consumer_process.join()

    print('生产和消费过程完成。')
```

#### 1). 代码说明

* Queue 初始化:
    ```python
    queue = Queue()
    ```
  Queue() 创建了一个共享的队列，可以在多个进程间传递数据。

* 生产者函数 producer:
    ```python
    def producer(queue):
        for i in range(5):
            item = f'Item {i}'
            print(f'生产者正在生产: {item}')
            queue.put(item)
            time.sleep(1)
    ```
  该函数向队列中添加 5 个项目。每生产一个项目，程序会暂停 1 秒，以模拟生产过程中的延迟。

* 消费者函数 consumer:
    ```python
    def consumer(queue):
        while True:
            item = queue.get()
            if item is None:
                break
            print(f'消费者正在消费: {item}')
            time.sleep(2)
    ```
  该函数不断从队列中读取数据，并消费数据。读取数据后，程序暂停 2 秒以模拟消费过程中的延迟。如果读取到 None，则退出循环。

* 进程启动和同步:
    ```python
    producer_process.start()
    consumer_process.start()
    
    producer_process.join()
    queue.put(None)
    consumer_process.join()
    ```
  * start() 用于启动进程;
  * join() 等待进程完成;
  * queue.put(None) 传递一个 None 值，告诉消费者数据已经全部处理完毕，可以停止消费;
  * 若 `consumer_process.join()` 放置在 `queue.put(None)` 之前则消费者进程（consumer_process）一直不会结束，一直在等待进程完成（进程完成需要队列中获取
    None）。

* 输出: 运行该代码时，生产者和消费者进程会并行工作，生产者生成的数据会被消费者消耗，直到所有数据都处理完毕。
* 其他说明
  * 线程安全：multiprocessing.Queue 是线程和进程安全的，意味着在多线程或多进程环境下使用时，不需要额外的同步机制；
  * 数据序列化：Queue 会自动序列化和反序列化数据，所以可以传递任何可以被 pickle 模块序列化的数据类型；

#### 2). 参数及方法说明

* `multiprocessing.Queue(maxsize)`（设置队列长度）  
  通过 multiprocessing.Queue(maxsize) 可以设置队列的最大长度。这个参数限制了队列中能存放的最大项数，防止队列无限制地增长导致内存耗尽。
  * `maxsize`: 用于指定队列的最大长度。默认值为 0，表示队列大小不受限制，允许存储无限数量的数据。当队列达到 maxsize 时，put()
    操作将阻塞，直到队列有空间。

* `put(item, block=True, timeout=None)`  
  `put()` 方法将数据放入队列中。它支持阻塞模式，意味着如果队列已满，则可以等待直到队列有空余空间。
  * 参数：
    * `item`: 需要放入队列的数据;
    * `block（默认值为 True）`：如果设置为 True，当队列已满时，该方法会阻塞，直到有空闲空间。如果为 False，则在队列满时会立即抛出
      queue.Full 异常;
    * `timeout（可选）`：等待队列有空闲空间的时间。如果超过这个时间还没有空闲空间，会抛出 queue.Full 异常;
  * 用法：
    ```python
    queue.put("Hello, world!", block=True, timeout=None)
    ```
  * 示例：
    ```python
    import multiprocessing
    import time
    
    def producer(queue):
        # 往队列产生5条数据
        for i in range(5):
            item = f"Putting item {i} in queue"
            print(item)
            # queue.put(item)
            # 队列阻塞，抛出 queue.Full 异常
            queue.put(item, block=False)
            # 模拟任务耗时
            time.sleep(1)

    if __name__ == "__main__":
        # 创建一个长度为 3 的队列
        q = multiprocessing.Queue(maxsize=3)
        p = multiprocessing.Process(target=producer, args=(q,))
        p.start()  # 启动子进程，子进程开始在后台运行
        p.join()  # 等待子进程结束
    
    # 输出：
    # Putting item 0 in queue
    # Putting item 1 in queue
    # Putting item 2 in queue
    # Putting item 3 in queue
    # Process Process-1:
    # Traceback (most recent call last):
    #   File "/usr/lib/python3.12/multiprocessing/process.py", line 314, in _bootstrap
    #     self.run()
    #   File "/usr/lib/python3.12/multiprocessing/process.py", line 108, in run
    #     self._target(*self._args, **self._kwargs)
    #   File "/home/bolean/workspace/examples/python-tricks/src/process_queue_demo20_04.py", line 61, in producer
    #     queue.put(item, block=False)
    #   File "/usr/lib/python3.12/multiprocessing/queues.py", line 90, in put
    #     raise Full
    # queue.Full
    ```

    队列阻塞，抛出 queue.Full 异常。

* `get(block=True, timeout=None)`  
  `get()` 方法从队列中取出数据。如果队列为空时，它可以阻塞进程，直到有数据可取。
  * 参数：
    * `block（默认值为 True）`：如果设置为 True，当队列为空时，该方法会阻塞，直到有数据可获取。如果设置为 False，队列为空时会立即抛出
      queue.Empty 异常；
    * `timeout（可选）`：阻塞等待的最大时间。如果超过此时间仍然无法获取到数据，会抛出 queue.Empty 异常；
  * 用法：
    ```python
    item = queue.get()  # 阻塞模式，等待直到队列中有数据
    ```
  * 示例：
    ```python
    import multiprocessing
    import time
    
    
    def consumer(queue):
        while True:
            # item = queue.get()
            item = queue.get(block=False)  # 设置为 False，队列为空时会立即抛出 queue.Empty 异常
            if item is None:  # 当获取到 None 时，停止循环
                break
            print(f"Got item: {item}")
            # time.sleep(2)
    
    
    if __name__ == "__main__":
        q = multiprocessing.Queue(maxsize=3)
        p = multiprocessing.Process(target=consumer, args=(q,))
        p.start()
        for i in range(5):
            q.put(i)
            time.sleep(2)  # 模拟队列为空情况
        q.put(None)  # 向队列发送结束信号
        p.join()
    
    # 输出：
    # Got item: 0
    # Process Process-1:
    # Traceback (most recent call last):
    #   File "/usr/lib/python3.12/multiprocessing/process.py", line 314, in _bootstrap
    #     self.run()
    #   File "/usr/lib/python3.12/multiprocessing/process.py", line 108, in run
    #     self._target(*self._args, **self._kwargs)
    #   File "/home/bolean/workspace/examples/python-tricks/src/process_queue_demo20_04.py", line 99, in consumer
    #     item = queue.get(block=False)  # 设置为 False，队列为空时会立即抛出 queue.Empty 异常
    #            ^^^^^^^^^^^^^^^^^^^^^^
    #   File "/usr/lib/python3.12/multiprocessing/queues.py", line 116, in get
    #     raise Empty
    # _queue.Empty
    ```
    队列为空，抛出 queue.Empty 异常。

* `empty()`  
  `empty()` 方法，如果队列为空返回 True ，否则返回 False。需要注意的是，在多进程环境下，由于进程间的竞争，该方法的结果可能不完全可靠。
  ```python
  if queue.empty():
      print("队列为空")
  ```
  见 full() 示例。

* `full()`  
  `full()` 方法，如果队列已满返回 True ，否则返回 False。与 empty() 类似，在多进程环境下，这个方法的结果也可能不是完全可靠的。
  ```python
  if queue.full():
      print("Queue is full")
  ```

  示例：
  ```python
  import multiprocessing
  import time
  
  def producer(queue):
      for i in range(5):
          if queue.empty():
              print("队列为空！")
          if queue.full():
              print("队列已满！")
          print(i)
          queue.put(i)
          # time.sleep(2)
  
  if __name__ == "__main__":
      q = multiprocessing.Queue(maxsize=3)
      p = multiprocessing.Process(target=producer, args=(q,))
      p.start()
      p.join()
  
  # 输出：
  # 队列为空！
  # 0
  # 队列为空！
  # 1
  # 2
  # 队列已满！
  # 3
  
  # time.sleep(2)
  # 暂停，则程序正常输出：
  # 队列为空！
  # 0
  # 1
  # 2
  # 队列已满！
  # 3
  ```
  这里为什么会输出 2 次“队列为空！”？（注，环境差异，输出或有不同）
  * 初始队列为空： 当 producer 函数刚开始运行时，队列是空的，因此第一次调用 queue.empty() 时，输出“队列为空！”;
  * 队列为空的并发条件： 虽然在循环中不断向队列中添加元素，但 queue.empty() 是基于队列的当前状态检查是否为空。当调用
    queue.empty() 时，如果这时 CPU 正好切换到另一个线程，导致元素还未真正放入队列，可能会再次检查到队列为空。因此有可能出现第二次输出“队列为空！”的情况;

  #### 【扩展知识点】：queue.empty() 和 queue.full() 非可靠性
  * `multiprocessing.Queue` 的工作原理  
    multiprocessing.Queue 是一个进程安全的队列，它支持在多个进程间安全地传递数据。通过 put() 方法往队列中插入元素，通过
    get() 方法从队列中读取元素。队列是有状态的——它可能是空的，也可能是满的。  
    但是，调用 queue.empty() 和 queue.full() 检查队列的状态时，它们并不一定是瞬时精确的，特别是在多进程环境下。这是因为进程调度器的运行时行为可能影响队列状态与执行流的同步。
  * 多进程环境中的竞争条件  
    多进程中的竞争条件是指，多个进程对共享资源（如队列）进行读写操作时，可能发生时间竞争，导致状态检查与实际操作不同步。
    <br/>
    例如，在上述代码中，队列初始是空的，producer 函数在向队列中 put() 数据之前，先检查队列是否为空。如果队列为空，则输出 "
    队列为空"。
    <br/>
    然而，由于 queue.empty() 检查队列的时间与队列实际写入数据之间存在时间差（微秒级），这时可能发生以下情况：
    * 第一次检查：队列刚创建时确实是空的，因此在第一次 queue.empty() 调用时，返回 True，并输出 "队列为空！"；
    * 队列写入的时机：接下来，程序通过 queue.put(i) 将数据放入队列。然而，在向队列写入数据时，CPU
      会对两个操作进行调度（检查队列是否为空与向队列写入数据）。操作顺序未必是严格按代码执行顺序完成的——例如，即便
      queue.put() 已经被调用，数据可能还未完全写入队列，而是处于处理中或在系统缓冲区。这种情况是操作系统调度器或底层系统资源的延迟。
    * 第二次检查：在队列还未真正放入数据之前，再次调用 queue.empty()
      时，队列可能仍然报告为空。尤其是在没有任何同步机制（如锁或条件变量）来确保这些操作的顺序时，这种现象更容易发生；
  * `queue.empty()` 和 `queue.full()` 的局限性   
    根据 Python 文档，queue.empty() 和 queue.full()
    并不是绝对可靠的检查函数，尤其在多进程或多线程环境中。这是因为这些方法仅仅是对队列状态的一个瞬时检查，而队列状态在检查后的时刻可能已经被另一个进程修改了。
    两次“队列为空”的输出，很可能是因为以下两点：
    * 首次调用：在 put() 之前，队列确实为空；
    * 第二次调用：尽管在逻辑上应该向队列写入了数据，但由于多进程的调度机制或者 put() 操作的延迟，队列状态仍然被认为是空的（即使可能已经在写入数据的过程中）；
  * 操作系统调度与 CPU 资源竞争  
    在多进程环境中，操作系统调度器负责分配 CPU 时间给不同的进程。如果两个进程之间存在资源竞争（如共享队列），系统可能会在某个关键操作（如
    put() 或 empty() 检查）之前或之后切换到其他进程；  
    因此，程序的执行顺序可能不是你想象中的严格按照代码顺序执行的，而是受到系统调度器的影响；
  * 如何避免这种情况？
    要避免这种竞争条件，可以使用如下方式：
    * 锁：使用 multiprocessing.Lock 来保证在一个进程修改队列时，其他进程不能同时访问队列;
    * 条件变量：使用 multiprocessing.Condition 来实现更复杂的同步机制，确保队列的状态检查和更新保持一致;
    * 信号量：使用 multiprocessing.Semaphore 来控制对共享资源的访问，确保某一时刻只有固定数量的进程可以操作队列;
  * 调整后的代码示例  
    尝试添加锁来同步操作，确保队列的状态检查和写入操作不会出现竞争条件。

    ```python
    import multiprocessing
    import time
    
    
    def producer(queue, lock):
        for i in range(5):
            with lock:
                if queue.empty():
                    print("队列为空！")
                if queue.full():
                    print("队列已满！")
                queue.put(i)
                print(f"放入 {i} 到队列")
                time.sleep(0.1)  # 模拟一些延迟
  
  
    if __name__ == "__main__":
        lock = multiprocessing.Lock()
        q = multiprocessing.Queue(maxsize=3)
        p = multiprocessing.Process(target=producer, args=(q, lock))
        p.start()
        p.join()
    # 输出：
    # 队列为空！
    # 放入 0 到队列
    # 放入 1 到队列
    # 放入 2 到队列
    # 队列已满！
    ```
    在这个例子中，`with lock:`
    确保了每次对队列的检查和修改都是同步进行的，从而避免竞争条件。这种方式可以确保程序按照预期行为执行，不会输出两次“队列为空！”。  
    **小结：**
    * queue.empty() 和 queue.full() 的非可靠性：多进程环境中，队列状态检查不是瞬时精确的。
    * 多进程调度和资源竞争：多进程程序的执行顺序并不严格按照代码顺序执行，系统调度器和资源竞争可能导致队列状态不一致。
    * 避免竞争条件：可以通过使用锁或条件变量等同步机制，确保对队列的操作顺序正确。

* `qsize()`  
  `qsize()` 返回队列中当前未被获取(即 消费 `.get()`)的数据项的数量。这个方法在 Unix 系统中可以正常使用，但在 Windows
  上通常不可用（会抛出 NotImplementedError）。

  ```python
  import multiprocessing


  def producer(queue):
      for i in range(5):
          queue.put(i)
          print(f"Queue size: {queue.qsize()}")
  
  
  if __name__ == "__main__":
      q = multiprocessing.Queue(maxsize=3)
      p = multiprocessing.Process(target=producer, args=(q,))
      p.start()
      p.join()
  
  # 输出：
  # Queue size: 1
  # Queue size: 2
  # Queue size: 3
  ```

* `close()`
  用于关闭队列，不会影响已经在队列中的数据处理。close() 的作用是防止再向队列中添加新的数据，但队列中已经存在的数据依然可以被读取和处理。
  * 关闭队列的作用：当调用 Queue.close() 后，队列将不允许再向其中添加新的数据（即不能再调用 put()
    方法）。这个动作相当于通知队列 "不要再接受新任务"；
  * 对已提交数据的处理：close() 并不会清空或影响已经在队列中的数据，这些数据依然会按照正常流程被消费者进程读取（通过 get()
    方法）并处理。因此，队列关闭后，所有已提交的数据仍可以被继续处理直至队列为空；
  * 与 join() 配合：在关闭队列后，通常会使用 join() 方法来确保所有已提交的数据都被正确处理。join()
    会等待队列中的所有数据都被处理完，之后程序才会继续执行；

  **示例：**

  ```python
  import multiprocessing
  import time
  
  
  def worker(queue):
      while True:
          data = queue.get()
          if data is None:  # 检测到 None 作为结束信号，退出循环
              break
          print(f"Processed data: {data}")
          time.sleep(1)  # 模拟处理时间
  
  
  if __name__ == "__main__":
      queue = multiprocessing.Queue()
  
      # 启动一个消费者进程
      process = multiprocessing.Process(target=worker, args=(queue,))
      process.start()
  
      # 向队列中放入数据
      for i in range(5):
          queue.put(i)
  
      # 将所有数据（包括 None 结束信号）放入队列，然后再调用 queue.close()，确保不会因关闭队列而无法放入数据。
      queue.put(None)
  
      # 关闭队列底层的通信管道
      queue.close()
  
      # 该位置放入结束信号无效，队列不允许再向其中添加新的数据，导致 process 进程一直等待结束信号，主进程未继续往下执行打印：“All tasks processed.”
      # queue.put(None)
  
      # 等待消费者进程处理完所有数据
      process.join()
  
      print("All tasks processed.")
  
  # 输出：
  # Processed data: 0
  # Processed data: 1
  # Processed data: 2
  # Processed data: 3
  # Processed data: 4
  # All tasks processed.
  ```
  假设 `queue.put(None)` 代码下移至 `queue.close()` 之后，则抛出：ValueError: Queue is closed 异常，消费者进程收不到
  queue.put(None) 的结束信号，进程阻塞。
  * 为什么不能在 queue.close() 之后调用 queue.put()？
    * queue.put() 是用来将数据放入队列的操作；
    * queue.close() 是关闭队列底层通信管道的操作。一旦队列关闭，任何尝试往队列中放数据的操作都会抛出 ValueError: Queue is
      closed 异常；
  * 子进程未退出的原因：
    * 子进程在 worker 函数中一直等待从队列中获取数据。如果没有收到 None 作为结束信号，它会一直阻塞，等待新的数据；
    * 当主进程调用 queue.close() 后再试图放入 None 时，抛出 ValueError，并且 None 没有成功放入队列，导致子进程永远等不到结束信号，进而阻塞；
    * 如果需要当主进程被终止时，守护进程（子进程）也被终止，可以设置进程：multiprocessing.Process(target=worker, args=(
      queue,), daemon=True)，则主进程抛出 ValueError: Queue is closed 异常，相应的子进程自动被终止执行；

  ```python
  import multiprocessing
  import time
  
  def worker(queue):
      while True:
          data = queue.get()
          if data is None:  # 检测到 None 作为结束信号，退出循环
              break
          print(f"Processed data: {data}")
          time.sleep(1)  # 模拟处理时间
  
  if __name__ == "__main__":
      queue = multiprocessing.Queue()
  
      # 启动一个消费者进程
      process = multiprocessing.Process(target=worker, args=(queue,))
      process.start()
  
      # 向队列中放入数据
      for i in range(5):
          queue.put(i)
  
      queue.put(None)
  
      # 关闭队列底层的通信管道
      queue.close()
  
      # 等待消费者进程处理完所有数据
      process.join()
  
      print("All tasks processed.")
  
      queue.put("New")
      process2 = multiprocessing.Process(target=worker, args=(queue,))
      process2.start()
      queue.put(None)
      process2.join()
  
  # 输出：
  # Processed data: 0
  # Processed data: 1
  # Processed data: 2
  # Processed data: 3
  # Processed data: 4
  # Traceback (most recent call last):
  #   File "/home/bolean/workspace/examples/python-tricks/src/process_queue_demo20_04.py", line 321, in <module>
  #     queue.put("New")
  #   File "/usr/lib/python3.12/multiprocessing/queues.py", line 88, in put
  #     raise ValueError(f"Queue {self!r} is closed")
  # ValueError: Queue <multiprocessing.queues.Queue object at 0x7fef3ca1bb00> is closed
  # All tasks processed.
  ```
  注释 `queue.close()` 队列可接受新的数据，可由后面消费者进程处理。

  #### 【扩展知识点】
  * process.join()：这个方法会阻塞主进程，直到对应的子进程结束。当 join() 调用返回时，子进程已经终止;
  * 如果你在调用 process.join() 之后再试图调用 queue.put() 放入数据，子进程（消费者）已经不再运行，因此这些数据无法被处理或接收;

* `join_thread()` 和 `cancel_join_thread()`

  * 1). `join_thread()`
    * 功能：
      join_thread() 用于等待队列的后台线程结束。在队列的生命周期中，Python 维护着一个后台线程来管理进程之间的数据通信。如果你调用了
      queue.close()，你可以显式调用 join_thread() 来确保所有数据都已经通过队列的底层通信管道被发送出去，避免数据丢失。
    * 适用场景：
      * 当你需要确保进程在退出之前，所有排入队列的数据都已经被处理完（例如，传输至消费者进程）时，可以使用 join_thread();
      * 适合用在你不希望丢失数据，确保所有任务都已经被完全处理的场景;
    * 使用要求：
      * 必须在 queue.close() 之后才能调用 join_thread()，否则会抛出异常;
      * 这将阻塞调用它的进程，直到后台线程完成所有队列数据的处理并终止;
    * 代码示例：
      ```python
        import multiprocessing
        import time
        
        
        def worker(queue):
            while True:
                data = queue.get()
                if data is None:  # 检测到结束信号
                    break
                print(f"Processed: {data}")
                time.sleep(1)
        
        
        if __name__ == "__main__":
            queue = multiprocessing.Queue()
        
            process = multiprocessing.Process(target=worker, args=(queue,))
            process.start()
        
            # 放入数据
            for i in range(5):
                queue.put(i)
        
            # 放入 None 作为结束信号
            queue.put(None)
        
            # 关闭队列并等待后台线程结束
            queue.close()
            queue.join_thread()  # 等待队列处理完所有数据
        
            # 等待子进程退出
            process.join()
            print("All tasks processed.")
      ```
  * 2). `cancel_join_thread()`
    * 功能：
      cancel_join_thread() 用于取消 join_thread() 的阻塞行为，防止进程在退出时等待后台线程结束。这意味着当你调用
      cancel_join_thread() 后，主进程会立即退出，而不会等待队列中的数据被完全处理。
    * 适用场景：
      * 当你需要立即结束进程，而不关心队列中尚未处理的数据，或者不在乎数据丢失时，可以使用 cancel_join_thread()；
      * 通常用于极少数的紧急情况下，比如当你希望程序在某些条件下快速终止时；
    * 注意事项：
      * 使用 cancel_join_thread() 可能导致队列中尚未发送的数据丢失，因为它会跳过等待数据完全写入底层管道的过程；
      * 这个方法仅适用于你不关心数据丢失的情况，因此它在实际开发中不常见
    * 代码示例：为了展示 cancel_join_thread() 导致数据丢失的场景，主进程将不会等待队列中的数据被完全处理，会在放入一部分数据后立即退出，剩余数据会丢失。
    ```python
        import multiprocessing
        import time
        
        def worker(queue):
            while True:
                data = queue.get()
                if data is None:
                    print("Received termination signal. Worker exiting.")
                    break
                print(f"Worker processing: {data}")
                time.sleep(1)  # 模拟数据处理的时间
        
        if __name__ == "__main__":
            queue = multiprocessing.Queue()
        
            # 启动消费者进程
            process = multiprocessing.Process(target=worker, args=(queue,))
            process.start()
        
            # 向队列中放入数据
            for i in range(5):
                queue.put(i)
        
            # 放入结束信号 None
            queue.put(None)
        
            # 调用 cancel_join_thread() 取消等待后台线程处理队列数据
            queue.cancel_join_thread()
        
            # 关闭队列
            queue.close()
        
            # 模拟主进程提前退出
            time.sleep(1)  # 让部分数据来得及处理，主进程快速退出
        
            process.terminate()  # 强制终止子进程
        
            print("Main process exiting early.")
        
        # 输出：
        # Worker processing: 0
        # Main process exiting early.
    ````
    解释：
    * 在主进程中调用 cancel_join_thread() 后，主进程不再等待后台线程处理完所有数据，并在部分数据处理后强制终止子进程；
    * 由于主进程提前退出，消费者进程只处理了部分数据，并未收到结束信号 None；
    * 剩余数据未被处理，数据丢失；

    通过强制终止子进程 (process.terminate())，可以更明显地展示 cancel_join_thread() 可能导致的数据丢失场景。

  * 3). 两者的区别
    * join_thread():阻塞主进程，直到队列的后台线程处理完所有数据并退出。通常用于确保队列中的所有数据都被处理完毕，适合严谨的数据处理场景;
    * cancel_join_thread():允许主进程直接退出，不会等待后台线程处理队列中的数据。适用于无需等待数据处理完成或不担心数据丢失的场景;
  * 4). 重要性和使用建议
    * 何时使用 join_thread():  
      当你有多个进程使用 multiprocessing.Queue 进行通信时，通常需要确保在进程退出前所有数据都已被发送或接收，此时使用
      join_thread() 是一种可靠的方式来确保数据完整性。
    * 何时使用 cancel_join_thread():  
      如果你需要在不等待后台线程的情况下立即退出进程，并且不担心队列中尚未处理的数据丢失，那么可以使用
      cancel_join_thread()。但这种情况非常罕见，通常建议避免使用，除非你明确知道其影响。

#### 20.1.3.2 管道 (Pipe)

在 Python 中，`multiprocessing.Pipe` 是用于进程间简单高效的通信工具。与 Queue 不同，Pipe 提供了一个单一的双向通信通道（duplex
参数配置），由两个连接点（端点）组成。每个端点可以用来发送或接收数据，两个进程可以通过 Pipe 进行相互通信。

* 工作原理:
  * multiprocessing.Pipe() 返回一对连接对象 (conn1, conn2)，每个连接对象都有 send() 和 recv() 方法，可以分别用于发送和接收数据;
  * 通常，Pipe 的两个连接对象分别在不同的进程中使用，一个用于发送数据，另一个用于接收数据;

* 参数说明: `multiprocessing.Pipe(duplex=True/False)`  
  duplex（默认值为 True）：决定管道是否为双向通信。
  * True(默认)：允许双向通信，两端都可以发送和接收数据；
  * False：管道为单向通信，意味着一端只能发送数据，另一端只能接收数据；

* 行为：
  * send(obj)：将 obj 发送到连接的另一端；
    * obj: 需要发送的对象，这个对象必须是可序列化的（可以通过 pickle 模块进行序列化）；
  * recv()：接收通过 send() 发送的对象。如果没有数据，会阻塞直到接收到数据；
  * close()：关闭管道，禁止进一步的发送或接收操作。调用此方法后，尝试使用 send() 或 recv() 会引发异常；
  * poll([timeout])：检查是否有数据可供接收，如果有返回 True，否则返回 False;
    * timeout（可选）：设置超时时间，等待管道中是否有数据可接收。如果不传递该参数，poll() 将立即返回 True 或
      False。如果设置了超时时间（单位为秒），poll() 会阻塞指定的时间，直到有数据或超时；
* 优缺：
  * 优点：
    * Pipe 提供了简单且高效的双向通信机制，非常适合轻量级的通信需求；
    * 在性能上，Pipe 通常比 Queue 更快，因为 Queue 基于底层的锁机制，而 Pipe 则是基于文件描述符的轻量机制；
  * 缺点：
    * Pipe 只允许两个进程之间通信，不像 Queue 那样适合多进程通信。如果需要多个进程之间进行通信，可以使用 Queue；
* Pipe 与 Queue 的比较
  * Pipe 更适合双进程之间的快速通信，提供更轻量的通信机制；
  * Queue 适合在多个进程之间共享数据，但性能较低，因为它需要处理更多的并发控制和锁；
* 使用 Pipe 的场景
  * 双向通信：两个进程之间需要相互通信，例如客户端-服务器模式；
  * 单向通信：只需要一个进程发送数据，另一个进程接收并处理数据，使用 duplex=False 的 Pipe 能减少复杂性和不必要的操作；
* 示例：

**示例 1：单向通信（duplex=False）**

```python
import multiprocessing


def worker(conn):
    # 子进程接收来自主进程的消息
    while True:
        msg = conn.recv()
        if msg == "END":
            print("Worker received termination signal.")
            break
        print(f"Worker received: {msg}")


if __name__ == "__main__":
    # 创建单向管道（第二个连接用来发送，第一个用来接收）
    recv_conn, send_conn = multiprocessing.Pipe(duplex=False)

    # 启动子进程
    process = multiprocessing.Process(target=worker, args=(recv_conn,))
    process.start()

    # 主进程发送消息
    send_conn.send("Hello Child")

    # 发送结束信号
    send_conn.send("END")
    process.join()

# 输出：
# Worker received: Hello Child
# Worker received termination signal.
```

在 multiprocessing.Pipe(duplex=False) 中，返回两个连接对象 (conn1 和 conn2)，但在单向通信中，它们的作用是固定的：

* conn1（recv_conn）：用于接收数据;
* conn2（send_conn）：用于发送数据;

如果尝试用 send_conn.recv() 或 recv_conn.send()，将会导致错误，因为 duplex=False 限制了通信的方向。

**示例 2：双向通信（默认：duplex=True）**

```python
import multiprocessing


def worker(conn):
    # 子进程接收来自主进程的消息
    msg = conn.recv()
    print(f"Worker received: {msg}")
    # 子进程发送响应消息
    conn.send("Message received by worker")
    # 关闭连接
    conn.close()
    # OSError: handle is closed
    # conn.send("Test")


if __name__ == "__main__":
    # 创建一个双向管道：默认：duplex=True
    conn1, conn2 = multiprocessing.Pipe(duplex=True)
    # 启动子进程
    process = multiprocessing.Process(target=worker, args=(conn2,))
    process.start()
    # 主进程发送消息
    conn1.send("Hello,World!")
    # 接受来自子进程的消息
    msg = conn1.recv()
    print(f"Parent received: {msg}")
    process.join()

# 输出：
# Worker received: Hello,World!
# Parent received: Message received by worker
```

双向管道允许主进程和子进程相互发送和接收消息，conn1 和 conn2 都可以用 send() 和 recv()。

**示例 3：综合示例**

```python
import multiprocessing
import time


def worker(conn):
    # 子进程通过 conn 发送数据
    for i in range(5):
        conn.send(i)
        print(f"Worker sent: {i}")
        time.sleep(1)
    # 关闭连接
    conn.send(None)  # 发送结束信号
    conn.close()


if __name__ == "__main__":
    # 创建管道
    parent_conn, child_conn = multiprocessing.Pipe()

    # 启动子进程
    process = multiprocessing.Process(target=worker, args=(child_conn,))
    process.start()

    # 主进程从管道中接收数据
    while True:
        # 检查管道中是否有数据
        if parent_conn.poll(2):  # 最多等待2秒
            data = parent_conn.recv()
            if data is None:  # 如果收到结束信号，则退出循环
                print("Received termination signal. Exiting.")
                break
            print(f"Parent received: {data}")
        else:
            print("No data received within the timeout.")

    # 等待子进程结束
    process.join()

    print("All tasks processed.")
```

#### 20.1.3.3 管理器（Manager）

管理器（Manager）提供了一种创建共享数据的方法，从而可以在不同进程中共享，甚至可以通过网络跨机器共享数据。管理器维护一个用于管理共享对象的服务。其他进程可以通过代理访问这些共享对象。

#### 20.1.3.3.1 multiprocessing.Manager()

multiprocessing.Manager() 会返回一个已经启动的 SyncManager 对象，该对象提供各种方法来创建共享数据对象，如共享的列表、字典、队列等。Manager
使用进程间的代理模式：每个进程都能通过代理对象访问这些共享数据，但实际的数据保存在由 Manager 启动的独立进程中。  
在 Manager 被垃圾回收或父进程退出时，管理器的进程也会立即终止。为了启动 Manager，需要调用 start() 方法。  
multiprocessing.Manager 提供的共享数据类型（如 list、dict、Namespace、Lock、Queue
等）是进程安全的。它们通过管理器进程进行通信和同步，以确保多个进程对共享数据的操作不会发生竞争条件。所有由 Manager
创建的共享对象都是通过代理访问的，代理对象会自动管理并发访问，提供必要的锁定和同步机制。

Manager 提供了一些常用的共享数据结构，可以通过管理器实例的方法创建：

* manager.list(): 共享的列表对象，允许多个进程同时访问和修改。操作时自动进行锁定，确保进程间的同步；
* manager.dict(): 共享的字典对象，进程间可安全地读取和修改键值对；
* manager.Queue(): 共享的队列，用于在进程之间传递消息或数据。底层实现了同步机制，可以在多个进程中安全地使用；
* manager.Value() 和 manager.Array(): 共享的基本数据类型和数组，允许多个进程安全地读写数值和数组；
* manager.Namespace(): 一个可以存储任意属性的对象，属性可以在进程间安全地共享和修改；
* manager.Lock() 和 manager.RLock(): 提供进程间同步机制，确保某些资源的独占访问权；
* manager.Event(): 用于跨进程的事件通知；
  所有这些数据类型都是通过一个后台的 Manager 服务进程管理的，每个共享对象在不同进程中的操作都通过代理进行，因此可以确保进程安全。

**为什么 multiprocessing.Manager 是进程安全的？**  
multiprocessing.Manager 提供的共享数据类型（如 list, dict, Namespace 等）通过代理对象的方式与底层的管理进程通信，因此，它们的操作（如
append() 或 update()）会被同步到管理进程上。这种设计确保了多个进程并发访问这些共享对象时，不会发生低层次的竞争条件。

**可能的竞争问题**  
虽然 Manager 提供了基础操作的同步，但复杂操作（多个操作组成的逻辑）依然会引发竞争关系。例如，如果两个进程同时尝试对同一个共享列表进行读取、修改、写入的组合操作（如检查值是否存在后再插入新值），仍然可能发生不一致的情况，因为
Manager 只保护单个操作是进程安全的，不能保证多个操作的原子性。

**示例代码**  
以下代码展示了 multiprocessing.Manager 共享列表的基本使用，并且由于操作是单步的，因此不会出现竞争关系：

```python
import multiprocessing


def worker(shared_list):
    # 每个进程向共享列表中添加数据
    for i in range(5):
        shared_list.append(i)


if __name__ == "__main__":
    # 创建 Manager 对象
    manager = multiprocessing.Manager()

    # 创建共享列表
    shared_list = manager.list()

    # 启动多个进程
    processes = []
    for _ in range(3):
        p = multiprocessing.Process(target=worker, args=(shared_list,))
        processes.append(p)
        p.start()

    # 等待所有进程结束
    for p in processes:
        p.join()

    # 打印共享列表的最终结果
    print(f"Shared list: {list(shared_list)}")
```

**竞争关系的例子**  
假设有以下场景：先检查某个值是否在列表中，然后再执行一些操作。如果这两步操作没有作为一个整体进行同步，那么就会出现逻辑竞争：

```python
import multiprocessing


def worker(shared_list, value):
    # 检查是否已经存在 value，若不存在则添加
    if value not in shared_list:
        shared_list.append(value)


if __name__ == "__main__":
    manager = multiprocessing.Manager()
    shared_list = manager.list()

    processes = []
    for i in range(3):
        p = multiprocessing.Process(target=worker, args=(shared_list, 10))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    print(f"Shared list: {list(shared_list)}")

# 输出：Shared list: [10, 10]
```

在上述例子中，多个进程可能会同时检查 shared_list 中是否有 10，如果都发现没有，则它们都可能向列表中插入 10，导致逻辑竞争。

**如何避免复杂逻辑中的竞争关系？**

```python
import multiprocessing


def worker(shared_list, value, lock):
    with lock:  # 使用锁保护复杂操作
        if value not in shared_list:
            shared_list.append(value)


if __name__ == "__main__":
    manager = multiprocessing.Manager()
    shared_list = manager.list()
    lock = multiprocessing.Lock()  # 创建锁

    processes = []
    for i in range(5):
        p = multiprocessing.Process(target=worker, args=(shared_list, 10, lock))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    print(f"Shared list: {list(shared_list)}")

# 输出:Shared list: [10]
```

通过使用锁，可以确保每个进程在执行检查和插入操作时不会被其他进程打断，从而避免竞争关系。

* 基础操作是安全的：multiprocessing.Manager 确保对共享对象（如列表、字典等）的基础操作（如 append() 或 setitem()）是进程安全的。
* 复杂操作需要锁：如果涉及多个步骤的复杂操作（如检查和修改），需要使用 Lock 等同步机制来避免逻辑上的竞争条件。
* 合理使用锁：尽量缩小加锁的范围，以避免性能上的损失。

**性能注意事项**

虽然 multiprocessing.Manager 提供了方便的进程间安全机制，但由于其底层通过 IPC（进程间通信）机制进行同步，性能可能会比直接使用共享内存结构（如
multiprocessing.Array、multiprocessing.Value）稍慢。如果在高并发场景中性能是关键因素，可能需要使用更高效的原生数据共享类型。

以下是关于 multiprocessing.Manager 的使用示例:

* **示例 1：共享字典和列表**  
  在这个例子中，多个进程可以同时修改共享的字典和列表，并且这些修改对其他进程可见。

  ```python
  import multiprocessing
  
  
  def worker(shared_dict, shared_list, value):
      shared_dict[value] = value * 2
      shared_list.append(value)
  
  
  if __name__ == "__main__":
      # 创建一个 Manager 对象
      manager = multiprocessing.Manager()
  
      # 创建共享的字典和列表
      shared_dict = manager.dict()
      shared_list = manager.list()
  
      # 启动多个子进程，每个进程向共享对象中添加不同的数据
      processes = []
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(shared_dict, shared_list, i))
          processes.append(p)
          p.start()
  
      # 等待所有进程结束
      for p in processes:
          p.join()
  
      # 打印最终的共享对象
      print(f"Shared Dictionary: {shared_dict}")
      print(f"Shared List: {shared_list}")
  
  # 输出：
  # Shared Dictionary: {1: 2, 2: 4, 3: 6, 0: 0, 4: 8}
  # Shared List: [1, 2, 3, 0, 4]
  ```
  * 每个进程都修改了共享的字典 shared_dict 和共享的列表 shared_list;
  * Manager 确保了这些对象可以在进程间安全地共享;   
    <br/>
* **示例 2：使用 Namespace 在进程之间共享简单的变量**  
  `Namespace` 允许多个进程共享简单的命名空间变量，适合共享单一值而不是复杂的数据结构。
  ```python
  import multiprocessing
  
  def worker(namespace, value):
      namespace.result += value  # 修改共享的命名空间变量
  
  if __name__ == "__main__":
      # 创建 Manager 对象
      manager = multiprocessing.Manager()
  
      # 创建一个共享的命名空间对象
      namespace = manager.Namespace()
      namespace.result = 0  # 初始化共享变量
  
      # 启动多个进程，每个进程修改命名空间中的共享变量
      processes = []
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(namespace, i))
          processes.append(p)
          p.start()
  
      # 等待所有进程结束
      for p in processes:
          p.join()
  
      # 打印命名空间中的最终结果
      print(f"Namespace result: {namespace.result}")
      
  # 输出：
  # Namespace result: 6
  ```
  上述用例，可以看到 `result: 6` 并不是准确的累加值，同时输出结果也不固定一致。

  原因:    
  `+=` 是非原子操作，导致竞争问题，最终结果可能不正确。 `+=` 实际上涉及多个步骤：
  * 读取当前值；
  * 进行加法操作；
  * 将新值写回变量；  
    在多进程环境中，如果两个进程同时执行 +=
    操作，它们可能都读取相同的旧值，在各自的进程中计算出新值，并将其写回，从而导致最终的结果没有累加所有进程的加法操作。  
    **举个例子：**    
    多个进程可能会按照以下步骤同时操作同一个变量 namespace.result：
  * 进程 A 读取 namespace.result（值为 0）；
  * 进程 B 读取 namespace.result（值为 0）；
  * 进程 A 将 namespace.result 修改为 0 + 1 = 1；
  * 进程 B 将 namespace.result 修改为 0 + 2 = 2；  
    最终结果是 namespace.result == 2，而实际上我们希望它等于 3（即所有操作都能累加上去）。这种情况就是竞争条件，并非
    Namespace 自身的问题，而是并发环境中多个进程同时操作共享资源所导致的问题。

  **锁：**  
  锁 (Lock)
  是用来防止这种竞争条件的。锁能确保只有一个进程能够访问共享变量并执行操作，其他进程必须等待这个进程释放锁后才能访问共享变量。这可以避免多个进程同时读取和修改共享变量的情况，从而保证操作的原子性。  
  修改后代码：
  ```python
  import multiprocessing
  
  
  def worker(namespace, value, lock):
      with lock:  # 使用锁确保对命名空间变量的修改是原子操作
          namespace.result += value  # 修改共享的命名空间变量
  
  
  if __name__ == "__main__":
      # 创建 Manager 对象
      manager = multiprocessing.Manager()
  
      # 创建一个共享的命名空间对象
      namespace = manager.Namespace()
      namespace.result = 0  # 初始化共享变量
  
      # 创建一个进程锁
      lock = multiprocessing.Lock()
  
      # 启动多个进程，每个进程修改命名空间中的共享变量
      processes = []
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(namespace, i, lock))
          processes.append(p)
          p.start()
  
      # 等待所有进程结束
      for p in processes:
          p.join()
  
      # 打印命名空间中的最终结果
      print(f"Namespace result: {namespace.result}")
  
  # 输出：
  # Namespace result: 10
  ```
  * multiprocessing.Manager() 确保多个进程可以安全地共享对象，但并不保证多个进程同时修改对象时的操作原子性。
  * 锁 (Lock) 确保每次对共享对象的修改是原子的，即一个进程的修改在另一个进程开始修改之前完成。这样可以防止数据被并发修改时产生的不一致问题。

  **【扩展知识点】**  
  竞争条件（Race
  Condition）是指在并发或多线程、多进程程序中，多个线程或进程对共享资源的访问或修改顺序不确定，导致程序的结果不一致或不可预期的情况。  
  当多个进程或线程并发地操作同一个共享资源（如变量、文件或内存区域）时，如果它们的执行顺序或操作顺序没有得到适当的同步控制，结果可能会因为不同的操作顺序而产生不同的结果。竞争条件是并发编程中常见的问题，特别是在没有使用适当的锁或同步机制的情况下。

  **竞争条件的发生条件：**
  * 多个进程或线程：至少两个以上的线程或进程同时运行；
  * 共享资源：这些线程或进程必须访问或修改同一个共享的资源（如变量、数据结构、文件等）；
  * 无同步机制：没有适当的锁或同步机制来控制对共享资源的访问顺序；

  **举个例子：**  
  假设我们有两个线程 T1 和 T2，它们都想修改一个共享的变量 x，初始值为 0：
    ```
    # 初始值
    x = 0
  
    def increment():
        global x
        x += 1
  
    ```
  现在，线程 T1 和 T2 同时执行 increment()，理想情况下，x 应该最终等于 2，因为两个线程各自将 x 增加了 1。然而，由于竞争条件，可能发生如下情况：
  * 线程 T1 读取 x 的值（此时 x = 0）；
  * 线程 T2 也读取 x 的值（x 仍然是 0）；
  * 线程 T1 将 x 修改为 1；
  * 线程 T2 将 x 也修改为 1（覆盖了线程 T1 的修改）；

  最终，x 的值是 1，而不是预期的 2。这就是典型的竞争条件：多个线程同时访问共享资源，且它们的操作顺序没有得到适当的控制。

  **如何避免竞争条件**  
  为了避免竞争条件，通常需要使用一些同步机制来控制多个进程或线程对共享资源的访问。例如：
  * 锁（Lock）：确保每次只有一个线程或进程能够访问共享资源，其他线程或进程必须等待锁被释放；
  * 信号量（Semaphore）：控制多个线程对资源的访问，可以允许多个线程同时访问某个资源，但限制同时访问的数量；
  * 条件变量（Condition Variable）：允许线程在等待某个条件时释放锁，避免资源死锁；

  **竞争条件的危害**
  * 数据不一致：竞争条件会导致数据的不可预测性，最终的数据状态可能与预期不符，造成错误的计算结果；
  * 难以调试：由于竞争条件往往是依赖于执行顺序的随机性，这使得调试和发现问题变得困难，问题可能只在特定的运行条件下才会显现；

  竞争条件是在并发环境下，由于多个进程或线程争用共享资源并且没有适当的同步机制，导致程序结果不确定或不一致的现象。为了避免竞争条件，需要使用适当的同步机制来确保共享资源的访问顺序是可控的。


* **示例 3：使用 Queue 在进程之间通信**

  multiprocessing.Manager.Queue() 是 multiprocessing.Queue() 的一种变体，它通过 Manager
  机制创建一个共享队列，可以在不同的进程甚至网络中的机器间共享。它与标准的 multiprocessing.Queue()
  一样是进程间通信的工具，允许将数据传递到不同的进程中。然而，Manager.Queue() 是通过 SyncManager
  代理来管理的，这使得它不仅可以用于本地进程间通信，还可以通过网络实现远程进程间通信。

  **特点：**
  * 进程安全：Manager.Queue() 是进程安全的，类似于 multiprocessing.Queue()，但它通过管理器代理实现，所以更加通用，可以跨机器使用；
  * 网络共享：Manager.Queue() 可以在不同机器间通过网络共享数据，这点区别于标准的 multiprocessing.Queue()；
  * 竞争关系：Manager.Queue() 通过代理机制来管理数据操作，类似于 list.append()、dict.update()
    之类的操作，管理器确保操作的进程安全性，因此能够避免竞争条件；

  **方法与属性：**
  * `put(item)`：将 item 放入队列中；
  * `get()`：从队列中取出一个项（如果队列为空，会阻塞直到有数据为止）；
  * `qsize()`：返回队列中的项目数（有时无法准确保证，尤其在跨机器使用时）；
  * `empty()`：检查队列是否为空；
  * `full()`：检查队列是否已满；
  * `put_nowait(item)`：非阻塞地将 item 放入队列中，如果队列已满，则抛出 queue.Full 异常；
  * `get_nowait()`：非阻塞地从队列中取出一项，如果队列为空，则抛出 queue.Empty 异常；
  * `close()`：关闭队列，防止进一步的 put 操作；

  ```python
  import multiprocessing
  import time
  
  
  def producer(queue, items):
      for item in items:
          print(f"Producer adding: {item}")
          queue.put(item)
          time.sleep(1)
  
  
  def consumer(queue):
      while True:
          item = queue.get()
          if item is None:  # 检测到 None 作为结束信号
              break
          print(f"Consumer processing: {item}")
  
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
  
      # 创建共享的队列
      queue = manager.Queue()
  
      # 启动生产者进程
      items_to_produce = [1, 2, 3, 4, 5]
      producer_process = multiprocessing.Process(target=producer, args=(queue, items_to_produce))
  
      # 启动消费者进程
      consumer_process = multiprocessing.Process(target=consumer, args=(queue,))
  
      # 启动生产者和消费者
      producer_process.start()
      consumer_process.start()
  
      # 等待生产者完成
      producer_process.join()
  
      # 发送结束信号给消费者
      queue.put(None)
  
      # 等待消费者完成
      consumer_process.join()
  
      print("All tasks processed.")
  
  # 输出：
  # Producer adding: 1
  # Consumer processing: 1
  # Producer adding: 2
  # Consumer processing: 2
  # Producer adding: 3
  # Consumer processing: 3
  # Producer adding: 4
  # Consumer processing: 4
  # Producer adding: 5
  # Consumer processing: 5
  # All tasks processed.
  ```

  * 生产者进程 不断向队列中放入数据；
  * 消费者进程 从队列中读取数据并处理，直到检测到 None 作为结束信号；
  * Manager.Queue() 保证了进程间队列通信的安全性；

  **竞争关系的说明：**  
  由于 Manager.Queue() 通过代理对象来管理队列，它的 put() 和 get() 操作是原子的，因此在多个进程同时执行时不会出现竞争条件。这一点与
  Namespace 中的非原子操作（如 +=）不同。  
  举个例子，如果两个进程同时调用 queue.put()，代理对象会确保每次操作完整进行，因此数据不会丢失或混乱。类似地，queue.get()
  也是进程安全的，如果两个进程同时从队列中取数据，代理对象会保证每个进程获取的数据都是唯一的。

  **进程安全锁的使用示例：**  
  尽管 Manager.Queue() 本身是进程安全的，但有时候我们可能需要对队列外的操作进行锁定，避免多个进程同时修改非队列的共享数据。

  ```python
  import multiprocessing
  
  
  def worker(queue, value, lock):
      with lock:  # 使用锁保护
          queue.put(value)  # 向队列中添加数据
          print(f"Worker {value} put data in queue.")
  
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      queue = manager.Queue()  # 创建一个共享队列
      lock = multiprocessing.Lock()  # 创建锁
  
      processes = []
  
      # 启动多个进程，向队列中放入数据
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(queue, i, lock))
          processes.append(p)
          p.start()
  
      # 等待所有进程结束
      for p in processes:
          p.join()
  
      # 从队列中取出所有数据
      while not queue.empty():
          data = queue.get()
          print(f"Main process got data: {data}")
  
  # 输出：
  # Worker 0 put data in queue.
  # Worker 1 put data in queue.
  # Worker 2 put data in queue.
  # Worker 3 put data in queue.
  # Worker 4 put data in queue.
  # Main process got data: 0
  # Main process got data: 1
  # Main process got data: 2
  # Main process got data: 3
  # Main process got data: 4
  ```

* **示例 4：使用 Lock & RLock 进程间同步和控制访问**  
  通过 manager.Lock() 和 manager.RLock() 来实现进程间的同步和控制访问。
  * `manager.Lock()`  
    `manager.Lock()` 是用于进程间同步的标准互斥锁，它确保只有一个进程在同一时刻可以访问共享资源。其他进程在尝试获取该锁时，如果锁已被持有，它们将被阻塞，直到锁被释放。

    **常见方法：**
    * `acquire(block=True, timeout=None)`：获取锁，如果 block 为 True（默认），则阻塞直到锁被释放；否则立即返回。如果设置了
      timeout，则最多等待 timeout 秒；
    * `release()`：释放锁，使其他等待的进程可以获取该锁；

    **Lock 示例：**

    ```python
    import multiprocessing
    
    def worker(lock, shared_list, value):
        with lock:  # 使用锁来同步对共享资源的访问
            shared_list.append(value)
            print(f"Process {value} added to list.")
    
    if __name__ == "__main__":
        manager = multiprocessing.Manager()
        lock = manager.Lock()  # 创建一个 Manager 提供的锁
        shared_list = manager.list()  # 创建一个共享列表
    
        processes = []
        for i in range(5):
            p = multiprocessing.Process(target=worker, args=(lock, shared_list, i))
            processes.append(p)
            p.start()
    
        for p in processes:
            p.join()
    
        print("Final list:", list(shared_list))
    
    # 输出：
    # Process 0 added to list.
    # Process 1 added to list.
    # Process 2 added to list.
    # Process 3 added to list.
    # Process 4 added to list.
    # Final list: [0, 1, 2, 3, 4]
    ``` 
    在这个例子中，每个进程在访问共享列表之前都要先获取锁，从而避免多个进程同时访问列表并导致竞态条件。只有一个进程在某个时刻可以修改共享列表。

  * `manager.RLock()`  
    `manager.RLock()` 是可重入锁，它允许同一个进程多次获取锁而不会发生死锁。与 Lock() 不同，RLock()
    允许持有锁的线程或进程再次获取锁，而不会导致自己阻塞，直到调用相同次数的 release() 以释放锁。

    **常见方法：**
    * `acquire(block=True, timeout=None)`：与 Lock 类似，但允许同一进程多次获取锁;
    * `release()`：与 Lock 类似，需要与获取锁的次数一致，调用多少次 acquire() 就需要相同次数的 release() 才能完全释放锁;

    **Lock 示例：**
    ```python
        import multiprocessing
    
    
    def worker(rlock, shared_dict, key, value):
        with rlock:  # 获取可重入锁
            if key not in shared_dict:
                print(f"Adding {key} to shared dict.")
                shared_dict[key] = value
            with rlock:  # 再次获取锁，测试重入
                print(f"Updating {key} to {value * 2}.")
                shared_dict[key] = value * 2
    
    
    if __name__ == "__main__":
        manager = multiprocessing.Manager()
        rlock = manager.RLock()  # 创建一个 Manager 提供的可重入锁
        shared_dict = manager.dict()  # 创建一个共享字典
    
        processes = []
        for i in range(5):
            p = multiprocessing.Process(target=worker, args=(rlock, shared_dict, i, i * 10))
            processes.append(p)
            p.start()
    
        for p in processes:
            p.join()
    
        print("Final dict:", dict(shared_dict))
    
    
    # 输出：
    # Adding 1 to shared dict.
    # Updating 1 to 20.
    # Adding 0 to shared dict.
    # Updating 0 to 0.
    # Adding 2 to shared dict.
    # Updating 2 to 40.
    # Adding 3 to shared dict.
    # Updating 3 to 60.
    # Adding 4 to shared dict.
    # Updating 4 to 80.
    # Final dict: {1: 20, 0: 0, 2: 40, 3: 60, 4: 80}
    ```
    在这个例子中，每个进程可以多次获取 RLock() 锁而不会发生死锁。在第一次获取锁后，进程会尝试再次获取锁，确保同一个进程可以重入。

  * Lock 和 RLock 的区别：
    * Lock 是普通的锁，只有一个进程或线程可以持有，其他试图获取锁的进程会阻塞。任何进程只能调用一次 acquire()，然后必须调用一次
      release() 才能释放锁；
    * RLock 是可重入锁，同一进程或线程可以多次获取该锁，但需要调用相同次数的 release() 才能完全释放锁。这对于递归函数或需要多次调用共享资源的情况下非常有用；

  * 竞争关系与进程同步：  
    在使用 Lock 或 RLock 时，锁的目的是防止多个进程同时访问和修改共享资源，从而避免出现竞争条件（例如多个进程同时修改一个变量的值，导致结果不正确）。
    * manager.Lock() 确保在某一时刻只有一个进程能够修改共享对象。这对于简单的修改操作非常有用，比如向共享列表中添加元素；
    * manager.RLock() 更加灵活，适用于需要递归调用或同一进程多次锁定资源的情况；

  * 进程同步的注意事项：  
    在进程间同步时，锁的使用能够保证数据的一致性，但是也要注意锁的使用可能会影响性能，尤其是在大量进程同时等待锁时。因此，应尽量将锁的作用范围控制在最小的代码区域中，减少锁的持有时间。

  * 【扩展知识点】  
    在 Python 中，Lock 和 RLock 都可以使用 with 语句来管理锁的获取和释放。这是通过上下文管理器（Context Manager）来实现的。with
    语句在处理锁时，自动处理了锁的 acquire() 和 release()，使得代码更加简洁和安全。具体来说：
    * with lock: 是上下文管理器的一部分，进入 with 语句块时，自动调用 lock.acquire()；
    * 当退出 with 语句块时，无论是正常退出还是发生异常，都会自动调用 lock.release()；

    **手动调用 acquire() 和 release():**

    通常，使用锁的标准做法是手动调用 acquire() 来获取锁，并在完成操作后手动调用 release() 来释放锁：

       ```
       lock.acquire()
       try:
       # 对共享资源进行操作
       finally:
           lock.release()  # 确保锁在任何情况下都能释放
       ``` 
    手动调用的风险在于，如果程序出现异常而没有执行 release()，锁将一直被持有，导致其他进程无法获取该锁，从而可能导致死锁或程序无法继续。

    **使用 with 管理锁**  
    with 语句相当于为 acquire() 和 release() 创建了一种自动化的机制，使得即使在出现异常时，锁也能够被正确释放。它让代码更加简洁和易于维护。等价的代码如下：
       ```
       with lock:
       # 对共享资源进行操作
       ```
    当进入 with 代码块时，lock.acquire() 被调用，执行代码块中的操作。当离开 with 代码块时，无论是正常离开还是异常离开，都会调用
    lock.release()，确保锁被释放。
    * 简洁性：减少代码量，不需要手动调用 acquire() 和 release()，且代码更加直观。
    * 安全性：避免忘记释放锁的风险，即使出现异常也能确保锁被正确释放，防止死锁。
    * 自动化管理：with 语句自动管理资源的获取和释放，遵循上下文管理器的机制，使代码更加 Pythonic。    
      <br/>
* **示例 5：Manager 中的 Value 和 Array**  
  multiprocessing.Manager().Value 和 multiprocessing.Manager().Array 是 multiprocessing
  模块中用于进程间共享数据的高级工具。它们通过管理器对象提供了进程间安全数据共享方式。与 multiprocessing.Value 和
  multiprocessing.Array 的共享内存实现不同，Manager().Value 和 Manager().Array
  通过代理模式使用，这使得它们不仅可以在同一台机器上共享，还可以通过网络在不同机器之间共享数据。
  * `Manager().Value(typecode, value)`：创建一个用于共享的值对象。
    * `typecode`：表示值的类型，例如 'i' 表示整数，'d' 表示双精度浮点数；
    * `value`：初始化时的值；
  * `Manager().Array(typecode, sequence)`：创建一个用于共享的数组对象。
    * `typecode`：表示数组元素的类型，例如 'i' 表示整数，'d' 表示双精度浮点数；
    * `sequence`：用于初始化数组的序列；

  示例：Manager().Value 的用法
  ```python
    import multiprocessing
    
    def worker(shared_value, lock):
        with lock:
            shared_value.value += 1  # 在共享变量上执行加法操作
    
    if __name__ == "__main__":
        manager = multiprocessing.Manager()
        shared_value = manager.Value('i', 0)  # 共享整数变量，初始值为0
        lock = manager.Lock()  # 用于同步的锁
    
        processes = []
        for _ in range(5):
            p = multiprocessing.Process(target=worker, args=(shared_value, lock))
            processes.append(p)
            p.start()
    
        for p in processes:
            p.join()
    
        print(f"Final Value: {shared_value.value}")  # 输出应该为 5
  ```

  * manager.Value('i', 0) 创建了一个初始值为 0 的共享整数;
  * 使用 lock 来确保对共享变量的操作是原子的（防止竞争条件）;
  * 每个进程都会对共享变量执行 +1 操作，因此最终值为 5;

  示例：Manager().Array 的用法
  ```python
  import multiprocessing
  
  def worker(shared_array, lock):
      with lock:
          for i in range(len(shared_array)):
              shared_array[i] += 1  # 增加每个数组元素的值
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      shared_array = manager.Array('i', [1, 2, 3])  # 创建一个共享数组
      lock = manager.Lock()
  
      processes = []
      for _ in range(3):
          p = multiprocessing.Process(target=worker, args=(shared_array, lock))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
  
      print(f"Final Array: {shared_array[:]}")  # 输出应该为 [4, 5, 6]  
  ```
  * manager.Array('i', [1, 2, 3]) 创建了一个初始为 [1, 2, 3] 的共享数组；
  * 每个进程都会将数组中的每个元素加 1；
  * 最终数组的结果是 [4, 5, 6]；

  **Manager().Value 和 Manager().Array 与 Namespace 的区别：**
  * 共享机制：
    * Manager().Value 和 Manager().Array 是通过代理模式进行通信的，支持跨进程和跨机器的数据共享；
    * Namespace 提供一个简单的共享空间，但它是一个更为通用的数据容器，并且需要手动使用锁来避免竞争条件；
  * 线程/进程安全：
    * Manager().Value 和 Manager().Array 默认在一定程度上提供了进程间安全的共享机制，但对于复杂的并发访问场景，仍建议使用显式锁来避免数据竞争。
    * Namespace 并没有内置锁机制，必须手动加锁来保证数据操作的安全性。  
      <br/>
* **示例 6：Manager().Barrier**  
  Barrier 用于同步多个进程，确保所有进程在某个点上等待彼此。
  ```python
  import multiprocessing
  import time
  
  def worker(barrier, id):
      print(f"Process {id} is doing some work...")
      time.sleep(2)  # 模拟工作
      print(f"Process {id} is ready at the barrier.")
      barrier.wait()  # 等待其他进程到达此点
      print(f"Process {id} has crossed the barrier.")
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      barrier = manager.Barrier(3)  # 设置需要等待的进程数量
  
      processes = []
      for i in range(3):
          p = multiprocessing.Process(target=worker, args=(barrier, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()  # 等待所有进程结束
          
  # 输出：
  # Process 1 is doing some work...
  # Process 0 is doing some work...
  # Process 2 is doing some work...
  # Process 1 is ready at the barrier.
  # Process 0 is ready at the barrier.
  # Process 2 is ready at the barrier.
  # Process 2 has crossed the barrier.
  # Process 0 has crossed the barrier.
  # Process 1 has crossed the barrier.
  ```

  * 每个进程在完成工作后会调用 barrier.wait()，这会使其阻塞，直到所有进程都到达这个点;
  * 当所有进程都到达后，它们将一起继续执行;  
    <br/>
* **示例 7：Manager().BoundedSemaphore**    
  BoundedSemaphore 用于限制同时访问的进程数。
  ```python
  import multiprocessing
  import time
  
  def worker(sem, id):
      with sem:  # 获取信号量
          print(f"Process {id} is accessing the resource.")
          time.sleep(2)  # 模拟访问资源
          print(f"Process {id} has released the resource.")
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      semaphore = manager.BoundedSemaphore(2)  # 允许同时两个进程访问
  
      processes = []
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(semaphore, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()  # 等待所有进程结束
  
  # 输出：
  # Process 1 is accessing the resource.
  # Process 3 is accessing the resource.
  # Process 1 has released the resource.
  # Process 0 is accessing the resource.
  # Process 3 has released the resource.
  # Process 2 is accessing the resource.
  # Process 0 has released the resource.
  # Process 4 is accessing the resource.
  # Process 2 has released the resource.
  # Process 4 has released the resource.
  ```

  * BoundedSemaphore 限制最多只有两个进程可以同时访问资源，其他进程需要等待；
  * 使用 with sem: 自动管理信号量的获取和释放；  
    <br/>
* **示例 8：Manager().Condition**    
  Condition 允许进程在某个条件下等待，并可以被其他进程通知。
  ```python
  import multiprocessing
  import time
  
  def worker(condition):
      with condition:
          print("Worker is waiting for the condition.")
          condition.wait()  # 等待条件被通知
          print("Worker has been notified and is continuing work.")
  
  def notifier(condition):
      time.sleep(2)  # 模拟一些工作
      with condition:
          print("Notifier is notifying the worker.")
          condition.notify()  # 通知等待的进程
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      condition = manager.Condition()
  
      worker_process = multiprocessing.Process(target=worker, args=(condition,))
      notifier_process = multiprocessing.Process(target=notifier, args=(condition,))
  
      worker_process.start()
      notifier_process.start()
  
      worker_process.join()
      notifier_process.join()
  
  # 输出：
  # Worker is waiting for the condition.
  # Notifier is notifying the worker.
  # Worker has been notified and is continuing work.
  ```

  * worker 进程在等待条件时会阻塞，直到被 notifier 进程通知；
  * condition.notify() 通知所有等待的进程继续执行；  
    <br/>
* **示例 9：Manager().Event**    
  Event 允许一个或多个进程等待某个事件的发生。
  ```python
  import multiprocessing
  import time
  
  def worker(event):
      print("Worker is waiting for the event to be set.")
      event.wait()  # 等待事件被设置
      time.sleep(2)
      print("Worker has received the event.")
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      event = manager.Event()
  
      worker_process = multiprocessing.Process(target=worker, args=(event,))
      worker_process.start()
  
      time.sleep(2)  # 主进程做一些工作
      print("Main process is setting the event.")
      event.set()  # 设置事件，通知工作进程，主进程继续执行
      print("Main process continue.")
      worker_process.join()  # 等待工作进程结束
  
  # 输出：
  # Worker is waiting for the event to be set.
  # Main process is setting the event.
  # Main process continue.
  # Worker has received the event.
  ```
  * worker 进程会阻塞在 event.wait()，直到主进程调用 event.set()；
  * 一旦事件被设置，工作进程将继续执行；  
    <br/>
* **示例 10：Manager().Semaphore**    
  Semaphore 控制对特定资源的访问，允许一定数量的进程同时访问。
  ```python
  import multiprocessing
  import time
  
  def worker(sem, id):
      with sem:  # 获取信号量
          print(f"Process {id} is accessing the resource.")
          time.sleep(2)  # 模拟访问资源
          print(f"Process {id} has released the resource.")
  
  if __name__ == "__main__":
      manager = multiprocessing.Manager()
      semaphore = manager.Semaphore(2)  # 允许同时两个进程访问
  
      processes = []
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(semaphore, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()  # 等待所有进程结束
  ```
  * Semaphore 允许最多两个进程同时访问资源，其他进程需等待;
  * 使用 with sem: 语句简化了信号量的管理;  
    <br/>
* **示例 11：Manager().Semaphore() 和 Manager().BoundedSemaphore() 区别**
  * Manager().Semaphore()
    * 功能：允许你创建一个信号量，可以控制同时访问某个资源的进程数；
    * 特点：没有上限，只要信号量的计数大于 0，进程就可以获取信号量。即使当前进程已经释放信号量，计数可以超过初始值；
  * Manager().BoundedSemaphore()
    * 功能：也是用于控制并发访问的信号量，但它具有一个上限；
    * 特点：确保信号量的计数不会超过初始值。若超过该值，调用 release() 将引发 ValueError；

  示例对比：
  ```python
    import multiprocessing
    
    
    # 使用 Semaphore
    def semaphore_worker(sem, id):
        sem.acquire()
        print(f"Semaphore Worker {id} acquired semaphore.")
        sem.release()
        print(f"Semaphore Worker {id} released semaphore.")
    
    
    # 使用 BoundedSemaphore
    def bounded_semaphore_worker(bsem, id):
        bsem.acquire()
        print(f"BoundedSemaphore Worker {id} acquired bounded semaphore.")
        bsem.release()
        print(f"BoundedSemaphore Worker {id} released bounded semaphore.")
    
    
    if __name__ == "__main__":
        manager = multiprocessing.Manager()
    
        # Semaphore 示例
        sem = manager.Semaphore(2)
        semaphore_processes = []
        for i in range(3):
            p = multiprocessing.Process(target=semaphore_worker, args=(sem, i))
            semaphore_processes.append(p)
            p.start()
    
        # 等待 Semaphore 进程结束
        for p in semaphore_processes:
            p.join()
    
        # BoundedSemaphore 示例
        bsem = manager.BoundedSemaphore(2)
        bounded_semaphore_processes = []
        for i in range(3):
            p = multiprocessing.Process(target=bounded_semaphore_worker, args=(bsem, i))
            bounded_semaphore_processes.append(p)
            p.start()
    
        # 等待 BoundedSemaphore 进程结束
        for p in bounded_semaphore_processes:
            p.join()
  ```
  * Semaphore: 你创建的 Semaphore 允许最多 2 个进程同时访问。如果同时有超过 2 个进程试图调用 acquire()
    ，后续的进程将被阻塞，直到信号量被释放；
  * BoundedSemaphore: 也有相同的行为，限制最多 2 个进程同时访问。与普通信号量不同的是，如果你尝试释放一个信号量而计数已经达到初始值，会抛出
    ValueError；
    如果你需要限制并发访问且想要防止超过最大限制，使用 BoundedSemaphore；如果不需要这种限制，可以使用普通的 Semaphore。

#### 20.1.3.3.2 自定义管理器

要创建一个自定义的管理器，需要新建一个 multiprocessing.managers.BaseManager 的子类，然后使用这个管理器类上的 register()
类方法将新类型或者可调用方法注册上去。例如:

```python
from multiprocessing.managers import BaseManager


class MathsClass:
    # MathsClass 提供了两个方法：add 用于加法，mul 用于乘法
    def add(self, x, y):
        return x + y

    def mul(self, x, y):
        return x * y


# 定义自定义管理器
class MyManager(BaseManager):
    # MyManager 继承自 BaseManager，并可用于注册共享对象
    pass


# 使用 register 方法将 MathsClass 注册为共享对象，命名为 'Maths'
MyManager.register('Maths', MathsClass)

if __name__ == '__main__':
    # 启动管理器，这样可以自动处理资源的释放
    with MyManager() as manager:
        #  创建 MathsClass 的共享实例
        maths = manager.Maths()
        # 通过调用 maths.add(4, 3) 和 maths.mul(7, 8) 分别计算并输出结果
        print(maths.add(4, 3))  # prints 7
        print(maths.mul(7, 8))  # prints 56
```

#### 20.1.3.3.3 使用远程管理器

可以将管理器服务运行在一台机器上，然后使用客户端从其他机器上访问。(假设它们的防火墙允许)

服务器端代码: 运行下面的代码可以启动一个服务，包含了一个共享队列，允许远程客户端访问
  
```
>>> from multiprocessing.managers import BaseManager
>>> from queue import Queue
# 创建一个共享队列
>>> queue = Queue()
# 定义一个管理器类
>>> class QueueManager(BaseManager): pass
# 注册共享队列，使得可以通过名称获取它
>>> QueueManager.register('get_queue', callable=lambda:queue)
# 启动管理器，绑定到特定地址和端口
>>> m = QueueManager(address=('', 50000), authkey=b'abracadabra')
# 创建并启动服务器
>>> s = m.get_server()
>>> s.serve_forever()
```

  * 导入模块：导入 BaseManager 和 Queue；
  * 创建队列：实例化一个 Queue 对象，用于在进程之间共享数据；
  * 定义管理器：创建一个自定义的 QueueManager 继承自 BaseManager；
  * 注册队列：通过 register 方法将共享队列注册到管理器，允许远程客户端访问；
  * 启动管理器：设置管理器的地址（在本地运行，监听所有接口）和端口（50000），以及授权密钥；
  * 启动服务器：调用 get_server() 方法并使服务器进入无限循环，等待客户端连接；

客户端代码（PUT 数据）：远程客户端可以通过下面的方式访问服务

```
>>> from multiprocessing.managers import BaseManager
# 定义管理器类
>>> class QueueManager(BaseManager): pass
# 注册共享队列
>>> QueueManager.register('get_queue')
# 连接到远程管理器
>>> m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
>>> m.connect()
# 获取共享队列
>>> queue = m.get_queue()
# 向队列中放入数据
>>> queue.put('hello')
```

  * 导入模块：同样导入 BaseManager；
  * 定义管理器：重新定义 QueueManager 类；
  * 注册队列：注册共享队列，使客户端能够访问；
  * 连接管理器：通过提供远程地址和端口连接到服务器；
  * 获取队列：获取远程共享队列的代理；
  * 放入数据：使用 put 方法向队列中添加数据；

客户端代码（GET 数据）：也可以通过下面的方式

```
>>> from multiprocessing.managers import BaseManager
# 定义管理器类
>>> class QueueManager(BaseManager): pass
# 注册共享队列
>>> QueueManager.register('get_queue')
# 连接到远程管理器
>>> m = QueueManager(address=('foo.bar.org', 50000), authkey=b'abracadabra')
>>> m.connect()
# 获取共享队列
>>> queue = m.get_queue()
# 从队列中获取数据
>>> queue.get()
```

  * 重复定义和注册：与前面的客户端代码相似，定义管理器和注册共享队列；
  * 连接和获取：连接到远程管理器并获取队列；
  * 获取数据：使用 get 方法从队列中读取数据；

本地进程也可以访问这个队列，利用上面的客户端代码通过远程方式访问:

```
>>> from multiprocessing import Process, Queue
>>> from multiprocessing.managers import BaseManager
# 定义工作进程
>>> class Worker(Process):
...     def __init__(self, q):
...         self.q = q
...         super().__init__()
...     def run(self):
...         # 向队列放入数据
...         self.q.put('local hello')
...
# 创建本地队列
>>> queue = Queue()
# 启动工作进程
>>> w = Worker(queue)
>>> w.start()
# 启动管理器
>>> class QueueManager(BaseManager): pass
...
>>> QueueManager.register('get_queue', callable=lambda: queue)
>>> m = QueueManager(address=('', 50000), authkey=b'abracadabra')
>>> s = m.get_server()
>>> s.serve_forever()
```

  * 工作进程类：定义一个 Worker 类，继承自 Process，用于将数据放入共享队列；
  * 创建本地队列：实例化一个本地 Queue 对象；
  * 启动工作进程：创建并启动工作进程；
  * 启动管理器：重新定义并注册共享队列；
  * 运行服务器：启动服务器并等待连接，允许本地进程和远程客户端访问共享队列；

#### 20.1.3.4 共享 ctypes 对象

ctypes 是 Python 的外部函数库，它提供了与 C 兼容的数据类型，并允许调用 DLL 或共享库中的函数。结合 multiprocessing 模块，我们可以利用 ctypes 来创建共享的 C 类型对象，这些对象可以在多个进程之间共享内存。这种共享机制避免了在不同进程之间复制数据，能够提升性能。

**常见共享的 ctypes 对象**

* multiprocessing.Value：用于创建一个共享的单个值，类似于 C 的标量类型（如 int，float）；
* multiprocessing.Array：用于创建共享数组，类似于 C 的数组类型；

这些对象是托管在共享内存中的，多个进程可以同时访问它们，且需要通过同步原语（如锁）来避免数据竞争。

**示例代码**  

* 示例 1：使用 `multiprocessing.Value`
  ```python
  import multiprocessing
  import time
  # 或者
  # import ctypes
  
  
  def worker(shared_value, lock):
      for _ in range(5):
          time.sleep(0.1)
          with lock:  # 获取锁
              shared_value.value += 1
  
  
  if __name__ == "__main__":
      # 创建一个共享内存的 Value
      # 这里使用的是字符串 'i'，这是 multiprocessing 提供的一种简化表示方式，表示一个 有符号整数。这种符号化表示来自 Python 的 struct 模块的格式化字符。'i' 相当于 ctypes.c_int，但写法上更简洁。multiprocessing 支持多种类似的符号化字符来表示不同的数据类型。
      shared_value = multiprocessing.Value('i', 0)  # 'i' 表示整型
      # 或者: 这里显式地使用 ctypes.c_int 来指定 Value 对象的数据类型，即创建一个存储在共享内存中的 ctypes 整数类型 (c_int)，使用 ctypes 提供的类型时，可以更明确地表示你想要使用的具体 C 数据类型
      # shared_value = multiprocessing.Value(ctypes.c_int, 0)
      lock = multiprocessing.Lock()  # 创建锁
  
      # 创建多个进程
      processes = []
      for _ in range(3):  # 启动3个进程
          process = multiprocessing.Process(target=worker, args=(shared_value, lock))
          processes.append(process)
          process.start()
  
      # 等待所有进程完成
      for process in processes:
          process.join()
  
      print(f'Final value: {shared_value.value}')
  ```
  * 创建锁：使用 multiprocessing.Lock() 创建一个锁对象；
  * 获取和释放锁：在访问共享内存时，使用 with lock: 语句来自动获取锁和释放锁。这可以确保在锁被持有时，其他进程无法访问该资源；
  * 多个进程：在这个示例中，我们启动了多个进程同时对共享变量进行增值操作，通过加锁确保数据的一致性；

* 示例 2：共享数组 `multiprocessing.Array`

  ```python
  import multiprocessing
  import time
  # 或者
  # import ctypes
  
  
  def worker(shared_array, lock):
      for i in range(len(shared_array)):
          time.sleep(0.1)
          with lock:  # 获取锁
              shared_array[i] += 1
  
  
  if __name__ == "__main__":
      # 创建一个共享内存的 Array
      shared_array = multiprocessing.Array('i', [0, 1, 2, 3, 4])  # 'i' 表示整型数组
      # 或者
      # shared_array = multiprocessing.Array(ctypes.c_int, [0，1, 2, 3, 4])
      lock = multiprocessing.Lock()  # 创建锁
  
      # 创建多个进程
      processes = []
      for _ in range(3):  # 启动3个进程
          process = multiprocessing.Process(target=worker, args=(shared_array, lock))
          processes.append(process)
          process.start()
  
      # 等待所有进程完成
      for process in processes:
          process.join()
  
      print(f'Final array: {[shared_array[i] for i in range(len(shared_array))]}')
  
  # 输出：
  # Final array: [3, 4, 5, 6, 7]
  ```

* `multiprocessing.Value` 和 `multiprocessing.Array` 会将数据放在共享内存中，因此多个进程可以读写这些数据；
* 如果多个进程需要并发读写共享数据，建议使用同步原语（如 multiprocessing.Lock）来避免数据竞争；
* ctypes 提供了对多种 C 类型的支持，如 c_int、c_float、c_double 等；
* 【说明】
   * 'i'：有符号整数（相当于 ctypes.c_int）；
   * 'I'：无符号整数（相当于 ctypes.c_uint）；
   * 'f'：浮点数（相当于 ctypes.c_float）；
   * 'd'：双精度浮点数（相当于 ctypes.c_double）；
   * 'b'：有符号字节（相当于 ctypes.c_byte）；
   * 'B'：无符号字节（相当于 ctypes.c_ubyte）；

#### 20.1.3.5 同步原语

在 Python 的多进程编程中，进程同步是指协调多个进程之间的执行顺序，以避免竞争条件和数据不一致的问题。Python 的 multiprocessing 模块提供了多种同步原语：Lock、RLock、Semaphore、Event、Condition、Barrier。通常来说同步原语在多进程环境中并不像它们在多线程环境中那么必要。

* `Lock`（锁）  
  锁用于确保同一时刻只有一个进程可以访问共享资源。可以使用 `with` 语句自动管理锁的获取和释放。  
  * `acquire(block=True, timeout=None)`：用于请求锁的获取。
    * `block（默认值为 True）`：如果设置为 True，则当锁被其他进程持有时，当前进程会阻塞，直到锁可用。如果设置为 False，则当前进程不会阻塞，如果锁不可用，则立即返回 False；
    * `timeout（默认值为 None）`：指定阻塞等待锁的最长时间（以秒为单位）。如果在超时时间内未获取到锁，方法将返回 False。如果不设置或设置为 None，则会无限期等待；
  * `release()`：用于释放锁，使其他进程可以获取该锁。调用 release 必须在锁被当前进程持有的情况下，否则会引发 RuntimeError。

  示例：    
  ```python
  import multiprocessing
  import time
  
  
  def worker(lock, num):
      with lock:  # 获取锁
          print(f'Process {num} is running.')
          time.sleep(1)  # 模拟处理时间
      print(f'Process {num} is success.')
  
  
  if __name__ == "__main__":
      lock = multiprocessing.Lock()
      processes = []
  
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(lock, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
  ```

* `RLock`（递归锁）  
  递归锁必须由持有进程亲自释放，如果某个进程拿到了递归锁，这个进程可以再次拿到这个锁而不需要等待，但是这个进程拿锁操作和释放锁操作的次数必须相同。
  
  ```python
  import multiprocessing
  import time
  
  
  def worker(rlock, num):
      with rlock:  # 获取锁
          print(f'Process {num} is running.')
          time.sleep(1)
          with rlock:  # 重新获取锁
              print(f'Process {num} is still running.')
  
  
  if __name__ == "__main__":
      rlock = multiprocessing.RLock()
      processes = []
  
      for i in range(3):
          p = multiprocessing.Process(target=worker, args=(rlock, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
          
  # 输出：
  # Process 0 is running.
  # Process 0 is still running.
  # Process 1 is running.
  # Process 1 is still running.
  # Process 2 is running.
  # Process 2 is still running.
  ```

* `Semaphore`（信号量）  
  一种信号量对象: 允许一定数量的进程同时访问某个资源。通过初始化信号量的计数，可以控制访问的数量。类似于 `threading.Semaphore` 不同在于，它的 `acquire` 方法的第一个参数名是和 `Lock.acquire()` 一样的 `block` 。  
  **【备注】** 
    * 在 macOS 上，不支持 sem_timedwait ，所以，调用 acquire() 时如果使用 timeout 参数，会通过循环 sleep 来模拟这个函数的行为；  
    * 这个包的某些功能依赖于宿主机系统的共享信号量的实现，如果系统没有这个特性， multiprocessing.synchronize 会被禁用，尝试导入这个模块会引发 ImportError 异常；    
  
  ```python
  import multiprocessing
  import time
  
  
  def worker(sem, num):
      with sem:  # 使用 with 语句自动获取和释放信号量
          print(f'Process {num} is running.')
          time.sleep(2)  # 模拟处理时间
      print(f'Process {num} is end.')
  
  
  # def worker(sem, num):
  #     sem.acquire()  # 获取信号量
  #     print(f'Process {num} is running.')
  #     time.sleep(2)  # 模拟处理时间
  #     sem.release()  # 释放信号量
  
  if __name__ == "__main__":
      sem = multiprocessing.Semaphore(2)  # 允许最多2个进程同时访问
      processes = []
  
      for i in range(3):
          p = multiprocessing.Process(target=worker, args=(sem, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
  
  # 输出：
  # Process 0 is running.
  # Process 1 is running.
  # Process 0 is end.
  # Process 2 is running.
  # Process 1 is end.
  # Process 2 is end.
  ```

  * with sem: 会在进入块时调用 sem.acquire()，在退出块时调用 sem.release()，即使在块中发生异常，也会确保信号量被释放；
  * 这样使用可以减少显式调用 acquire() 和 release() 的错误风险，并使代码更易于维护；

* `BoundedSemaphore`（信号量）  
  `BoundedSemaphore` 类似于 `Semaphore`，但它限制了信号量的最大计数，防止过度释放信号量。

  ```python
  import multiprocessing
  import time
  
  
  def worker(sem, num):
      with sem:
          print(f'Worker {num} is running.')
          time.sleep(1)  # 模拟工作
  
  
  if __name__ == "__main__":
      sem = multiprocessing.BoundedSemaphore(2)  # 最大计数为2
      processes = []
  
      for i in range(5):
          p = multiprocessing.Process(target=worker, args=(sem, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
  
  # 输出：
  # Worker 0 is running.
  # Worker 1 is running.
  # Worker 4 is running.
  # Worker 2 is running.
  # Worker 3 is running.
  ```
  
* `Condition`（条件）  
  允许一个或多个进程等待某个条件的发生，通常用于复杂的同步场景，比如生产者-消费者模型。

  * acquire(): 获取条件的锁；
  * release(): 释放条件的锁；
  * wait(): 释放锁并等待其他进程发出通知；
  * wait_for(): 重复地调用 wait() 直到满足判断式或者发生超时；
  * notify(n=1): 唤醒一个等待该条件的进程；
  * notify_all(): 唤醒所有等待该条件的进程；
  [参考]：(https://docs.python.org/zh-cn/3/library/threading.html#threading.Condition.acquire)
  
  **示例:**  
  ```python
  import multiprocessing
  import time
  
  
  def producer(cond):
      # 在获取锁后模拟生产，完成后调用 notify() 通知消费者
      with cond:  # 获取条件的锁
          print('Producing...')
          time.sleep(2)  # 模拟生产时间
          cond.notify()  # 通知消费者
          print('Producer has notified the consumer.')
  
  
  def consumer(cond):
      # 先打印等待消息，然后在获取锁后调用 wait() 等待通知
      print('Waiting for producer...')
      with cond:  # 获取条件的锁
          cond.wait()  # 等待生产者的通知
          print('Consumer has been notified.')
  
  
  if __name__ == "__main__":
      condition = multiprocessing.Condition()
  
      # 创建并启动消费者进程
      consumer_process = multiprocessing.Process(target=consumer, args=(condition,))
      consumer_process.start()
  
      # 确保消费者先运行
      time.sleep(1)
  
      # 创建并启动生产者进程
      producer_process = multiprocessing.Process(target=producer, args=(condition,))
      producer_process.start()
  
      # 等待两个进程完成
      consumer_process.join()
      producer_process.join()
  
  # 输出：
  # Waiting for producer...
  # Producing...
  # Producer has notified the consumer.
  # Consumer has been notified.  
  ```

  主进程先启动消费者，确保它在生产者之前等待。可以使用 time.sleep(1) 暂停一下，确保消费者先运行。

* `Event`（事件）  
  Event 用于在进程之间进行简单的信号传递，一个进程可以设置事件，其他进程可以等待该事件的发生。  
  ```python
  import multiprocessing
  import time
  
  
  def worker(event):
      print('Worker is waiting for event.')
      event.wait()  # 等待事件
      print('Worker is proceeding.')
  
  
  if __name__ == "__main__":
      event = multiprocessing.Event()
      p = multiprocessing.Process(target=worker, args=(event,))
      p.start()
  
      time.sleep(2)  # 模拟处理时间
      event.set()  # 触发事件
      print('Event has been set.')
  
      p.join()
  ```
  
* `Barrier`    
  Barrier 用于协调多个进程的执行，确保所有进程在到达某个点后一起继续执行。  
  ```python
  import multiprocessing
  import time
  
  
  def worker(barrier, id):
      print(f'Process {id} is doing some work...')
      time.sleep(2)  # 模拟工作
      print(f'Process {id} has reached the barrier.')
  
      # 等待其他进程到达
      barrier.wait()
  
      print(f'Process {id} is continuing after the barrier.')
  
  
  if __name__ == "__main__":
      num_processes = 3
      barrier = multiprocessing.Barrier(num_processes)
  
      processes = []
      for i in range(num_processes):
          p = multiprocessing.Process(target=worker, args=(barrier, i))
          processes.append(p)
          p.start()
  
      for p in processes:
          p.join()
  ```

  * 创建 Barrier：使用 multiprocessing.Barrier(num_processes) 创建一个 Barrier 对象，指定需要同步的进程数量；
  * 工作函数： 每个进程执行 worker 函数，模拟一些工作，然后调用 barrier.wait()，在这里进程将会被阻塞，直到指定进程数都到达此点；
  * 进程启动： 主进程创建并启动多个子进程，最后通过 join() 等待它们完成；

#### 20.1.3.6 跨进程直接访问内存共享

`multiprocessing.shared_memory` 模块是在 Python 3.8 中引入的，用于在多个进程之间直接共享内存，避免了进程间数据的复制，这对于需要高效共享大块数据的应用非常有用。

**关键类与方法**  

* `multiprocessing.shared_memory.SharedMemory`: 创建一个 SharedMemory 类的实例用来新建一个共享内存块或关联到一个已存在的共享内存块；

  ```python  
  class multiprocessing.shared_memory.SharedMemory(name=None, create=False, size=0, *, track=True)
  ``` 
  
  参数：
    * name (str | None)：指定请求共享内存名称，以字符串形式指定。“None”（默认值），则随机生成一个新名称；
    * create (bool)：指定新建共享内存块 (True) 还是关联到已有的共享内存块 (False)；
    * size (int)：新建共享内存块所请求的字节数。由于某些平台会选择根据平台的内存页大小来分配内存块，因此共享内存块的实际大小可能会大于等于所请求的大小。 当关联到已有的共享内存块时，size 形参将被忽略；
    * track (bool)：当为 True 时，将在 OS 不会自动为共享内存块注册资源跟踪器进程的平台上执行注册操作；
    
  方法：
  * SharedMemory.create(size): 创建指定大小的共享内存块；
  * SharedMemory.attach(name): 附加到一个已有的共享内存块；
  * SharedMemory.close(): 关闭共享内存，但不删除它；
  * SharedMemory.unlink(): 删除共享内存，使其不可用；

* ShareableList: 这是一个类似 Python 列表的对象，可以跨进程共享；


* 示例：跨进程共享 NumPy 数组

  ```python
  import multiprocessing
  import numpy as np
  from multiprocessing import shared_memory
  
  # worker 函数: 这个函数在子进程中运行，用来访问和修改共享内存中的数据。
  def worker(shared_name, shape, dtype):
      # shared_name: 共享内存的名称。
      # shape 和 dtype: 用于恢复共享的 NumPy 数组的形状和数据类型。
      # 附加到共享内存块，附加到由主进程创建的共享内存块，通过 name 参数，子进程可以识别和连接到相同的共享内存块。
      shm = shared_memory.SharedMemory(name=shared_name)
      
      # 恢复 NumPy 数组，使用共享内存缓冲区 shm.buf 恢复 NumPy 数组。shape 和 dtype 用来正确地重建数组。
      array = np.ndarray(shape, dtype=dtype, buffer=shm.buf)
      
      # 修改数组的内容，修改共享内存中的数组，具体来说，将数组的第一个元素设置为 999。
      array[0] = 999
      # 关闭共享内存对象，但不删除它，因为其他进程可能仍在使用它。
      shm.close()
  
  if __name__ == "__main__":
      # 创建 NumPy 数组：创建了一个 NumPy 数组 array，它包含 5 个元素，数据类型为 64 位整数（np.int64）。
      array = np.array([1, 2, 3, 4, 5], dtype=np.int64)
      
      # 分配共享内存：create=True: 表示正在创建一个新的共享内存块。
      # 为了确保共享内存足够大，我们根据数组的字节大小来分配共享内存（array.nbytes 是数组的字节大小）。
      shm = shared_memory.SharedMemory(create=True, size=array.nbytes)
      
      # 将 NumPy 数组的内容复制到共享内存
      # 使用共享内存的缓冲区 shm.buf，创建与原始数组相同形状和数据类型的 NumPy 数组 shared_array。
      shared_array = np.ndarray(array.shape, dtype=array.dtype, buffer=shm.buf)
  
      # 将原始数组 array 的所有内容复制到共享内存中的 shared_array。
      shared_array[:] = array[:]
      
      # 启动进程，修改共享内存中的数据
      # 创建一个新进程，目标函数是 worker。通过 args 参数传递共享内存的名称 shm.name、数组的形状 array.shape 以及数据类型 array.dtype。
      process = multiprocessing.Process(target=worker, args=(shm.name, array.shape, array.dtype))
      process.start()
      process.join()
      
      # 检查共享内存中的数据
      print("Main process array:", shared_array)
      
      # 清理共享内存
      # 关闭主进程中的共享内存对象。
      shm.close()
      # 删除共享内存块，释放它的资源。unlink 是必须的，因为共享内存不会自动删除。
      shm.unlink()
  
  # 输出：
  # Main process array: [999   2   3   4   5]
  ```
     
* 示例：使用 ShareableList

  除了使用共享内存直接处理 NumPy 数组，还可以通过 ShareableList 共享简单的 Python 列表。

  ```python
  import multiprocessing
  from multiprocessing import shared_memory
  
  
  def worker(shared_list):
      shared_list[0] = 42  # 修改共享列表中的值
  
  
  if __name__ == "__main__":
      # 创建共享列表
      shared_list = shared_memory.ShareableList([1, 2, 3, 4, 5])
  
      # 启动进程，修改共享列表中的数据
      process = multiprocessing.Process(target=worker, args=(shared_list,))
      process.start()
      process.join()
  
      # 查看修改后的共享列表
      print("Main process list:", shared_list)
  
      # 清理共享内存
      shared_list.shm.close()
      shared_list.shm.unlink()
  
  # 输出：
  # Main process list: ShareableList([42, 2, 3, 4, 5], name='psm_4ef9f08d')
  ```

参考官网：https://docs.python.org/zh-cn/3/library/multiprocessing.shared_memory.html

### 20.2 线程 (Thread)

线程是 CPU 调度的基本单位，一个进程可以包含多个线程，线程之间共享进程的内存空间。

参考：https://bbs.huaweicloud.com/blogs/289314

**特点：**

* 轻量级：相比于进程，线程的创建和销毁成本较低；
* 共享内存：线程之间可以共享内存空间，这使得线程间通信更加便捷；

**缺点：**

* GIL 限制：在 CPython 解释器中，线程受到全局解释器锁（GIL）的限制，导致 Python 的多线程在 CPU 密集型任务中无法利用多核并行执行；
* 安全问题：由于线程共享内存，多个线程同时修改共享数据时，可能会发生竞争条件，需要使用锁 (Lock) 等同步机制来防止数据竞争；

**使用场景：**

* I/O 密集型任务，如文件读写、网络请求等；

**代码示例：**

```python
import threading
import time


def worker(num):
    time.sleep(1)
    print(f'Worker: {num}')


threads = []
for i in range(5):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

#### 20.2.1 threading.Thread

`threading.Thread` 是 Python 中的线程类，用于创建和管理线程。通过指定参数，可以自定义线程的行为。

```
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
```

**参数介绍**  
* `group`: 目前默认始终为 `None`，保留参数，保留给将来实现 ThreadGroup 类的扩展使用；
* `target`: 线程执行目标函数（用于 run() 方法调用的可调用对象），默认是 None，表示不需要调用任何方法；
* `name`: 线程名称，默认情况下，以 "Thread-N" 形式构造唯一名称，其中 N 为一个较小的十进制数值，或是 "Thread-N (target)" 的形式，其中 "target" 为 `target.__name__`（前提指定了 target 参数）；
* `args`: 调用目标函数的参数列表或元组，默认为 `()`；
* `kwargs`: 调用目标函数的关键字参数字典，默认是 `{}`；
* `daemon`: 是否将线程设置为守护线程：守护线程在主线程结束时会被强制终止；非守护线程会让主线程等待它执行完。 默认值（None），线程将继承当前线程的守护模式属性（主线程默认是非守护线程），如果不是 None，daemon 参数将显式地设置该线程是否为守护模式。

**常用方法**   

* `start()`: 启动线程，调用 run() 方法（同一线程里最多只能被调用一次，否则抛出 RuntimeError），线程将在后台开始执行；
* `run()`: 线程启动时调用的方法，可以在子类里重载该方法来定制线程执行的逻辑。如果指定了 target 参数，则默认调用 target(*args, **kwargs)；
  ```
  >>> from threading import Thread
  >>> t = Thread(target=print, args=[1])
  >>> t.run()
  1
  >>> t = Thread(target=print, args=(1,))
  >>> t.run()
  1
  ```
* `join(timeout=None)`: 阻塞当前线程，等待该线程终止，timeout 是可选的超时时间，指定秒数后超时返回；
* `name`: 只用于识别的字符串，它没有语义，多个线程可以赋予相同的名称，初始名称由构造函数设置；  
  `getName()`、`setName()` 3.10 版本弃用，改为直接以特征属性（name）方式使用它；
  ```
  >>> t = Thread(target=print,args=[1])
  >>> t.name
  'Thread-2 (print)'
  >>> t.name='Thread-2 (print)-xxx'
  >>> t.name
  'Thread-2 (print)-xxx'
  ```
* `ident`: 线程的 '线程标识符'，如果线程尚未开始则为 None。它是一个非零的整数，它的值没有直接含义，主要是用作 magic cookie，比如作为含有线程相关数据的字典的索引。线程标识符可能会在线程退出，新线程创建时被复用；
  ```
  >>> t.ident
  >>> t.start()
  1
  >>> t.ident
  140461852206848
  ```
* `native_id`: 此线程的线程 ID (TID)，由 OS (内核) 分配。 这是一个非负整数，或者如果线程还未启动则为 None。它的值可被用来在整个系统中唯一地标识这个特定线程（直到线程终结，在那之后该值可能会被 OS 回收再利用）；
  ```
  >>> t.native_id
  51645
  ```
* `is_alive()`: 返回线程是否存活。当 run() 方法刚开始直到 run() 方法刚结束，这个方法返回 True 。`threading.enumerate()` 返回当前所有存活的 Thread 对象的列表；
* `daemon`: 布尔值，表示这个线程是否是一个守护线程（True）或不是（False）。 这个值必须在调用 start() 之前设置，否则会引发 RuntimeError 。它的初始值继承自创建线程，主线程不是一个守护线程，因此所有在主线程中创建的线程默认为 daemon = False （当没有存活的非守护线程时，整个Python程序才会退出）；
  ```
  >>> t.daemon
  False
  ```
  `isDaemon()`、`setDaemon()`，自 3.10 版本弃用的 daemon 的取值/设值 API，改为直接以特征属性（daemon）方式使用它。

**Thread 类示例**    

* 示例 1: 使用 target、args 和 kwargs 参数  
  创建两个线程，分别传递不同的参数给目标函数：  

  ```
  import threading
  import time

  def worker(message: str, delay: int):
      """
      线程执行目标函数
      """
      for _ in range(3):
          # 返回当前对应调用者控制线程的 Thread 对象
          print(f"{threading.current_thread().name}: {message}")
          time.sleep(delay)
   
  if __name__ == '__main__':
      # 创建 2 个线程
      # 通过位置参数为 worker 函数传递 "Hello from Thread 1" 和延迟 1 秒
      thread1 = threading.Thread(target=worker, args=("Hello from Thread 1", 1), name="worker-1")
      # 通过关键字参数传递不同的参数
      thread2 = threading.Thread(target=worker, kwargs={"message": "Hello from Thread 2", "delay": 2}, name="worker-2")
  
      # 启动线程
      thread1.start()
      thread2.start()
  
      # 等待线程结束
      thread1.join()
      thread2.join()
  
      print("Main thread finished.")
  
  # 输出：
  # worker-1: Hello from Thread 1
  # worker-2: Hello from Thread 2
  # worker-1: Hello from Thread 1
  # worker-2: Hello from Thread 2
  # worker-1: Hello from Thread 1
  # worker-2: Hello from Thread 2
  # Main thread finished.  
  ```

* 示例 2: 自定义线程类   
  继承 threading.Thread 类创建自定义线程类，重写 run() 方法： 
  ```
  import threading
  import time
  
  
  class CustomThread(threading.Thread):
  
      def __init__(self, name, delay):
          super().__init__()
          self.name = name
          self.delay = delay
  
      def run(self):
          """重载 run 方法，定义线程的执行逻辑"""
          for i in range(3):
              print(f"{self.name} is running. Count: {i}")
              time.sleep(self.delay)
  
  
  if __name__ == "__main__":
      # 创建 2 个自定义线程
      thread1 = CustomThread(name="Custom-Thread-1", delay=1)
      thread2 = CustomThread(name="Custom-Thread-2", delay=2)
  
      # 启动线程
      thread1.start()
      thread2.start()
  
      # 等待线程结束
      thread1.join()
      thread2.join()
  
      print("Main thread finished.")
  
  # 输出：
  # Custom-Thread-1 is running. Count: 0
  # Custom-Thread-2 is running. Count: 0
  # Custom-Thread-1 is running. Count: 1
  # Custom-Thread-1 is running. Count: 2
  # Custom-Thread-2 is running. Count: 1
  # Custom-Thread-2 is running. Count: 2
  # Main thread finished.  
  ```
  
* 示例 3: 守护线程  
  守护线程在主线程结束时会自动停止，不管它是否完成了它的任务：  
  ```python
  import threading
  import time
  
  
  def worker():
      # 线程任务，死循环模拟守护线程
      while True:
          print(f"{threading.current_thread().name} running in the background")
          time.sleep(1)
  
  
  if __name__ == "__main__":
      # 创建守护线程：daemon=True，将线程设置为守护线程
      daemon_thread = threading.Thread(target=worker, name="", daemon=True)
  
      # 启动守护线程
      daemon_thread.start()
  
      # 主线程等待 3 秒：守护线程会在主线程结束时自动停止，因此在主线程等待 3 秒后，守护线程也会自动结束
      time.sleep(3)
      print("Main thread finished.")
  
  # 输出：
  # Thread-1 (worker) running in the background
  # Thread-1 (worker) running in the background
  # Thread-1 (worker) running in the background
  # Main thread finished.  
  ```

* 示例 4: 非守护线程  
  非守护线程的特点是主线程会等待所有非守护线程执行完毕后才会终止：

  ```python
  import threading
  import time
  
  
  def worker():
      for i in range(5):
          print(f"{threading.current_thread().name} task {i}\n")
          time.sleep(1)
  
  
  if __name__ == "__main__":
      # 创建非守护线程
      non_daemon_thread = threading.Thread(target=worker, name="Non-Daemon-Thread", daemon=False)
  
      # 启动非守护线程
      non_daemon_thread.start()
  
      print("Main thread finished. Waiting for non-daemon thread to complete.\n")
      non_daemon_thread.join()  # 主线程等待非守护线程结束
      print("Non-daemon thread finished.\n")
  
  # 输出：
  # Non-Daemon-Thread task 0
  # Main thread finished. Waiting for non-daemon thread to complete.
  # Non-Daemon-Thread task 1
  # Non-Daemon-Thread task 2
  # Non-Daemon-Thread task 3
  # Non-Daemon-Thread task 4
  # Non-daemon thread finished.
  
  # non_daemon_thread.join() 注释后输出：
  # Non-Daemon-Thread task 0
  # Main thread finished. Waiting for non-daemon thread to complete.
  # Non-daemon thread finished.
  # Non-Daemon-Thread task 1
  # Non-Daemon-Thread task 2
  # Non-Daemon-Thread task 3
  # Non-Daemon-Thread task 4
  ```

#### 20.2.2 线程池
Python 3  `concurrent.futures` 模块提供了一个 `ThreadPoolExecutor` 类，用于实现线程池。`ThreadPoolExecutor` 是 `Executor` 的子类，它使用线程池来异步执行调用。

```
class concurrent.futures.ThreadPoolExecutor(max_workers=None, thread_name_prefix='', initializer=None, initargs=())
```

不建议将 ThreadPoolExecutor 用于长期运行的任务：由于 ThreadPoolExecutor 会确保线程池中的所有线程在程序退出之前被合并，如果线程池中的任务是长期运行的（例如无限循环或长时间阻塞的任务），这可能会导致程序在退出时被卡住，因为它要等待这些长期运行的线程完成工作。因此，不建议将 ThreadPoolExecutor 用于长期运行的任务。如果你需要处理长期任务，应该使用其他的解决方案，比如守护线程（daemon threads），或者自己实现更复杂的线程管理机制，来确保程序能够在需要时正常退出，而不是等待长期运行的线程完成。

* `max_workers`(可选)：指定线程池中最大的工作线程数量。如果提交的任务超过了这个数量，多余的任务会排队等待有空闲线程。
   * 如果 max_workers 为 None 或没有指定，将默认为机器处理器的个数，假如 ThreadPoolExecutor 侧重于I/O操作而不是CPU运算，那么可以乘以 5 ，同时工作线程的数量可以比 ProcessPoolExecutor 的数量高；
   * 在 3.8 版本: max_workers 的默认值已改为 min(32, os.cpu_count() + 4)。 这个默认值会保留至少 5 个工作线程用于 I/O 密集型任务。 对于那些释放了 GIL 的 CPU 密集型任务，它最多会使用 32 个 CPU 核心。这样能够避免在多核机器上不知不觉地使用大量资源；
   * 在 3.13 版本: max_workers 的默认值已改为 min(32, (os.process_cpu_count() or 1) + 4)；
* `thread_name_prefix (可选)`： 允许用户控制由线程池创建的 threading.Thread 工作线程名称以方便调试。如果不设置，线程名称会是类似 ThreadPoolExecutor-0_0 这样的格式；
* `initializer (可选)`：每个线程启动时都会执行的初始化函数。这个函数会在每个线程开始处理任务前运行一次，常用于初始化线程特定的资源或环境（例如数据库连接、文件句柄等）；
* `initargs (可选)`：传递给 initializer 函数的参数，作为一个元组传入，默认 `()`；

**ThreadPoolExecutor 常用方法**  

`ThreadPoolExecutor` 是 `Executor` 的子类，Executor 抽象类提供异步执行调用方法，要通过它的子类调用，即 `ThreadPoolExecutor` 常用方法：

* `submit(fn, /, *args, **kwargs)`：向线程池提交一个可调用对象（函数:），并立即返回一个 Future 对象，通过 Future 对象可以获取任务的执行结果；
  ```
  with ThreadPoolExecutor(max_workers=1) as executor:
    future = executor.submit(pow, 323, 1235)
    print(future.result())
  ```
* `map(fn, *iterables, timeout=None, chunksize=1)`：将 function 应用于 iterable 的每一项，并产生其结果的迭代器，timeout 超时时间（可以是整数或浮点数），如果 timeout 未指定或为 None，则不限制等待时间； 
* `shutdown(wait=True, *, cancel_futures=False)`： 停止线程池的工作；
   * wait=True：则此方法只有在所有待执行的 future 对象完成执行且释放已分配的资源后才会返回；
   * wait=False：方法立即返回，所有待执行的 future 对象完成执行后会释放已分配的资源。不管 wait 的值是什么，整个 Python 程序将等到所有待执行的 future 对象完成执行后才退出；
   * cancel_futures=True：此方法将取消所有执行器还未开始运行的挂起的 Future。无论 cancel_futures 的值是什么，任何已完成或正在运行的 Future 都不会被取消；
   * cancel_futures=True，wait=True：已开始运行的所有 Future 将在此方法返回之前完成。 其余的 Future 会被取消；
   * 如果使用 with 语句，可以避免显式调用这个方法，它将会停止 Executor (就好像 Executor.shutdown() 调用时 wait 设为 True 一样等待)：
     ```
     import shutil
     with ThreadPoolExecutor(max_workers=4) as e:
         e.submit(shutil.copy, 'src1.txt', 'dest1.txt')
         e.submit(shutil.copy, 'src2.txt', 'dest2.txt')
         e.submit(shutil.copy, 'src3.txt', 'dest3.txt')
         e.submit(shutil.copy, 'src4.txt', 'dest4.txt')
     ```

【注意】：死锁
当可调用对象已关联了一个 Future 然后在等待另一个 Future 的结果，会导致死锁情况，例如:

```python
import time
# 线程池
from concurrent.futures import ThreadPoolExecutor

def wait_on_b():
    time.sleep(5)
    print(b.result())  # b 永远不会结束因为它在等待 a。
    return 5

def wait_on_a():
    time.sleep(5)
    print(a.result())  # a 永远不会结束因为它在等待 b。
    return 6

executor = ThreadPoolExecutor(max_workers=2)
a = executor.submit(wait_on_b)

# 或者

def wait_on_future():
    f = executor.submit(pow, 5, 2)
    # 这将永远不会完成因为只有一个工作线程
    # 并且它正在执行此函数。
    print(f.result())


executor = ThreadPoolExecutor(max_workers=1)
future = executor.submit(wait_on_future)
result = future.result()
print(result)
```

**示例：**  
* 示例 1：使用 submit() 提交单个任务  
  ```python
  import time
  from concurrent.futures import ThreadPoolExecutor
  
  
  def task(name):
      print(f"Task {name} is running")
      time.sleep(2)
      return f"Task {name} completed"
  
  
  if __name__ == '__main__':
      with ThreadPoolExecutor(max_workers=3) as executor:
          future = executor.submit(task, 'jpzhang')  # 向线程池提交任务并立即返回 Future 对象
          result = future.result()  # 阻塞等待任务执行完成并获取结果
          print(result)
  ```  

* 示例 2：使用 map() 并行处理多个任务  

  ```python
  import time
  from concurrent.futures import ThreadPoolExecutor
  
  
  def task(n):
      print(f"Processing {n}")
      time.sleep(1)
      return n * n
  
  
  if __name__ == '__main__':
      # 创建线程池
      with ThreadPoolExecutor(max_workers=3) as executor:
          nums = [1, 2, 3, 4, 5]
          results = executor.map(task, nums)  # 并行处理多个任务
          for result in results:
              print(result)
  
  # 输出：
  # Processing 1
  # Processing 2
  # Processing 3
  # Processing 4
  # 1Processing 5
  #
  # 4
  # 9
  # 16
  # 25
  ```
  * `executor.map(task, nums)` 对列表 nums 中的每个元素都并行执行了 task() 函数，结果是平方数的迭代器；
  * `map()` 方法返回的是一个结果迭代器，可以使用 for 循环遍历； 

* 示例 3：处理多个异步任务并等待完成  
  ```python
  import time
  from concurrent.futures import ThreadPoolExecutor, as_completed
  
  
  def task(name):
      print(f"Task {name} is running")
      time.sleep(2)
      return f"Task {name} completed"
  
  
  if __name__ == '__main__':
      # 创建线程池
      with ThreadPoolExecutor(max_workers=3) as executor:
          # 提交多个任务
          tasks = [executor.submit(task, f"Task-{i}") for i in range(5)]
          for future in as_completed(tasks):  # 等待每个任务完成
              print(future.result())  # 获取每个任务的结果
  
  # 输出：
  # Task Task-0 is running
  # Task Task-1 is running
  # Task Task-2 is running
  # Task Task-3 is running
  # Task Task-0 completed
  # Task Task-4 is running
  # Task Task-1 completed
  # Task Task-2 completed
  # Task Task-3 completed
  # Task Task-4 completed
  ```
  * executor.submit(task, ...) 提交了多个任务；
  * as_completed() 返回一个迭代器，在每个任务完成时会返回 Future 对象；
  * future.result() 用于获取每个任务的执行结果；

* 示例 4：线程池中的异常处理  
  当任务执行过程中抛出异常时，submit() 返回的 Future 对象可以捕获这些异常。

  ```python
  from concurrent.futures import ThreadPoolExecutor
  
  
  def task(num):
      if num == 2:
          raise ValueError("Task encountered an error!")
      return num * num
  
  
  if __name__ == "__main__":
      with ThreadPoolExecutor(max_workers=3) as executor:
          futures = [executor.submit(task, i) for i in range(5)]
          for future in futures:
              try:
                  print(future.result())
              except Exception as e:
                  print(f"Task raised an exception: {e}")
  
  # 输出:
  # 0
  # 1
  # Task raised an exception: Task encountered an error!
  # 9
  # 16
  ```
  * 当 task(2) 抛出异常时，future.result() 会捕获并抛出该异常，可以在 try-except 块中处理。

* 示例 5：手动关闭线程池  

  ```python
  import time
  from concurrent.futures import ThreadPoolExecutor
  
  
  def task(name):
      print(f"Task {name} is running")
      time.sleep(1)
      return f"Task {name} completed"
  
  
  if __name__ == "__main__":
      # 创建线程池
      executor = ThreadPoolExecutor(max_workers=3)
      futures = [executor.submit(task, f"Task-{i}") for i in range(5)]
  
      # 手动关闭线程池
      executor.shutdown(wait=True)  # 等待所有任务完成
      print("All tasks completed!")
  
  # 输出：
  # Task Task-0 is running
  # Task Task-1 is running
  # Task Task-2 is running
  # Task Task-3 is running
  # Task Task-4 is running
  # All tasks completed!
  
  # executor.shutdown(wait=False) ，立即返回
  # Task Task-0 is running
  # Task Task-1 is running
  # Task Task-2 is runningAll tasks completed!
  #
  # Task Task-3 is running
  # Task Task-4 is running
  ```
  * executor.shutdown() 关闭线程池，并等待所有提交的任务完成。
  * wait=True 表示调用 shutdown() 后会阻塞，直到所有任务完成，否则，立即返回，任务依旧会继续完成。

* 示例 6：
  ```python
  import concurrent.futures
  import urllib.request
  
  URLS = ['http://www.foxnews.com/',
          'http://www.cnn.com/',
          'http://europe.wsj.com/',
          'http://www.bbc.co.uk/',
          'http://nonexistant-subdomain.python.org/']
  
  # 获取一个页面并报告其 URL 和内容
  def load_url(url, timeout):
      with urllib.request.urlopen(url, timeout=timeout) as conn:
          return conn.read()
  
  # 我们可以使用一个 with 语句来确保线程被迅速清理
  with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
      # 开始加载操作并以每个 Future 对象的 URL 对其进行标记
      future_to_url = {executor.submit(load_url, url, 60): url for url in URLS}
      for future in concurrent.futures.as_completed(future_to_url):
          url = future_to_url[future]
          try:
              data = future.result()
          except Exception as exc:
              print('%r generated an exception: %s' % (url, exc))
          else:
              print('%r page is %d bytes' % (url, len(data)))
  ```

#### 20.2.3 GIL（Global Interpreter Lock，全局解释器锁）

在 Python 3 中的多线程主要是并发而不是并行，特别是在使用标准的 CPython 解释器时。这主要是由于“全局解释器锁（GIL）”的存在。

参考：https://zhuanlan.zhihu.com/p/75780308

**GIL（Global Interpreter Lock， 全局解释器锁）**
  * **什么是 GIL？** GIL 是 CPython 解释器中的一个机制，用于保证同一时间只有一个线程执行 Python 字节码。它使得 Python 的多线程在处理 CPU 密集型任务时不能真正实现并行。
  * **影响**：即便系统有多个 CPU 核心，Python 多线程在执行 Python 代码时仍然只能同时运行一个线程（这是因为 GIL 会让每个线程轮流获取执行权限）。这导致了多线程在 CPython 中无法充分利用多核 CPU 进行并行计算，而更多地表现为并发。

**并发 vs 并行**
  * **并发**：并发指的是多个任务在同一时间段内交替进行执行。在一个多线程环境中，线程轮流获取 CPU 时间片执行任务，给人一种同时执行的错觉。CPython 中的多线程是这种情况，尤其是在 CPU 密集型任务中。
  * **并行**：并行指的是多个任务在同一时刻真正同时执行，通常依赖于多核 CPU 来并行处理多个任务。在 Python 中，要真正实现并行的计算，一般使用多进程（如 multiprocessing 模块）而不是多线程。

**多线程的适用场景**
  虽然 GIL 限制了 Python 多线程的并行能力，但在某些场景下，多线程仍然非常有用：
  * I/O 密集型任务：例如网络请求、文件 I/O 等操作。在这些任务中，线程会等待外部资源的响应，而不需要大量 CPU 计算，因此在等待期间可以切换到其他线程进行工作，充分提高程序的执行效率。
  * 轻量级的任务切换：通过多线程实现任务的交替执行，尤其适合处理大量短小的任务。

**如何实现真正的并行**
  如果程序需要真正的并行计算（特别是在 CPU 密集型任务中），通常有以下几个选择：
  * 使用 multiprocessing 模块：这个模块允许创建多个进程，每个进程拥有自己的 GIL，因此可以充分利用多核 CPU 进行并行计算。
  * 使用 C 扩展模块：对于需要高性能的部分，可以使用 C/C++ 编写扩展模块，然后通过 Python 调用。GIL 对非 Python 代码不生效，因此可以通过这种方式实现并行。
  * 使用 concurrent.futures.ProcessPoolExecutor：这个工具通过多进程来实现真正的并行，适合需要执行大量 CPU 密集型任务的场景。

#### 20.2.4 线程同步
多个线程共享资源或数据，如果没有合理的同步机制，可能会导致数据竞争、死锁等问题。因此，线程同步是确保多个线程安全地访问共享资源的关键。

#### 20.2.4.1 Lock (锁)
`Lock` 是最简单的一种锁机制，表示一个互斥锁。锁的状态可以是“锁定”或“非锁定”。当一个线程获取锁时，其他线程必须等待该线程释放锁才能获取它。这可以确保只有一个线程可以访问共享资源，从而避免竞争条件。

```
class threading.Lock
```
锁对象类，一旦一个线程获得一个锁，会阻塞随后尝试获得锁的线程，直到它被释放，任何线程都可以释放它。  

【备注】   
* Lock 锁支持上下文管理协议，因此推荐使用 with 而不是手动调用 acquire() 和 release() 来针对一个代码块处理锁的获取和释放。  
* 在 3.13 版本发生变更: 现在 Lock 是一个类。 在更早的 Python 版本中，Lock 是一个返回下层私有锁类型的实例的工厂函数。  

**基础方法：**  
* `acquire(blocking=True, timeout=-1)`：阻塞或非阻塞获得锁。
  * blocking=True（默认值），阻塞直到锁被释放，然后将锁锁定并返回 True；
  * blocking=False，非阻塞，如果调用时锁未释放，则立即返回 False；否则，将锁锁定并返回 True；
  * timeout：阻塞超时时间，超时未获得锁则返回 False，-1 无限制；
* `release()`：释放一个锁，这个方法可以在任何线程中调用，不单指获得锁的线程；
* `locked()`：当锁被获取时，返回 True；

**示例 1：`acquire`/`release`**
```python
import threading
import time

# 共享资源
counter = 0
lock = threading.Lock()


def increment_counter():
    global counter
    # 获取锁
    lock.acquire()
    try:
        local_counter = counter
        time.sleep(0.1)  # 模拟其他操作
        local_counter += 1
        counter = local_counter
    finally:
        # 释放锁,任务出错，锁也能被释放，防止死锁
        lock.release()


if __name__ == "__main__":
    # 创建线程
    threads = []
    for i in range(5):
        thread = threading.Thread(target=increment_counter)
        threads.append(thread)
        thread.start()

    # 等待所有线程完成
    for thread in threads:
        thread.join()

    print(f"Final counter value: {counter}")

# 输出：
# Final counter value: 5
```

**示例 2：`with`**
```
import threading

lock = threading.Lock()
counter = 0

def task():
    global counter
    with lock:  # 使用 with 自动获取和释放锁
        local_counter = counter
        local_counter += 1
        counter = local_counter
        print(f"Counter value: {counter}")

if __name__ == "__main__":
    threads = []
    for _ in range(5):
        t = threading.Thread(target=task)
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    print(f"Final counter value: {counter}")

# 输出：
# Counter value: 1
# Counter value: 2
# Counter value: 3
# Counter value: 4
# Counter value: 5
# Final counter value: 5
```

#### 20.2.4.2 RLock (递归锁)
RLock 表示可重入锁（reentrant lock），允许同一个线程多次获取同一把锁而不会导致死锁。每次 RLock.acquire() 的调用必须匹配一次 RLock.release()，只有在所有的锁定请求都被释放后，锁才会被真正释放（在 “锁定/非锁定” 状态上附加了 "所属线程" 和 "递归等级" 的概念）。

```
class threading.RLock
```
此类实现了重入锁对象，重入锁必须由获取它的线程释放，一旦线程获得了重入锁，同一个线程再次获取它将不阻塞；需要注意的是 RLock 其实是一个工厂函数，返回平台支持的具体递归锁类中最有效的版本的实例。

【备注】   
* RLock 锁支持上下文管理协议，因此推荐使用 with 而不是手动调用 acquire() 和 release() 来针对一个代码块处理锁的获取和释放。

**基础方法：**  
* `acquire(blocking=True, timeout=-1)`：阻塞或非阻塞获得锁。
  * blocking = True (默认值): 
    * 如无任何线程持有锁，则获取锁并立即返回；
    * 如有其他线程持有锁，则阻塞执行直至能够获取锁，或直至 timeout，如果将其设为一个正浮点数值的话；
    * 如同一线程持有锁，则再次获取该锁，并立即返回。 这是 Lock 和 RLock 之间的区别；Lock 将以与之前相同的方式处理此情况，即阻塞执行直至能够获取锁；
  * blocking = False：
    * 如无任何线程持有锁，则获取锁并立即返回；
    * 如有其他线程持有锁，则立即返回；
    * 如同一线程持有锁，则再次获取该锁并立即返回；

如果被多次调用，则未能调用相同次数的 release() 可能导致死锁。考虑将 RLock 用作上下文管理器而不是直接调用 acquire/release。
* `release()`: 释放锁，自减递归等级。如果减到零，则将锁重置为非锁定状态(不被任何线程拥有)，并且，如果其他线程正被阻塞着等待锁被解锁，则仅允许其中一个线程获得锁。如果自减后，递归等级仍然不是零，则锁保持锁定，仍由调用线程拥有。只有在调用方线程持有锁时才能调用此方法。如果在未获取锁的情况下调用此方法则会引发 RuntimeError。

**示例 1：`acquire`/`release`**
```python
import threading
import time
# 共享资源
counter = 0
rlock = threading.RLock()


def increment():
    global counter
    rlock.acquire()
    try:
        local_counter = counter
        time.sleep(0.1)
        local_counter += 1
        counter = local_counter
    finally:
        rlock.release()


def increment_counter():
    global counter
    # 使用 RLock
    rlock.acquire()
    try:
        increment()  # 递归调用
    finally:
        rlock.release()


if __name__ == "__main__":
    # 创建线程
    threads = []
    for i in range(5):
        thread = threading.Thread(target=increment_counter)
        threads.append(thread)
        thread.start()

    # 等待所有线程完成
    for thread in threads:
        thread.join()

    print(f"Final counter value: {counter}")

# 输出：
# Final counter value: 5
```

**示例 2：`with`**
```python
import threading

lock = threading.Lock()
counter = 0

def task():
    global counter
    with lock:  # 使用 with 自动获取和释放锁
        local_counter = counter
        local_counter += 1
        counter = local_counter
        print(f"Counter value: {counter}")

if __name__ == "__main__":
    threads = []
    for _ in range(5):
        t = threading.Thread(target=task)
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    print(f"Final counter value: {counter}")

# 输出：
# Counter value: 1
# Counter value: 2
# Counter value: 3
# Counter value: 4
# Counter value: 5
# Final counter value: 5
```

**`Lock` 与 `RLock` 的区别**  

| 特性                  | `Lock`                              | `RLock`                           |
|-----------------------|--------------------------------------|------------------------------------|
| 获取锁                | 只能获取一次，必须释放后才能再次获取 | 可以多次获取，适用于递归调用       |
| 适用场景              | 简单互斥操作                         | 复杂递归或重复锁定的场景           |
| 是否支持多次获取      | 否                                   | 是                                |
| 获取锁行为            | `acquire()` 和 `release()` 必须一一对应 | 每次调用 `acquire()` 后可多次调用 `release()` |

#### 20.2.4.3 Condition (条件对象)
`threading.Condition` 是 Python 中用于线程同步的条件变量，常用于多个线程之间协调某个共享资源的访问。通过 Condition，线程可以等待某个条件的发生，直到另一个线程通知它们该条件已被满足，线程才会继续执行。这通常用于生产者-消费者模型或类似的并发模式。

```
class threading.Condition(lock=None)
```
* `lock`: 一个可选的锁（如 Lock 或 RLock）对象。如果不指定，Condition 会内部创建一个 RLock 作为锁对象。

Condition 对象结合了锁机制和条件变量的功能，使用时它必须与锁结合。当线程需要等待某个条件时，它首先会获取锁，然后调用 wait() 方法挂起自己，释放锁，直到其他线程调用 notify() 或 notify_all() 来通知它条件已经满足。

**常用方法**
* wait(timeout=None): 当前线程等待，直到其他线程通知条件变量，或直到可选的 timeout 时间结束为止。调用 wait() 时，线程会释放锁，并在被唤醒或超时后重新获得锁。
* wait_for(predicate, timeout=None)：等待特定条件（predicate: 可调用对象而且它的返回值可被解释为一个布尔值）为 True。它提供了比 wait() 更加智能的等待机制，可以通过一个条件函数（predicate）来判断是否满足条件，而不是单纯依赖于外部 notify() 信号。
* notify(n=1): 通知等待条件的线程，默认唤醒一个线程。如果 n 大于 1，则唤醒指定数量的线程。只有在持有锁的情况下才能调用。如果没有线程在等待，这是一个空操作。
* notify_all(): 唤醒所有等待该条件的线程。
* acquire(), release(): 获取和释放与条件关联的锁，通常在使用 with 语句时自动处理锁的获取和释放。
* 代码片段：
  ```
  # 消费一个条目
  with cv:
      while not an_item_is_available():
          cv.wait()
      get_an_available_item()
  
  # 生产一个条目
  with cv:
      make_an_item_available()
      cv.notify()
  ```
  wait_for()：  
  ```
  # 消费一个条目
  with cv:
      cv.wait_for(an_item_is_available)
      get_an_available_item()
  ```
  
**完整示例：** 生产者-消费者  

```python
import threading
import time

class ProducerConsumer:
    def __init__(self):
        self.items = []  # 用于存储共享数据
        self.condition = threading.Condition()  # 创建条件变量
        self.producer_done = False  # 标志生产者是否完成

    def producer(self):
        for i in range(1, 6):
            with self.condition:  # 每次生产商品时获取条件的锁
                print(f"Producing item {i}")
                self.items.append(i)  # 生产一个商品
                self.condition.notify_all()  # 通知消费者有新商品
            time.sleep(0.5)  # 模拟生产过程
        # 生产完成，标记生产者已完成
        with self.condition:
            self.producer_done = True
            self.condition.notify_all()  # 通知所有消费者，生产结束

    def consumer(self):
        while True:
            with self.condition:
                # 消费者线程被唤醒后，检查是否有商品可消费
                while not self.items and not self.producer_done:  # 如果 items 为空，等待生产者通知
                    print(f"{threading.current_thread().name} - No items to consume, waiting...")
                    self.condition.wait()  # 等待被生产者唤醒
                if not self.items and self.producer_done:  # 生产者完成且没有商品可消费
                    print(f"{threading.current_thread().name} - All items consumed, exiting...")
                    break
                # 消费商品
                item = self.items.pop(0)
                print(f"{threading.current_thread().name} - Consumed item {item}")
            # 这里消费完商品后，已经释放锁，再进行模拟消费的延时
            time.sleep(0.1)  # 模拟消费过程

if __name__ == "__main__":
    pc = ProducerConsumer()

    # 创建多个消费者线程
    consumers = [threading.Thread(target=pc.consumer, name=f"Consumer-{i+1}") for i in range(2)]

    # 创建一个生产者线程
    producer_thread = threading.Thread(target=pc.producer)

    # 启动所有消费者线程
    for consumer in consumers:
        consumer.start()

    time.sleep(0.5)  # 让消费者先启动，模拟现实中的消费等待
    producer_thread.start()

    # 等待所有线程结束
    producer_thread.join()
    for consumer in consumers:
        consumer.join()
```

使用 while 循环检查所要求的条件成立与否是有必要的，因为 wait() 方法可能要经过不确定长度的时间后才会返回，而此时导致 notify() 方法调用的那个条件可能已经不再成立。这是多线程编程所固有的问题。 wait_for() 方法可自动化条件检查，并简化超时计算。

`wait_for()`：示例    

```
import threading
import time


class ProducerConsumer:
    def __init__(self):
        self.items = []  # 共享资源列表
        self.condition = threading.Condition()

    def producer(self):
        with self.condition:
            for i in range(1, 6):
                print(f"Producing item {i}")
                self.items.append(i)  # 生产物品
                self.condition.notify()  # 通知消费者
                time.sleep(1)

    def consumer(self):
        def item_available():  # 谓词函数：检测 items 是否有元素
            return len(self.items) > 0

        
        with self.condition:
            self.condition.wait_for(item_available)  # 等待 items 不为空
            item = self.items.pop(0)  # 消费物品
            print(f"Consumed item {item}")


if __name__ == "__main__":
    pc = ProducerConsumer()

    # 创建消费者线程
    consumer_thread = threading.Thread(target=pc.consumer)
    # 创建生产者线程
    producer_thread = threading.Thread(target=pc.producer)

    consumer_thread.start()
    time.sleep(0.5)  # 模拟消费者等待
    producer_thread.start()

    consumer_thread.join()
    producer_thread.join()
```

#### 20.2.4.4 Semaphore (信号量对象)
Semaphore 用于控制同时进行的线程数量，确保资源不会被超过设定数量的线程同时使用。一个信号量管理一个内部计数器，该计数器因 acquire() 方法的调用而递减，因 release() 方法的调用而递增。 计数器的值永远不会小于零；当 acquire() 方法发现计数器为零时，将会阻塞，直到其它线程调用 release() 方法。

```
class threading.Semaphore(value=1)
```

* Semaphore 是信号量的一个实现，用于保护有限资源，防止多个线程同时访问，或者限制某个代码段的并发执行数量；
* 参数 value: 代表信号量的初始值。value 的默认值为 1，意味着它像一个互斥锁（mutex）一样，仅允许一个线程访问资源。如果设置为大于 1 的值，则表示可以有多个线程同时访问受保护的资源。当 value 为 0 时，表示完全阻塞，所有线程需要等待信号量的释放才能继续执行；
* 信号量对象也支持上下文管理协议（with）；

**示例**
信号量通常用于保护数量有限的资源，例如数据库服务器。在资源数量固定的任何情况下，都应该使用有界信号量。在生成任何工作线程前，应该在主线程中初始化信号量。

```
maxconnections = 5
# ...
pool_sema = BoundedSemaphore(value=maxconnections)
```

工作线程生成后，当需要连接服务器时，这些线程将调用信号量的 acquire 和 release 方法：

```
with pool_sema:
    conn = connectdb()
    try:
        # ... 使用连接 ...
    finally:
        conn.close()
```

使用 Semaphore 控制最多允许 3 个线程同时访问一个共享资源：  

```
import threading
import time

semaphore = threading.Semaphore(2)

def access_resource():
    with semaphore:
        print(f"{threading.current_thread().name} accessing resource.")
        time.sleep(2)

threads = []
for i in range(4):
    t = threading.Thread(target=access_resource)
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

#### 20.2.4.5 Event (事件对象)

Event 对象通过内部的一个布尔标志来实现线程间的协调和通信，当布尔标志为 True 时，所有等待的线程都会被唤醒；当布尔标志为 False 时，所有等待的线程都会被阻塞。   

`class threading.Event`：实现事件对象的类。事件对象管理一个内部标识，调用 set() 方法可将其设置为true。调用 clear() 方法可将其设置为 false 。调用 wait() 方法将进入阻塞直到标识为true。这个标识初始时为 false 。

**主要方法**  
* `set()`：将内部的标志设为 True，表示事件发生，并唤醒所有等待此事件的线程；
* `clear()`：将内部的标志重置为 False，使得后续调用 wait() 的线程进入等待状态；
* `is_set()`：返回内部标志的状态，如果标志为 True 则返回 True，否则返回 False；
* `wait(timeout=None)`：阻塞当前线程，直到内部标志为 True 时唤醒线程。如果设置了 timeout 参数，线程会等待指定的秒数，如果超时还没有被唤醒，则返回 False；

**工作流程**  
* 当 Event 对象的内部标志为 False 时，调用 wait() 的线程会被阻塞，直到其他线程调用 set() 方法将标志设为 True；
* 一旦标志被设为 True，所有调用 wait() 的线程都会被唤醒，并可以继续执行；

**示例**  

示例 1：生产者与消费者之间的事件同步

````python
import threading
import time

# 创建一个事件对象
event = threading.Event()

def producer():
    print("Producer is working...")
    time.sleep(3)  # 模拟生产过程
    print("Producer has produced an item!")
    event.set()  # 生产完成，触发事件

def consumer():
    print("Consumer is waiting for an item...")
    event.wait()  # 等待事件被触发
    print("Consumer has consumed the item!")

if __name__ == "__main__":
    # 创建生产者线程和消费者线程
    producer_thread = threading.Thread(target=producer)
    consumer_thread = threading.Thread(target=consumer)

    # 启动线程
    consumer_thread.start()
    producer_thread.start()

    # 等待线程结束
    producer_thread.join()
    consumer_thread.join()

    print("All done!")

# 输出：
# Consumer is waiting for an item...
# Producer is working...
# Producer has produced an item!
# Consumer has consumed the item!
# All done!
````

* 消费者线程首先调用 event.wait()，进入等待状态；
* 生产者线程经过 3 秒钟后，调用 event.set()，将事件标志设为 True，这时消费者线程被唤醒并继续执行；
* 生产者和消费者同步完成后，程序正常退出；

示例 2：定时触发的事件

```python
import threading
import time

# 创建事件对象
event = threading.Event()


def worker():
    while not event.is_set():
        # 每隔 2 秒检查一次事件是否已触发
        print("Waiting for the event to be set...")
        event.wait(2)
    print("Event has been set, continuing work...")


def trigger_event():
    time.sleep(5)
    print("Setting the event...")
    event.set()


if __name__ == "__main__":
    # 启动工作线程
    worker_thread = threading.Thread(target=worker)
    worker_thread.start()

    # 启动触发事件的线程
    trigger_thread = threading.Thread(target=trigger_event)
    trigger_thread.start()

    # 等待所有线程结束
    worker_thread.join()
    trigger_thread.join()

# 输出：
# Waiting for the event to be set...
# Waiting for the event to be set...
# Waiting for the event to be set...
# Setting the event...
# Event has been set, continuing work...
```

#### 20.2.4.6 Timer (定时器对象)

线程定时器对象 (threading.Timer) 是 threading 模块中的一种高级线程工具，专门用于在特定的时间后执行某个操作。Timer 类继承自 Thread 类，本质上是一个延时启动的线程，允许在指定时间间隔后调用某个函数。

`class threading.Timer(interval, function, args=None, kwargs=None)`：创建一个定时器，经过 “interval” 秒的间隔时间后，将会用参数 “args” 和关键字参数 “kwargs” 调用 “function”。如果 “args” 为 “None” （默认值），则会使用一个空列表。如果 “kwargs” 为 “None” （默认值），则会使用一个空字典。

例如：  

```
def hello():
    print("hello, world")

t = Timer(30.0, hello)
t.start()  # 30 秒之后，将打印 "hello, world"
```

**方法：**  

* start(): 启动定时器，开始倒计时，倒计时结束后执行目标函数；
* cancel(): 如果定时器仍在倒计时中，调用此方法可以取消定时器，避免目标函数的执行；

**示例：**  
```python
import threading
import time


def worker(name, message):
    print(f"Hello, {name}! {message}")


if __name__ == '__main__':
    # 创建一个 3 秒后执行的定时器，并传递参数
    timer = threading.Timer(3, worker, args=("Alice", "How are you today?"))

    print("Timer started!")
    timer.start()  # 启动定时器

    # 继续执行主线程中的任务
    for i in range(5):
        print(f"Main thread working... {i + 1}")
        time.sleep(1)

# 输出：
# Timer started!
# Main thread working... 1
# Main thread working... 2
# Main thread working... 3
# Hello, Alice! How are you today?
# Main thread working... 4
# Main thread working... 5
```

取消定时器：

```python
import threading
import time

def say():
    print("Hello, world!")



if __name__ == '__main__':
    # 创建一个 5 秒后执行的定时器
    timer = threading.Timer(5, say)

    print("Timer started!")
    timer.start()  # 启动定时器

    # 主线程等待 2 秒后取消定时器
    time.sleep(2)
    timer.cancel()
    print("Timer canceled!")

# 输出：
# Timer started!
# Timer canceled!
```

#### 20.2.4.7 Barrier (栅栏对象)
在 Python 3 中，线程栅栏对象 (threading.Barrier) 是一种线程同步机制，能够协调一组线程并确保它们在执行的某个阶段上达到同步。栅栏的作用类似于一个路障，要求所有线程在同一个点等待，直到某个条件满足，然后才能继续执行。

`class threading.Barrier(parties, action=None, timeout=None)`  

* `parties`：必须参数，表示需要等待的线程数。只有指定数量的线程都到达栅栏时，这些线程才会继续执行；
* `action`：可选参数，当最后一个线程到达栅栏时，执行的可选操作（回调函数）；
* `timeout`：可选参数，定义一个栅栏的默认超时时间。如果超过这个时间，等待栅栏的线程会抛出 BrokenBarrierError 异常；

**常用方法**  
* wait(timeout=None)：所有线程调用此方法后，会进入等待状态，直到指定数量的线程都调用了 wait()。当所有线程都到达栅栏后，线程可以继续执行。如果提供了 timeout 参数，则超时后线程抛出 BrokenBarrierError；
  * 函数返回值是一个整数，取值范围在0到 parties -- 1，在每个线程中的返回值不相同。可用于从所有线程中选择唯一的一个线程执行一些特别的工作。例如：
    ```
    i = barrier.wait()
    if i == 0:
        # 只有一个线程需要打印此文本
        print("passed the barrier")
    ```
* reset()：重置栅栏，允许重新使用栅栏，但当前处于等待的线程会抛出 BrokenBarrierError；
* abort()：中止栅栏，任何在等待中的线程都会抛出 BrokenBarrierError；
* broken：检查栅栏是否已被打破（返回布尔值 True 或 False）；
* parties：需要同步的线程的数量；
* n_waiting：当前已达到栅栏，并处于等待状态的线程数量；

**示例**  

使用 Barrier 同步线程：

```python
import threading
import time

def worker(barrier, worker_id):
    print(f"Worker {worker_id} is waiting at the barrier.")
    worker_ready = barrier.wait()  # 等待其他线程到达栅栏
    if worker_ready == 0:  # 选择唯一的一个线程打印如下内容
        print(f"Worker {worker_id} is the last to reach the barrier!")
    print(f"Worker {worker_id} is proceeding.")

if __name__ == "__main__":
    num_threads = 4  # 定义线程数
    barrier = threading.Barrier(num_threads)  # 创建栅栏，线程数为 4

    threads = []
    for i in range(num_threads):
        thread = threading.Thread(target=worker, args=(barrier, i))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    print("All workers have passed the barrier.")
```
设置回调函数：通过 action 参数，可以在最后一个线程到达栅栏（Barrier）时执行一个额外的动作，例如记录日志或进行某种特定操作。

```python
import threading
import time

def barrier_action():
    print("All threads have reached the barrier. Executing action!")

def worker(barrier, worker_id):
    print(f"Worker {worker_id} is waiting at the barrier.")
    barrier.wait()  # 等待其他线程到达栅栏
    print(f"Worker {worker_id} is proceeding.")

if __name__ == "__main__":
    num_threads = 4
    # 创建栅栏，指定 action 为 barrier_action
    barrier = threading.Barrier(num_threads, action=barrier_action)

    threads = []
    for i in range(num_threads):
        thread = threading.Thread(target=worker, args=(barrier, i))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    print("All workers have passed the barrier.")
```

### 20.3 协程 (Coroutine)

协程是 Python 中的一种轻量级并发模型，通过异步编程实现任务的切换。协程与进程和线程不同，它不是由操作系统调度，而是通过程序本身来控制。

详见 21 章节，或者官网：https://docs.python.org/zh-cn/3/library/asyncio.html

**概念：**  
* 协程是一种特殊的生成器，能够在其执行中暂停和恢复。它们可以用 async def 定义，使用 await 关键字在协程内调用其他协程；
* 与传统线程不同，协程不需要多线程的上下文切换，因此开销较小，性能更高；

**协程的定义与运行：**  
定义协程：协程使用 async def 关键字定义
```
async def worker():
    print("Start coroutine")
    # 模拟异步 I/O 操作
    await asyncio.sleep(5)
    print("End coroutine")
```

运行协程：可以使用 asyncio 模块中的事件循环来运行协程

```
import asyncio

async def main():
    # 调用协程
    await worker()


if __name__ == '__main__':
    # 运行事件循环
    asyncio.run(main())
```

**示例：使用协程进行异步编程**  
下面是一个示例，演示如何使用协程来并发处理多个任务：

```python
import asyncio


async def get_data(num):
    """
    定义了一个异步函数，模拟从网络获取数据的过程，使用 await asyncio.sleep(2) 模拟耗时的操作
    """
    print(f"Task {num}: Start fetching data...")
    await asyncio.sleep(2)  # 模拟耗时的网络请求
    print(f"Task {num}: Data fetched!")
    return f"Data {num}"


async def main():
    """
    创建多个协程任务并通过 asyncio.gather() 并发运行它们。gather 会等待所有协程完成，并返回结果
    """
    tasks = [get_data(i) for i in range(1, 4)]  # 创建多个协程任务
    results = await asyncio.gather(*tasks)  # 并发运行所有协程
    print("All tasks completed.")
    print(results)


# 运行事件循环
if __name__ == "__main__":
    """
    运行 main() 协程并启动事件循环
    """
    asyncio.run(main())

# 输出：
# Task 1: Start fetching data...
# Task 2: Start fetching data...
# Task 3: Start fetching data...
# Task 1: Data fetched!
# Task 2: Data fetched!
# Task 3: Data fetched!
# All tasks completed.
# ['Data 1', 'Data 2', 'Data 3']
```

或者：

```
import asyncio


async def task_1():
    print("Task 1 started")
    await asyncio.sleep(2)
    print("Task 1 completed")


async def task_2():
    print("Task 2 started")
    await asyncio.sleep(1)
    print("Task 2 completed")


async def main():
    # 并发执行多个任务
    await asyncio.gather(task_1(), task_2())


# 启动事件循环
asyncio.run(main())
```

或者：

```python
import asyncio


async def task_1():
    print("Task 1 started")
    await asyncio.sleep(2)
    print("Task 1 finished")


async def task_2():
    print("Task 2 started")
    await asyncio.sleep(1)
    print("Task 2 finished")


async def main():
    task1 = asyncio.create_task(task_1())
    task2 = asyncio.create_task(task_2())

    await task1
    await task2


asyncio.run(main())


# 输出： task_1 需要 2 秒完成，但 task_2 只需要 1 秒，因此它会先完成
# Task 1 started
# Task 2 started
# Task 2 finished
# Task 1 finished
```

**常见 asyncio 函数**  
  * `asyncio.create_task(coro)`: 将协程封装为 Task，可以让事件循环并发地执行多个任务；
  * `asyncio.gather(*coros)`: 并发地运行多个协程，并收集结果；
  * `asyncio.sleep(seconds)`: 模拟异步等待，通常用于测试；
  * `asyncio.run(coro)`: 运行协程，自动处理事件循环的创建和关闭；

**协程优势**  
  * 非阻塞：在等待 I/O 操作时，协程不会阻塞其他任务的执行，可以有效提高程序的并发能力；
  * 轻量级：协程比线程占用的内存更少，创建和切换协程的开销更小；
  * 简洁：通过 async/await 语法，异步代码更易于阅读和理解；

**适用场景**  
  * 高并发 I/O 密集型任务（如网络请求、文件读写等）；
  * 需要频繁进行异步操作的应用（如 Web 服务器、爬虫等）；

**注意事项**
  * 协程只能在异步上下文中运行，不能直接在普通函数中调用；
  * 需要使用 asyncio.run() 或者创建一个事件循环来调度协程的执行；

### 20.4 并行 & 并发

在计算机科学中，“并行”和“并发”是两种不同的概念，它们虽然都涉及到多任务处理，但在执行任务的方式上有着明显的区别。下面分别对这两个概念进行解释：

#### 20.4.1 并行 (Parallelism)

并行指的是多个任务同时在多个处理器或 CPU 核心上运行。并行任务的执行方式是同时进行的，多个任务真正地同时执行，以此来提高程序的执行效率。

**特点：**

* 物理并行：需要多个 CPU 核心或处理器来实现，因为每个任务会在不同的核心上同时运行；
* 同时执行：所有任务在同一时刻运行，并行计算可以提高程序执行的速度，尤其是在多核 CPU 环境下；
* 适用于 CPU 密集型任务：并行更适合需要大量计算的任务，如科学计算、图像处理等；

**举例：**  
在多核处理器中，一个核心执行任务 A，另一个核心同时执行任务 B，这就是并行。比如图像渲染或视频编码时，每个帧可以分配给不同的处理器核心。

**图示：**

```
任务 A |=====| 
任务 B |=====| 
任务 C |=====| 

同一时刻多个任务同时执行。
```

#### 20.4.2 并发 (Concurrency)

并发是指在一个时间段内，有多个任务可以交替执行。并发不一定要求多个任务同时进行，而是多个任务在多个时间片内切换执行。它的目的是提高系统的响应性，而不是直接加快任务的执行速度。

**特点：**

* 任务交替执行：即使只有一个 CPU 核心，多个任务也可以通过切换时间片的方式交替执行，达到 “同时处理多个任务” 的效果；
* 逻辑并行：在单核处理器上，并发任务实际上是按顺序执行的，但由于任务切换速度非常快，用户感觉像是同时执行的；
* 适用于 I/O 密集型任务：并发适合需要等待 I/O 操作的任务，如网络请求、文件读写等，因为任务在等待时可以让出 CPU 资源，让其他任务继续执行；

**举例：**  
一个操作系统同时处理多个任务，比如用户在浏览网页的同时，音乐播放器在后台播放音乐。这些任务实际上是交替运行的，操作系统快速切换任务，使得用户感觉它们是在同时进行。

**图示：**

```
任务 A |==    ==    ==|
任务 B |  ==    ==    ==|
任务 C |    ==    ==    ==|

任务 A、B、C 轮流执行，在切换时间片时交替运行。
```

#### 20.4.3 并行与并发的区别

| 特性 | 并行 (Parallelism)  | 	   并发 (Concurrency)       |
|:---| :---------|:---------|
| 核心概念   | 多个任务真正同时执行 | 多个任务交替执行 |
| 依赖硬件   | 依赖多核 CPU 或多处理器 | 可以在单核或多核 CPU 上实现 |
| 适用场景   | CPU 密集型任务 (如计算、渲染) | I/O 密集型任务 (如网络请求、文件读写) |
| 执行效率   | 提高任务执行的速度 | 提高系统响应性 |
| 执行方式   | 物理上同时运行 | 逻辑上同时运行 |
| 资源共享   | 每个任务可能运行在独立的资源上 | 任务之间共享资源或切换使用资源 |
| 代表模型   | 多进程 (Multiprocessing) | 多线程、协程 |

**总结：**

* 并行是多任务真正同时在不同的处理器上运行，需要硬件支持多核 CPU；
* 并发是在同一时间段内交替执行多个任务，通常不需要多个核心，而是通过任务切换实现任务的同时处理效果；

可以根据程序的需求来选择并行或并发。例如，如果需要处理大量计算，可以选择并行；如果任务涉及大量 I/O 操作，选择并发会更合适。

### 20.5 进程、线程、协程对比

| 特性 | 进程  | 	   线程       | 	   协程       |
|:---| :---------|:---------|:---------|
|  并行/并发  | 并行 | 并发 | 并发 |
|  内存空间  | 独立 | 共享 | 共享 |
|  创建开销  | 高 | 中 | 低 |
|  通信方式  | IPC (管道等) | 共享变量 | 事件循环 |
|  GIL 限制  | 无 | 有 (CPython) | 无 |
|  适用场景  | CPU 密集型任务 | I/O 密集型任务 | I/O 密集型任务 |

总之，进程适合 CPU 密集型任务，而线程和协程适合 I/O 密集型任务。协程是目前在 Python 中实现高并发的首选方式，尤其适用于网络应用和异步操作。
可以根据具体的应用场景选择适合的并发模型。进程是指在系统中正在运行的一个应用程序，是CPU的最小工作单元，一个进程可以有一个或多个线程，一个线程可以有很多协程。

## 21. asyncio 异步 I/O

Python 的 asyncio 模块是用于编写并发代码的库，它使用 async/await 语法来实现异步 I/O 操作。异步 I/O 允许在等待 I/O 操作完成时执行其他任务，从而提高程序的性能，特别是在 I/O 密集型任务（如网络请求、文件读写）中效果显著。

**核心概念：**  
* `事件循环 (Event Loop)`: asyncio 的核心，负责调度和执行协程。事件循环会不断运行，监听 I/O 操作完成后触发的事件；
* `协程 (Coroutine)`: 是可以被挂起和恢复的函数，通常使用 async def 定义，并通过 await 关键字挂起协程直到异步操作完成；
* `任务 (Task)`: Task 是由事件循环执行的协程。通过将协程封装为任务，事件循环可以并发地运行多个任务；
* `Future`: 类似于 JavaScript 中的 Promise，表示一个将来完成的操作。通常你不需要直接操作 Future，而是通过协程和任务来使用它；

示例：  

```python
import asyncio

async def main():
    print('Hello ...')
    await asyncio.sleep(1)
    print('... World!')

asyncio.run(main())
```

在上述例子中，main 是一个异步协程，调用 asyncio.sleep(1) 会暂停 1 秒，并让出控制权给事件循环。之后，事件循环继续执行下一个任务，1 秒后再恢复这个协程的执行。

**asyncio REPL**  
在 Python 中运行 `python -m asyncio` 启动 asyncio 调试模式。启动后，进入到 asyncio 的交互环境，方便进行一些异步函数的简单测试。

可以在 REPL 中尝试使用 asyncio 并发上下文：

```
$ python -m asyncio
asyncio REPL 3.12.3 (main, Apr 27 2024, 19:00:26) [GCC 9.4.0] on linux
Use "await" directly instead of "asyncio.run()".
Type "help", "copyright", "credits" or "license" for more information.
>>> import asyncio
>>> await asyncio.sleep(10, result='hello')
'hello'
```

### 21.1 运行 asyncio 程序

* `asyncio.run(coro, *, debug=None, loop_factory=None)`  

  此函数会运行传入的协程(coroutine) coro 并返回结果，负责管理 asyncio 事件循环，终结异步生成器，并关闭执行器。当有其他 asyncio 事件循环在同一线程中运行时，此函数不能被调用。
  
  * `coro`：要运行的协程，必须是一个协程对象，通常是由 async def 定义的函数；
  * `debug`：如果设置为 True，事件循环将运行于调试模式。可以通过传递 True 或 False 来启用或禁用调试功能，也可以设置为 None（默认值），将沿用全局 Debug 模式设置；
  * `loop_factory`：工厂函数，用于创建自定义事件循环。如果传递了该参数，asyncio.run() 会使用该工厂函数生成事件循环，默认 loop_factory 为 None，则使用 asyncio.new_event_loop() 并通过 asyncio.set_event_loop() 将其设置为当前事件循环；
    假设使用 [uvloop](https://pypi.org/project/uvloop/) 作为 asyncio 的事件循环：
    ```python
    import asyncio
    import uvloop
        
    async def task():
        print("Task started")
        await asyncio.sleep(1)
        print("Task finished")
    
    # 使用自定义的事件循环工厂
    asyncio.run(task(), loop_factory=uvloop.new_event_loop)
    ```

* `class asyncio.Runner(*, debug=None, loop_factory=None)`  
  在相同上下文中，多个异步函数调用可通过上下文管理器进行简化。`debug`、`loop_factory` 与 `asyncio.run` 一样。
  通过上下文管理器重写 `asyncio.run()` 示例：
  ```python
  import asyncio

  async def main():
      await asyncio.sleep(1)
      print('hello')
  
  
  with asyncio.Runner() as runner:
      runner.run(main())
  ```
  
  示例：
  ```
  import asyncio
  import uvloop
  
  
  async def task():
      print("Task started")
      await asyncio.sleep(1)
      print("Task finished")
  
  
  # 创建 Runner 实例，使用自定义的事件循环工厂
  runner = asyncio.Runner(loop_factory=uvloop.new_event_loop)
  
  try:
      runner.run(task())
  finally:
      runner.close()
  ```
  
  `asyncio.Runner` 方法：  
    * run(coro, *, context=None)：运行一个协程，返回协程的结果或者引发其异常。该方法类似于 asyncio.run()，但需要在 Runner 实例的上下文中调用。
      * 参数 context 允许指定一个自定义 contextvars.Context 用作 coro 运行所在的上下文。 如果为 None 则会使用运行器的默认上下文。
    * runner.close()：关闭事件循环，释放相关资源。
    * get_loop()：返回运行器实例的事件循环。

### 21.2 协程与任务

#### 21.2.1 协程
通过 async/await 语法声明协程，是编写 asyncio 应用的推荐方式。

例如：

```
In [1]: import asyncio

In [2]: async def main():
   ...:     print('hello')
   ...:     await asyncio.sleep(1)
   ...:     print('world')
   ...: 

In [3]: main()
Out[3]: <coroutine object main at 0x7fe01b04a680>

In [4]: asyncio.run(main())
hello
world

In [5]: await main()
hello
world
```

如上，简单通过 main() 调用并不会执行协程，要运行一个协程，asyncio 提供以下机制：

* `asyncio.run()`：  
  ```
  asyncio.run(main())
  ```
* `await`：  
  ```
  import asyncio
  import time
  
  async def say_after(delay, what):
      await asyncio.sleep(delay)
      print(what)
  
  async def main():
      print(f"started at {time.strftime('%X')}")
  
      await say_after(1, 'hello')
      await say_after(2, 'world')
  
      print(f"finished at {time.strftime('%X')}")
  
  asyncio.run(main())
  
  # 输出：
  # started at 14:00:29
  # hello
  # world
  # finished at 14:00:32
  ```
  运行时间 3 秒。
* `asyncio.create_task()`：  
  `asyncio.create_task(coro, *, name=None, context=None)`:将 coro 协程封装为一个 Task 并调度其执行，返回 Task 对象。  
  并发运行多个协程，如，修改以上示例，并发运行两个 say_after 协程:
  ```python
  async def main():
      task1 = asyncio.create_task(say_after(1, 'hello'))
      task2 = asyncio.create_task(say_after(2, 'world'))
      print(f"started at {time.strftime('%X')}")
  
      # 等待直到两个任务都完成
      # （会花费约 2 秒钟。）
      await task1
      await task2
      print(f"finished at {time.strftime('%X')}")
  
  asyncio.run(main())
  
  # 输出：
  # started at 14:08:53
  # hello
  # world
  # finished at 14:08:55
  ```
  并发起作用，运行时间 2 秒，快上 1 秒。
* `asyncio.TaskGroup`:  
  任务分组异步上下文管理器，可以使用 create_task() 将任务添加到分组中。
  ```python
  async def main():
      async with asyncio.TaskGroup() as tg:
          task1 = tg.create_task(say_after(1, 'hello'))
          task2 = tg.create_task(say_after(2, 'world'))
          print(f"started at {time.strftime('%X')}")
  
      # 当存在上下文管理器时 await 是隐式执行的。
      print(f"finished at {time.strftime('%X')}")
      
  asyncio.run(main())
  
  # 输出：
  # started at 14:24:42
  # hello
  # world
  # finished at 14:24:44
  ```
  
#### 21.2.2 任务 

#### 21.2.2.1 可等待对象  

如果一个对象可以在 await 语句中使用，那么它就是可等待对象。可等待对象有三种主要类型：协程, 任务 和 Future。  

* 协程  
  Python 协程属于可等待对象，因此可以在其他协程中被等待:
  ```python
  import asyncio
  
  async def nested():
      return 42
  
  async def main():
      # nested()  # Raise：RuntimeWarning
      print(await nested())  # 输出 "42"
  
  asyncio.run(main())
  ```
  注意：
    * 协程函数: 定义形式为 async def 的函数;
    * 协程对象: 调用 协程函数 所返回的对象；

* Futures  
  Future 对象表示一个异步操作的最终结果。可以看作一个占位符，表示稍后会提供结果的操作。可以用于等待异步操作的完成或从异步操作中获取结果。在 asyncio 中，Future 对象通常由事件循环创建和管理。asyncio 的协程会隐式地与 Future 对象交互，最终由事件循环将结果设置到 Future 上。主要用于低层次的操作，例如等待某个异步操作的结果。开发者通常不会直接创建 Future 对象，而是通过 await 来等待异步操作。

  **核心方法和属性**   
    * `future.result()`：获取异步操作的结果，如果结果尚未准备好，会抛出异常；
    * `future.set_result(value)`：手动设置 Future 对象的结果；
    * `future.set_exception(exception)`：手动为 Future 设置一个异常；
    * `future.done()`：返回 True，表示 Future 已经完成（无论是正常结束还是异常）；
    * `future.add_done_callback(fn)`：当 Future 完成时，注册一个回调函数 fn 来处理结果；
  
  示例：手动创建和完成 Future  
  
  ```python
  import asyncio


  async def set_future_value(fut):
      await asyncio.sleep(1)  # 模拟异步操作
      fut.set_result("Future result")  # 设置 Future 的结果
  
  
  async def main():
      # 创建 Future 对象
      fut = asyncio.Future()
  
      # 启动异步任务来设置 Future 的值
      await asyncio.create_task(set_future_value(fut))
  
      # 获取 Future 的结果
      print(fut.result())  # 输出: Future result
  
  
  # 使用 asyncio.run 运行协程
  asyncio.run(main())
  ```

* 任务  
  任务（Tasks）是 asyncio 中的一种高级对象，它封装了一个协程并允许将其加入事件循环中执行。与 Future 不同，Task 本质上是一个 Future，但它主要用于调度和运行协程。
  ```python
  import asyncio
  
  async def nested():
      return 42
  
  async def main():
      task = asyncio.create_task(nested())
      result = await task
      print(result)
  asyncio.run(main())
  ```

#### 21.2.2.2 创建任务

`asyncio.create_task(coro, *, name=None, context=None)`: 是 asyncio 提供的一个函数，用于将协程（coroutine）封装为 Task 并将其调度到事件循环中执行。这个函数可以让协程以异步的方式执行，而不需要阻塞当前的代码。
返回值是一个 Task 对象，即 asyncio.Task 实例。任务一旦创建，事件循环将会立即开始执行它，直到任务完成或被取消。

* coro（必须）：要封装并调度的协程对象，它是一个异步函数的调用，该协程会立即被调度，加入到事件循环中执行；
* name=None（可选）：任务的名称。可以为任务设置一个字符串名称，便于调试和日志记录，任务名称可以帮助你在调试时更容易识别任务，如果不设置，默认任务没有名字；
* context=None（可选）：传递给 Task 的上下文（Context），允许在不同的 Task 之间共享上下文对象，它提供了异步任务的上下文管理，方便追踪状态、异常等，不常用，默认使用当前上下文；

**任务的调度**：调用 asyncio.create_task() 后，协程 coro 不会同步阻塞当前的函数，它会被放入事件循环中，并且一旦事件循环准备好执行该任务，任务将自动开始。
**与 await 的区别**：await 是同步等待协程执行完的结果，而 create_task() 是异步的，不会阻塞当前函数，会让协程并行执行。

* 示例 1：基本用法
  ```
  import asyncio
  
  
  async def task():
      print("Task started")
      await asyncio.sleep(2)
      print("Task finished")
  
  
  async def main():
      # 创建并调度任务
      task1 = asyncio.create_task(task())
  
      # 主函数不会等待任务完成，继续执行
      print("Task is running in the background")
  
      # 等待任务完成（可选的）
      await task1
  
  asyncio.run(main())
  
  # 输出：
  # Task is running in the background
  # Task started
  # Task finished
  ```

* 示例 2：设置任务名称

  ```python
  import asyncio
  
  
  async def my_named_task():
      await asyncio.sleep(1)
      print("Named Task completed")
  
  
  async def main():
      # 创建并命名任务
      task = asyncio.create_task(my_named_task(), name="MyUniqueTask")
      print(f"Task Name: {task.get_name()}")  # 输出任务的名称
      await task
  
  
  asyncio.run(main())
  ```

* 示例 3：任务与上下文（context）

  对于 context 的使用，涉及 contextvars 模块，可以在异步任务之间传递上下文信息，一个典型的例子是追踪请求的状态或 ID。

  ```python
  import asyncio
  import contextvars
  
  # 创建一个上下文变量
  request_id = contextvars.ContextVar('request_id')
  
  
  async def task_with_context():
      # 在任务中读取上下文变量
      print(f"Task request_id: {request_id.get()}")
      await asyncio.sleep(1)
  
  
  async def main():
      # 创建新的上下文
      ctx = contextvars.copy_context()
  
      # 设置上下文变量的值
      ctx.run(request_id.set, "12345")
  
      # 使用设置好的上下文来创建任务
      task = asyncio.create_task(task_with_context(), context=ctx)
  
      await task
  
  
  # 运行主协程
  asyncio.run(main())
  
  # 输出：
  # Task request_id: 12345
  ```

  * contextvars.ContextVar：这是一个可以在协程之间共享的变量类型。在不同的协程中可以有独立的值，并且不会相互干扰；
  * contextvars.copy_context()：这个函数用于复制当前上下文，生成一个新的上下文。在新上下文中可以进行变量设置并传递给任务；
  * ctx.run(request_id.set, "12345")：使用上下文的 run 方法来设置 request_id 变量的值；
  * asyncio.create_task(..., context=ctx)：将创建的上下文 ctx 传递给 create_task，使得这个任务在执行过程中使用这个上下文；

  通过上下文管理，可以在异步任务之间共享状态或变量，不会造成并发数据污染。

* 示例 4：取消任务  

  ```python
  import asyncio
  
  
  async def my_task():
      print("Task started")
      try:
          await asyncio.sleep(5)  # 模拟长时间运行的任务
      except asyncio.CancelledError:
          print("Task was cancelled!")
          raise
      print("Task finished")
  
  
  async def main():
      task = asyncio.create_task(my_task())
  
      await asyncio.sleep(2)  # 让任务执行一段时间
      task.cancel()  # 取消任务
  
      try:
          await task  # 等待任务完成并捕获取消异常
      except asyncio.CancelledError:
          print("Task has been cancelled")
  
  
  # 使用 asyncio.run 运行协程
  asyncio.run(main())
  
  # 输出：
  # Task started
  # Task was cancelled!
  # Task has been cancelled
  ```

#### 21.2.2.3 任务组
asyncio.TaskGroup 是 Python 3.11 引入的新特性，提供了一种结构化的方式来管理异步任务组。它旨在简化多个任务的创建和管理，增强了异常处理机制，使得在并发编程中处理任务更安全和高效。与传统的 asyncio.gather() 和 asyncio.wait() 不同，TaskGroup 通过上下文管理器的方式来确保所有任务的生命周期都在同一个范围内被创建和管理。

* create_task(coro, *, name=None)：该方法与 asyncio.create_task() 类似，在任务组中创建并调度协程。返回一个 Task 对象；
* __aenter__() 和 __aexit__()：支持上下文管理，当退出上下文时会确保所有任务都已完成或者已经被取消；
* 异常传播：如果某个任务抛出异常，任务组会自动取消其他所有任务，并将异常抛出给调用者；

**示例**  

* 示例 1：基本用法示例  
  ```python
  import asyncio
  
  async def task(num):
      await asyncio.sleep(1)
      print(f"Task {num} done")
      return f"Result {num}"
  
  async def main():
      # 使用 TaskGroup 管理任务组
      async with asyncio.TaskGroup() as tg:
          # 创建并调度任务
          task1 = tg.create_task(task(1))
          task2 = tg.create_task(task(2))
          task3 = tg.create_task(task(3))
  
      # 在 TaskGroup 上下文结束后，所有任务已经完成
      print(f"Task 1: {task1.result()},Task 2: {task2.result()},Task 3: {task3.result()}")
  
  # 运行主协程
  asyncio.run(main())
  # 输出：
  # Task 1 done
  # Task 2 done
  # Task 3 done
  # Task 1: Result 1,Task 2: Result 2,Task 3: Result 3
  ```

* 示例 2：异常处理  
  asyncio.TaskGroup 提供了自动化的异常处理机制。如果任务组中的某个任务抛出异常，TaskGroup 会自动取消其他任务，并且该异常会传播到调用者。
  
  ```python
  import asyncio
  
  
  async def task(num):
      await asyncio.sleep(num)
      if num == 2:
          raise ValueError("Task 2 encountered an error!")
      print(f"Task {num} done")
      return f"Result {num}"
  
  
  async def main():
      try:
          # 使用 TaskGroup 管理任务组
          async with asyncio.TaskGroup() as tg:
              # 创建并调度任务
              task1 = tg.create_task(task(1))
              task2 = tg.create_task(task(2))  # 这个任务会抛出异常
              task3 = tg.create_task(task(3))
      except Exception as e:
          print(f"An exception occurred: {e}")
  
      # 在 TaskGroup 上下文结束后，所有任务已经完成
      print(f"Task 1: {task1.result()},Task 2: {task2.result()},Task 3: {task3.result()}")
  
  
  # 运行主协程
  asyncio.run(main())
  ```
  * 当 task2 抛出异常时，TaskGroup 自动捕获该异常并取消 task3，因此 task3 不会完成；
  * TaskGroup 会自动传播异常给调用者，因此外层的 try/except 块能够捕获并处理该异常；
  
  或：  
  ```python
  import asyncio
  from asyncio import TaskGroup
  
  
  class TerminateTaskGroup(Exception):
      """Exception raised to terminate a task group."""
  
  
  async def force_terminate_task_group():
      """Used to force termination of a task group."""
      raise TerminateTaskGroup()
  
  
  async def job(task_id, sleep_time):
      print(f'Task {task_id}: start')
      await asyncio.sleep(sleep_time)
      print(f'Task {task_id}: done')
  
  
  async def main():
      try:
          async with TaskGroup() as group:
              # spawn some tasks
              group.create_task(job(1, 0.5))
              group.create_task(job(2, 1.5))
              # sleep for 1 second
              await asyncio.sleep(1)
              # add an exception-raising task to force the group to terminate
              group.create_task(force_terminate_task_group())
      except* TerminateTaskGroup:
          pass
  
  
  asyncio.run(main())
  ```

* 示例 3：自动取消任务示例  
  如果某个任务抛出异常，TaskGroup 会立即取消其他所有正在运行的任务，这有助于避免在错误条件下无用的任务继续运行。
  
  ```python
  import asyncio
  
  async def task_1():
      await asyncio.sleep(1)
      print("Task 1 done")
      return "Result 1"
  
  async def task_2():
      await asyncio.sleep(2)
      raise ValueError("Task 2 failed!")
  
  async def task_3():
      try:
          await asyncio.sleep(3)
          print("Task 3 done")
      except asyncio.CancelledError:
          print("Task 3 was cancelled!")
  
  async def main():
      try:
          async with asyncio.TaskGroup() as tg:
              tg.create_task(task_1())
              tg.create_task(task_2())  # This will raise an exception
              tg.create_task(task_3())
      except Exception as e:
          print(f"An exception occurred: {e}")
  
  # 运行主协程
  asyncio.run(main())
  
  # 输出：
  # Task 1 done
  # Task 3 was cancelled!
  # An exception occurred: unhandled errors in a TaskGroup (1 sub-exception)
  ```
  当 task_2 发生异常时，TaskGroup 取消了所有其他任务，因此 task_3 被取消并捕获到了 CancelledError。
  
#### 21.2.2.4 休眠

`coroutine asyncio.sleep(delay, result=None)` 阻塞 delay 指定的秒数，如果指定了 result，则当协程完成时将其返回给调用者。

以下协程示例运行 5 秒，每秒显示一次当前日期：  

```python
import asyncio
import datetime

async def display_date():
    loop = asyncio.get_running_loop()
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

asyncio.run(display_date())
```
  * asyncio.get_running_loop()：获取当前正在运行的事件循环。如果没有正在运行的事件循环抛出 RuntimeError；
  * loop.time()：事件循环中的一个高精度时钟，用来获取事件循环的当前时间。与 time.time() 不同，loop.time() 返回的是相对时间，并且不会受系统时间修改的影响，因此非常适合用于异步编程中的定时任务；

#### 21.2.2.5 并发运行任务

`asyncio.gather(*aws, return_exceptions=False)`， 并发运行 aws 序列中的可等待对象。

**参数说明：**  
* `*aws`：传递多个可等待对象（协程、任务、Future 等），这些任务将会并发运行，asyncio.gather() 会等待所有的任务完成，并返回每个任务的结果，结果的顺序与传入的任务顺序相同。
* `return_exceptions`（可选，默认为 False）：
  * 当 False 时，任何一个任务抛出的异常会被立即传播并中止 gather，未完成的任务会被取消；
  * 当 True 时，异常不会立即传播，而是作为结果的一部分返回，这样即使某些任务失败，gather 仍然会等待其他任务的结果，异常对象将作为返回值列表中的一部分返回；

**返回值：**  
* 如果所有任务成功完成，asyncio.gather() 会返回一个包含每个任务结果的列表，顺序与传入的任务相同；
* 如果设置 return_exceptions=True，并且某些任务抛出异常，则返回的列表会包含异常对象；

**示例：**  

* 示例 1：基本用法  
  ```python
  import asyncio
  
  async def task(num):
      await asyncio.sleep(num)
      print(f"Task {num} completed")
      return f"Num {num}"
  
  async def main():
      # gather 可以并行运行多个任务
      tasks = []
      for i in range(1,4):
          tasks.append(task(i))
      results = await asyncio.gather(*tasks)
      print(f"Results: {results}")
  
  asyncio.run(main())
  
  # 输出：
  # Task 1 completed
  # Task 2 completed
  # Task 3 completed
  # Results: ['Num 1', 'Num 2', 'Num 3']
  ```  

* 示例 2：异常示例（return_exceptions = False）  
  ```
  import asyncio
  
  
  async def task(num):
      await asyncio.sleep(num)
      if num == 2:
          # 模拟异常处理
          raise ValueError(f"Task {num} failed!")
      print(f"Task {num} completed")
      return f"Num {num}"
  
  
  async def main():
      # gather 可以并行运行多个任务
      tasks = []
      for i in range(1,4):
          tasks.append(task(i))
      try:
          results = await asyncio.gather(*tasks)
          print(f"Results: {results}")
      except Exception as e:
          print(f"An error occurred: {e}")
  
  asyncio.run(main())
  
  # 输出：
  # Task 1 completed
  # An error occurred: Task 2 failed!
  ```

* 示例 3：异常示例（return_exceptions = True）  
  ```python
  import asyncio
  
  
  async def task(num):
      await asyncio.sleep(num)
      if num == 2:
          # 模拟异常处理
          raise ValueError(f"Task {num} failed!")
      print(f"Task {num} completed")
      return f"Num {num}"
  
  
  async def main():
      # gather 可以并行运行多个任务
      tasks = []
      for i in range(1, 4):
          tasks.append(task(i))
      try:
          results = await asyncio.gather(*tasks, return_exceptions=True)
          print(f"Results: {results}")
      except Exception as e:
          print(f"An error occurred: {e}")
  
  
  asyncio.run(main())
  
  # 输出：
  # Task 1 completed
  # Task 3 completed
  # Results: ['Num 1', ValueError('Task 2 failed!'), 'Num 3']
  ```
  
* 示例 4：结合 I/O 操作的并发下载    
  假设有多个 URL，需要并发下载它们，可以用 asyncio.gather() 来并发执行多个下载任务：
  ```python
  import asyncio
  import aiohttp
  
  
  async def fetch(session, url):
      async with session.get(url) as response:
          print(f"Fetching: {url}")
          return await response.text()
  
  
  async def main():
      urls = [
          "https://example.com",
          "https://httpbin.org/get",
          "https://jsonplaceholder.typicode.com/posts"
      ]
  
      async with aiohttp.ClientSession() as session:
          results = await asyncio.gather(
              *(fetch(session, url) for url in urls)
          )
          for i, content in enumerate(results):
              print(f"Content {i + 1} length: {len(content)}")
  
  
  asyncio.run(main())
  ```
  使用 aiohttp 库来并发下载多个 URL 的内容，并使用 asyncio.gather() 来并发运行这些下载任务。gather 会等所有的下载任务完成，然后返回每个下载任务的结果。

#### 21.2.2.6 任务 “立即执行”
`asyncio.create_eager_task_factory()` 和 `asyncio.eager_task_factory()` 提供了一种任务的主动调度，而不是被动等待。
通常情况下，像 asyncio.create_task() 方法创建的任务虽然被加入事件循环，但它们的实际执行依赖于事件循环的调度顺序。而 eager_task_factory 提供的 API 强调的是任务被立即执行，即在可能的情况下尽早进入执行状态。

* `asyncio.eager_task_factory(loop, coro, *, name=None, context=None)`  
  用于立即创建并调度任务。当使用这个工厂函数时 (通过 loop.set_task_factory(asyncio.eager_task_factory))，协程将在 Task 构造期间同步地开始执行。任务仅会在它们阻塞时被加入事件循环上的计划任务。这可以达成性能提升因为对同步完成的协程来说可以避免循环调度的开销。

  **参数说明**  
  * `loop`：事件循环实例，任务会在这个事件循环中被调度；
  * `coro`：需要执行的协程对象；
  * `name`（可选）：为任务指定一个名称，有助于调试；
  * `context`（可选）：任务的上下文对象，用于处理任务运行时的上下文；
  
  **返回值：**  
  * 返回一个 Task 对象，该任务立即调度并开始运行；

  **示例：**  
  ```python
  import asyncio
  
  async def task():
      await asyncio.sleep(1)
      print("Task completed")
  
  async def main():
      loop = asyncio.get_running_loop()
      # 立即创建并调度任务
      task1 = asyncio.eager_task_factory(loop, task(), name="MyTask")
      await task1
  
  asyncio.run(main())
  ```
  
  **性能对比：**  
  
  ```python
  import asyncio
  import time
  
  
  async def light_coro():
      pass
  
  
  async def main():
      print('Before running task group!')
      time0 = time.time()
      asyncio.get_event_loop().set_task_factory(asyncio.eager_task_factory)
      async with asyncio.TaskGroup() as tg:
          for _ in range(1000000):
              tg.create_task(light_coro())
      print(f'It took {time.time() - time0} to run!')
      print('After running task group with eager task factory!')
  
  asyncio.run(main())
  
  # 输出：
  # Before running task group!
  # It took 1.201669692993164 to run!
  # After running task group with eager task factory!
  ```
  
  注释：`asyncio.get_event_loop().set_task_factory(asyncio.eager_task_factory)`  

  ```python
  import asyncio
  import time
  
  
  async def light_coro():
      pass
  
  
  async def main():
      print('Before running task group!')
      time0 = time.time()
      # asyncio.get_event_loop().set_task_factory(asyncio.eager_task_factory)
      async with asyncio.TaskGroup() as tg:
          for _ in range(1000000):
              tg.create_task(light_coro())
      print(f'It took {time.time() - time0} to run!')
      print('After running task group with eager task factory!')
  
  asyncio.run(main())
  
  # 输出：
  # Before running task group!
  # It took 8.283295392990112 to run!
  # After running task group with eager task factory!  
  ```
  可以看到性能提升接近 7 倍。  

* `asyncio.create_eager_task_factory(custom_task_constructor)`
该函数用于创建一个任务工厂，该工厂能够使用自定义的任务构造函数。通常在默认的任务工厂之外，你可以使用这个方法来创建更多可控的任务。

**参数说明：**  
  * custom_task_constructor：这是一个自定义的任务构造函数，该构造函数通常会接受事件循环和协程作为输入，并返回一个任务对象；

**返回值：**  
  * 返回一个工厂函数，可以用于生成任务对象；

**示例：**
TODO： 代补充