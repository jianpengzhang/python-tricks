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

在 Python 中，函数参数传递涉及到 “可变类型” 和 “不可变类型” 两个概念，理解这两个概念对理解参数传递过程中的 “值变与不变” 非常重要。

**（1）不可变类型 vs 可变类型:**

* 不可变类型: 包括 int, float, str, tuple 等，这些类型的对象一旦创建，其值就不能被修改；
* 可变类型: 包括 list, dict, set 等，这些类型的对象创建后，其内容可以被修改；

**（2）参数传递机制：**

Python 中函数参数传递本质上是 “传对象引用”，这意味着在函数内部，参数名和传递的对象指向同一个内存地址。

* 不可变类型的参数传递：  
当将 "不可变类型" 作为参数传递给函数时，函数内部对参数的修改不会影响外部的变量。这是因为修改 "不可变类型" 的值会创建一个新的对象，而不会修改原来的对象。

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

#### 2.1 元类（Metaclass）简介

在 Python 中，一切皆为对象，包括类本身，而元类（Metaclass）是用于创建类的 “类”，即元类定义了类的行为，而类定义了实例的行为，换句话说，元类控制类的创建过程，默认情况下，Python 中所有类的元类是 "type"，它负责创建所有的类。

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

这里 "MyMeta" 是一个元类，它重载了 `__new__` 方法，在创建类时会输出一条信息，“MyClass” 使用这个元类，因此在它被创建时，“MyMeta” 的 `__new__` 方法被调用。

**元类的常见方法：**

* `__new__(cls, name, bases, dct)`: 在类创建之前调用，用于控制类的创建过程，“name” 是类的名称，“bases” 是类的基类，“dct” 是类的属性和方法字典。
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

#### 2.2 元类面试题

* 面试题 1: 解释 Python 中的元类，元类的作用是什么？  
  **解答**：  
  元类是用于创建类的 “类”，控制类的创建过程。元类可以定制类的创建规则，如自动添加方法、属性校验、修改继承关系等。Python 中的所有类默认由 “type” 这个元类创建。元类的典型用法包括实现单例模式、注册类、自动化创建方法等。

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
  **解释**：在 "AutoMethodMeta" 中，通过 `__new__` 方法向类添加了一个名为 "auto_method"、“auto_method_print_a” 方法，因此每个使用该元类的类都会自动拥有这两个方法。

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

#### 3.1 实例方法

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

#### 3.2 类方法 (classmethod)

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

#### 3.3 静态方法 (staticmethod)

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

#### 3.4 区别总结

| \\                 | 实例方法                                      | 类方法                                                               | 静态方法                                                             |
|:-------------------|:------------------------------------------|:------------------------------------------------------------------|:-----------------------------------------------------------------|
| 区别                 | - 第一个参数是 self，表示实例本身；<br/> - 可以访问实例属性和方法； | - 使用 @classmethod 装饰，第一个参数是 cls，表示类本身；<br/> - 可以访问类属性，不能直接访问实例属性； | - 使用 @staticmethod 装饰，没有默认参数；<br/> - 不能访问类属性或实例属性，仅与类有逻辑关系的独立方法； |
| 调用形式：obj=MyClass() | obj.instance_method()                     | obj.class_method()                                                | obj.static_method()                                              |
| 调用形式：MyClass | 不可用                                       | MyClass.class_method()                                            | MyClass.static_method()                                          |
| 使用场景               | 用于需要访问或修改实例状态的操作；                         | 用于需要访问类属性或执行一些与类有关的操作，而不需要实例化类的场景；                                | 用于独立于实例或类状态，但与类逻辑相关的操作；                                          |

## 4. 类变量和实例变量

#### 4.1 类变量（Class Variables）

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

#### 4.2 实例变量（Instance Variables）

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

#### 4.3 类变量与实例变量的区别

* 作用域：类变量属于类，实例变量属于实例；
* 共享性：类变量在所有实例之间共享，实例变量是独立的；
* 访问方式：类变量可以通过类名或实例名访问，但实例变量只能通过实例名访问；
* 生命周期：类变量的生命周期与类相同，实例变量的生命周期与实例相同；

#### 4.4 注意事项（避免混淆）

在 Python 中，当类变量是一个可变对象（如列表、字典、集合等）时，通过实例修改类变量的行为会与不可变对象（如整数、字符串、元组等）有所不同。这是由于 Python 中对象的可变性和引用机制所导致的（如第一章节）。

#### 4.4.1 类变量 —— 不可变对象

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

在这个例子中，obj.class_variable = "Modified by instance" 实际上在 obj 的实例命名空间中创建了一个新的实例变量 class_variable，而类变量 MyClass.class_variable 并未受到影响。

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

因此，在使用类变量作为可变对象时，需要小心其共享特性，以避免意外修改影响所有实例。如果希望每个实例都有自己的独立副本，通常应该在 `__init__` 方法中初始化该变量为实例变量，而不是使用类变量。同时，建议通过类名访问和修改类变量避免混淆。

## 5. Python 自省

Python 自省（Introspection）是指在运行时检查对象的能力。通过自省，Python 程序可以在运行时动态获取对象的类型、属性、方法、父类、模块等信息。自省机制使 Python 具有强大的动态性，能够根据当前对象的状态进行相应的操作。

#### 5.1 自省主要功能

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

#### 5.2 自省应用场景

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

#### 5.3 面试示例
* 问题 1：什么是 Python 自省？请举例说明：
  答：自省是指在运行时检查对象的能力。示例：可以是使用 type() 获取对象类型，使用 dir() 列出对象的属性和方法。

* 问题 2：如何在 Python 中动态获取对象的所有属性？
  答：可以使用 dir() 函数列出对象的所有属性和方法，也可以使用 vars() 获取对象的属性字典。

* 问题 3：编写代码，动态导入模块并调用其中的函数。
  答：使用 `__import__()` 动态导入模块，并调用其中的函数，如 module.sqrt(16)。

## 6. 列表、字典、集合推导式

推导式是 Python 中非常强大的语法工具，允许我们以简洁的方式生成和操作列表、字典以及集合。

#### 6.1 列表推导式（List Comprehension）

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
squares = [x**2 for x in range(10)]
print(squares)  # 输出: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 过滤出偶数的平方
even_squares = [x**2 for x in range(10) if x % 2 == 0]
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

#### 6.2 字典推导式（Dict Comprehension）

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
square_dict = {x: x**2 for x in range(10)}
print(square_dict)  # 输出: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81}

# 过滤出偶数的平方
even_square_dict = {x: x**2 for x in range(10) if x % 2 == 0}
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

#### 6.3 集合推导式（Set Comprehension）

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

#### 6.4 常见面试题

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

#### 7.1  单下划线（“_”）使用

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
变量 “_protected_var” 和方法 “_protected_method” 都以单下划线开头，这是一种约定，告诉其他开发人员这些成员是类内部使用的，不建议在类外部直接访问，公共方法 “public_method” 可以访问内部方法 “_protected_method”。

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

#### 7.2 双下划线（“__”）使用

#### 7.2.1 前缀 "__var" (名称改写机制)

双下划线作为变量名或方法名前缀，表示该变量或方法是类内的私有成员（Private）。Python 会将其名称进行 “改写”（name mangling），将其改为 "_ClassName__var" 的形式，以避免与子类中的名称冲突。

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

双下划线前后缀用于定义 Python 的特殊方法或魔术方法（Magic Methods），这些方法通常由 Python 解释器调用，而不是用户直接调用，一般不应为普通变量或方法随意命名为这种形式。  

例如：  
* `__init__`：是构造函数；
* `__str__`：通过 str(object) 以及内置函数 format() 和 print() 调用以生成一个对象的 “非正式” 或格式良好的字符串表示。返回值必须是字符串对象。
* `__repr__`：是由 repr() 内置函数调用，用来输出一个对象的“官方”字符串表示。返回值必须是字符串对象，此方法通常被用于调试。内置类型 object 所定义的默认实现会调用 object.__repr__()。

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"MyClass with value {self.value}"


obj = MyClass(10)
print(obj)  # 输出: MyClass with value 10
```

#### 7.3 面试示例

* 问题 1：_var 和 __var 有什么区别？  
  `_var`：表示受保护的变量，这是约定俗成的表示，意味着该变量不应被外部直接访问，但实际上可以访问；  
  `__var`：会触发名称改写机制，将其改为 _ClassName__var，从而避免名称冲突，并使其成为类的私有变量；  


* 问题 2：如何在 Python 中定义一个类的私有方法，并确保它不会与子类中的方法冲突？  
  使用双下划线前缀（如 __private_method）来定义类的私有方法，Python 会自动将其名称改写为 _ClassName__private_method，从而避免与子类中的方法冲突；


* 问题 3：\__init\__ 和 \__str\__ 的作用是什么？  
  `__init__`：是类的构造函数，用于初始化对象；  
  `__str__`：是用于定义对象的字符串表示形式，当使用 print 或 str() 调用对象时会被调用；

## 8. Python \"_\_str__" vs. \"_\_repr__"

在 Python 中，`__str__` 和 `__repr__` 是两个用于对象字符串表示的特殊方法（魔术方法），它们虽然都用于返回对象的字符串表示，但在语义和用途上有所不同。

#### 8.1 \"_\_str__"

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

#### 8.2 \"_\_repr__"

由 repr() 内置函数调用，用来输出一个对象的 “官方” 字符串表示，返回值必须是字符串对象，此方法通常被用于调试，内置类型 object 所定义的默认实现会调用 `object.__repr__()`。

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

#### 8.3 "\_\_str__" vs. "\_\_repr__"

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

#### 9.1 使用 "%" 操作符

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

#### 9.2 使用 "str.format()" 方法

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

#### 9.3 使用 "f-strings" (Python 3.6+)

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

#### 9.4 使用 "string.Template"

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

#### 9.5 其他简单字符串拼接

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