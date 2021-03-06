---
title: 深入理解Java虚拟机第四部分：程序编译与代码优化
date: {date}
tags: Java
categories: JVM
---
# 前端编译与优化

Java中即时编译器在运行期的优化过程对于程序运行来说更重要，而前端编译器在编译期的优化过程对于程序编码来说关系更加密切。

## Javac编译器

### 解析与填充符号表过程

1. 词法，语法分析

* 词法分析是将源代码的字符流转变为标记(Token)集合，单个字符是程序编写过程的最小元素，而标记则是编译过程中的最小元素

* 语法分析是根据Token序列构造抽象语法树的过程

  抽象语法树是一种用来描述程序代码语法结构的树形表示方式，语法树的每一个节点都代表着程序代码中的一个语法结构，例如包、类型、修饰符、运算符、接口、返回值甚至代码注释等都可以是一个语法结构	

* 经过这个步骤之后，编译器就基本不会再対源码文件进行操作了，后续的操作都基于抽象语法树	

2. 填充符号表

符号表是由一组符号地址和符号信息构成的表格，符号表中所登记的信息在编译的不同阶段都要用到。

在语义分析中，符号表所登记的内容将用于语义检查和产生中间代码。

在目标代码生成阶段，当对符号名进行地址分配时，符号表是地址分配的依据。

## 注解处理器

## 语义分析与字节码生成

语法树能够表示一个结构正确的源程序的抽象，但无法保证源程序是符合逻辑的。语义分析的主要任务是对结构上正确的源程序进行上下文有关性质的审查，如进行类型审查。

* 标注检查步骤检查的内容包括诸如变量使用前是否被声明、变量与赋值之间的数据类型是否匹配；标注检查中还有一个重要的动作称为常量折叠。
* 数据及控制流分析是对程序上下文逻辑更进一步的验证，它可以检查出诸如程序局部变量在使用前是否有赋值，方法的每条路径是否都有返回值，是否所有的受查异常都被正确处理了等问题
* 解语法糖：语法糖指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用
* 字节码生成阶段不仅仅是把前面各个步骤所生成的信息转化成字节码写到磁盘中，编译器还进行了少量的代码添加和转换工作

## Java语法糖的味道

### 泛型

泛型的本质是参数化类型（Parameterized Type）或者参数化多态（Parametric Polymorphism）的应用，即可以将操作的数据类型指定为方法签名中的一种特殊参数，这种参数类型能够用在类、接口和方法的创建中，分别构成泛型类、泛型接口和泛型方法。泛型让程序员能够针对泛化的数据类型编写相同的算法，这极大地增强了编程语言的类型系统及抽象能力。



C#里面的泛型无论在程序源码中、编译后的 Intermediate Language 中，或是运行期的CLR中，都是切实存在的，List(int)与List(String)就是两个不同的类型，它们在系统运行期生成，有自己的虚方法表和类型数据，这种实现称为类型膨胀，基于这种方法实现的泛型称为**真实泛型**。



Java语言中的泛型则不一样，它只在程序源码中存在，编译后的字节码文件中，就已经替换为原始类型了，并且在相应的地方插入了强制类型转换。因此对于运行期的Java语言来说，ArrayList(Integer)与ArrayList(String)就是同一个类，所以泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为**类型擦除**，基于这种方法实现的泛型称为伪泛型



擦除法所谓的擦除，仅仅是对方法的Code属性中的字节码进行擦除，实际上元数据中还是保留了泛型信息，这也是我们能通过反射手段取得参数化类型的根本依据。

### 自动装箱、拆箱与遍历循环

### 条件编译

Java语言中条件编译的实现，也是Java语言的一颗语法糖，根据布尔常量值的真假，编译器将会把分支中不成立的代码块消除掉

# 后端编译与优化

## 即时编译器

Java程序最初是通过解释器进行解释执行的，当虚拟机发现某个方法或代码块的运行特别频繁时，就会把这些代码认定为"热点代码"。为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成本地平台的机器码，并进行各种层次的优化，完成这个任务的编译器称为即时编译器JIT

### 编译对象与触发条件

* 热点代码：被多次调用的方法以及被多次执行的循环体（编译器都是编译整个方法）

* 热点探测判定方式：

1. 基于采样的热点探测：虚拟机周期性地检查各个线程的栈顶，如果发现某个方法经常出现在栈顶，那个这个方法就是“热点方法”。
   * 优点是实现简单、高效，还可以很容易的获取方法调用关系。
   * 缺点是不够精确，且容易受到线程阻塞或别的外界因素的印象而扰乱热点探测。
2. 基于计数器的热点探测：虚拟机为每个方法建立计数器，统计方法的执行次数，执行次数超过一定的阈值就认为它是热点方法
   * 实现麻烦，需要为每个方法建立并维护计数器，并且不能直接获取到方法的调用关系。
   * 统计结果精确严谨。
   * 默认方法调用计数器统计的并不是方法被调用的次数，而是一个相对的执行频率。当超过一定的时间限度，如果方法的调用次数仍不足以让它提交给JIT，那个这个方法的调用计数器就被减少一半，这个过程称为方法调用计数器热度的衰减，而这段时间就称为此方法统计的半衰周期。
   * 回边计数器，它的作用是统计一个方法中循环体代码执行的次数，建立回边计数器的目的就是为了出发OSR编译

## Java与C/C++的编译器对比

即时编译器运行占用的是用户程序的运行时间，所以不可以随便引入大规模的优化技术；静态编译的时间成本不是主要关注点。

Java语言是动态的类型安全语言，意味着虚拟机来确保程序不会违反语言语义或访问非结构化内存。动态检查会消耗不少的运行时间。

虚方法的频率远大于C/C++语言，方法接收者进行多态选择的频率高于C/C++语言。即时编译器的一些优化难度较高，例如方法内联。

Java语言是可动态扩展的语言，运行时加载新的类可能改变程序类型的继承关系，这使得很多全局优化难以进行，因为编译器无法看到程序的全貌，许多全局优化都只能以激进优化的方式来完成，便以其不得不时刻注意并随着类型的变化而在运行时撤销或者重新进行一些优化。

C/C++垃圾回收不存在无用对象筛选的过程，效率较高