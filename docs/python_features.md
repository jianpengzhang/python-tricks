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