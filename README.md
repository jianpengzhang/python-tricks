# Python-Tricks

✨ Python 技巧、面试题、代码片段

***

## 目录

### Python 语言特性
*** 
* [1. Python 函数参数传递](docs/python_features.md#1-python-函数参数传递)
* [2. Python 元类(metaclass)](docs/python_features.md#2-python-元类metaclass)
* [3. @staticmethod和@classmethod](docs/python_features.md#3-staticmethod和classmethod)
* [4. 类变量和实例变量](docs/python_features.md#4-类变量和实例变量)
* [5. Python 自省](docs/python_features.md#5-python-自省)
* [6. 列表、字典、集合推导式](docs/python_features.md#6-列表字典集合推导式)
* [7. Python 单下划线和双下划线](docs/python_features.md#7-python-单下划线和双下划线)
* [8. Python "\_\_str__" vs. \"_\_repr__"](docs/python_features.md#8-python-__str__-vs-__repr__)
* [9. 字符串格式化 & 拼接](docs/python_features.md#9-字符串格式化--拼接)
* [10. 可迭代对象、迭代器和生成器](docs/python_features.md#10-可迭代对象迭代器和生成器)
    * [10.1 容器 (container)](docs/python_features.md#101-容器-container)
    * [10.2 可迭代对象 (iterable)](docs/python_features.md#102-可迭代对象-iterable)
    * [10.3 迭代器 (iterator)](docs/python_features.md#103-迭代器-iterator)
    * [10.4 生成器 (generator)](docs/python_features.md#104-生成器-generator)
* [11. \*args and **kwargs](docs/python_features.md#11-args-and-kwargs)
    * [11.1 *args](docs/python_features.md#111-args)
    * [11.2 **kwargs](docs/python_features.md#112-kwargs)
    * [11.3 混合使用](docs/python_features.md#113-混合使用)
    * [11.4 函数参数顺序](docs/python_features.md#114-函数参数顺序)
    * [11.5 "*" 和 "**" 用法](docs/python_features.md#115--和--用法)
* [12 面向切面编程(AOP)和装饰器](docs/python_features.md#12-面向切面编程-aop-和装饰器)
    * [12.1 面向切面编程 (AOP)](docs/python_features.md#121-面向切面编程-aop)
    * [12.2 装饰器](docs/python_features.md#122-装饰器)
* [13 闭包 (closure)](docs/python_features.md#13-闭包-closure)
    * [13.1 函数作用域](docs/python_features.md#131-函数作用域)
    * [13.2 嵌套函数](docs/python_features.md#132-嵌套函数)
    * [13.3 闭包](docs/python_features.md#133-闭包)
* [14 函数是一等公民 (First-Class Citizens)](docs/python_features.md#14-函数是一等公民-first-class-citizens)
* [15 鸭子类型 (Duck typing)](docs/python_features.md#15-鸭子类型-duck-typing)
* [16 Python 中重载](docs/python_features.md#16-python-中重载)
* [17 新式类和旧式类](docs/python_features.md#17-新式类和旧式类)
* [18 __new__和__init__区别](docs/python_features.md#18-__new__和__init__区别)
* [19 设计模式](docs/python_features.md#19-设计模式)
    * [19.1 单例模式（Singleton Pattern）](docs/python_features.md#191-单例模式singleton-pattern)
        * [19.1.1 使用 \_\_new__ 方法](docs/python_features.md#1911-使用-__new__-方法)
        * [19.1.2 使用装饰器](docs/python_features.md#1912-使用装饰器)
        * [19.1.3 共享属性](docs/python_features.md#1913-共享属性)
        * [19.1.4 使用元类（MetaClass）](docs/python_features.md#1914-使用元类metaclass)
        * [19.1.5 使用模块（import方法）](docs/python_features.md#1915-使用模块import方法)
        * [19.1.6 线程安全单例](docs/python_features.md#1916-线程安全单例)
    * [19.2 工厂模式（Factory Pattern）](docs/python_features.md#192-工厂模式factory-pattern)
        * [19.2.1 简单工厂模式](docs/python_features.md#1921-简单工厂模式)
        * [19.2.2 工厂方法模式](docs/python_features.md#1922-工厂方法模式)
        * [19.2.3 抽象工厂模式](docs/python_features.md#1923-抽象工厂模式)
        * [19.2.4 单例模式与工厂模式结合](docs/python_features.md#1924-单例模式与工厂模式结合)
    * [19.3 观察者模式（Observer Pattern）](docs/python_features.md#193-观察者模式observer-pattern)
    * [19.4 责任链模式（Chain of Responsibility Pattern）](docs/python_features.md#194-责任链模式chain-of-responsibility-pattern) 
    * [19.5 策略模式（Strategy Pattern）](docs/python_features.md#195-策略模式strategy-pattern)
    * [19.6 命令模式（Command Pattern）](docs/python_features.md#196-命令模式command-pattern)
    * [19.7 适配器模式（Adapter Pattern）](docs/python_features.md#197-适配器模式adapter-pattern)
    * [19.8 模板方法模式（Template Method Pattern）](docs/python_features.md#198-模板方法模式template-method-pattern)
    * [19.9 组合模式（Composite Pattern）](docs/python_features.md#199-组合模式composite-pattern)
    * [19.10 外观模式（Facade Pattern）](docs/python_features.md#1910-外观模式facade-pattern)
    * [19.11 原型模式（Prototype Pattern）](docs/python_features.md#1911-原型模式prototype-pattern)
    * [19.12 状态模式 (State Pattern) ](docs/python_features.md#1912-状态模式state-pattern)
    * [19.x 更多设计模式](docs/python_features.md#19x-更多设计模式)
* [20 进程 (Process)、线程 (Thread)、协程 (Coroutine)](docs/python_features.md#20-进程process线程thread协程coroutine)
    * [20.1 进程 (Process)](docs/python_features.md#201-进程process)
        * [20.1.1 子进程 (Process)](docs/python_features.md#2011-子进程-process)
        * [20.1.2 进程池 (Pool)](docs/python_features.md#2012-进程池-pool)
        * [20.1.3 进程间通信（Inter-Process Communication, IPC）](docs/python_features.md#2013-进程间通信inter-process-communication-ipc)
          * [20.1.3.1 队列 (Queue)](docs/python_features.md#20131-队列-queue)
          * [20.1.3.2 管道 (Pipe)](docs/python_features.md#20132-管道-pipe))
          * [20.1.3.3 管理器（Manager）](docs/python_features.md#20133-管理器manager))
            * [20.1.3.3.1 multiprocessing.Manager()](docs/python_features.md#201331-multiprocessingmanager)
            * [20.1.3.3.2 自定义管理器](docs/python_features.md#201332-自定义管理器))
            * [20.1.3.3.3 使用远程管理器](docs/python_features.md#201333-使用远程管理器))
          * [20.1.3.4 共享 ctypes 对象](docs/python_features.md#20134-共享-ctypes-对象)
          * [20.1.3.5 同步原语](docs/python_features.md#20135-同步原语)
          * [20.1.3.6 跨进程直接访问内存共享](docs/python_features.md#20135-跨进程直接访问内存共享)
    * [20.2 线程 (Thread)](docs/python_features.md#202-线程-thread)
        * [20.2.1 threading.Thread](docs/python_features.md#2021-threadingthread)
        * [20.2.2 线程池](docs/python_features.md#2022-线程池)
        * [20.2.3 GIL（Global Interpreter Lock，全局解释器锁）](docs/python_features.md#2023-gilglobal-interpreter-lock全局解释器锁)
        * [20.2.4 线程同步](docs/python_features.md#2024-线程同步)
          * [20.2.4.1 Lock (锁)](docs/python_features.md#20241-lock-锁)
          * [20.2.4.2 RLock (递归锁)](docs/python_features.md#20242-rlock-递归锁)
          * [20.2.4.3 Condition (条件对象)](docs/python_features.md#20243-condition-条件对象)
          * [20.2.4.4 Semaphore (信号量对象)](docs/python_features.md#20244-semaphore-信号量对象)
          * [20.2.4.5 Event (事件对象)](docs/python_features.md#20245-event-事件对象)
          * [20.2.4.6 Timer (定时器对象)](docs/python_features.md#20246-timer-定时器对象) 
          * [20.2.4.7 Barrier (栅栏对象)](docs/python_features.md#20247-barrier-栅栏对象)
    * [20.3 协程 (Coroutine)](docs/python_features.md#203-协程-coroutine)
    * [20.4 并行 & 并发](docs/python_features.md#204-并行--并发)
    * [20.5 进程、线程、协程对比](docs/python_features.md#205-进程线程协程对比)
* [21 asyncio 异步 I/O](docs/python_features.md#21-asyncio-异步-io)
  * [21.1 运行 asyncio 程序](docs/python_features.md#211-运行-asyncio-程序)
  * [21.2 协程与任务](docs/python_features.md#212-协程与任务)
  * [21.3 asyncio streams（流）](docs/python_features.md#213-asyncio-streams流)
  * [21.4 同步原语](docs/python_features.md#214-同步原语)
  * [21.5 子进程](docs/python_features.md#215-子进程)