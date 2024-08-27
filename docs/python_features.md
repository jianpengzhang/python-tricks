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
   #foo是外围函数 
   a = 1 
   # printer是嵌套函数 
   def printer(): 
       print(a)
   printer() 
foo() # 1
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

