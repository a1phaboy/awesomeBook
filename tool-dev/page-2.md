---
description: Taint Analysis Technology
---

# 污点分析技术

污点分析技术是信息流分析技术的一种实践方式，污点分析技术将系统或应用程序中的数据标记为污点或非污点，当污点通过信息流随数据传播到指定程序点或信息泄漏点时，将违反信息流策略。

### 污点分析模型

污点分析模型包括污点源，污点汇聚点和无害化处理。可以简单抽象成一个三元组`<sources,sinks,sanitizers>`的形式。

#### 污点源 sources

代表直接引入不受信任的数据或者机密数据到系统中

#### 污点汇聚点 sinks

污点 汇聚点表示系统将污点数据输出到敏感数据区或者外界，造成敏感数据区被非法改写或者隐私数据泄露

#### 无害化处理 sanitizers

通过数据加密、重新赋值或是过滤转义等操作使数据传播不再对系统的完整性和保密性产生危害

### 依赖关系

#### 数据依赖

数据依赖主要包括程序中各变量间的直接赋值、数学计算、函数调用等操作，在分析数据依赖时通常需要借助函数调用关系图来根据具体的函数特性进行函数内或函数间的污点传播分析。

#### 控制依赖

控制依赖主要包括程序中各变量间的条件判断与指令跳转等情况，在分析控制依赖时通常需要恢复控制流图，难点在于间接跳转。

### 污点传播分析

#### **显式流分析**

污点传播分析中的显式流分析就是分析污点标记如何随程序中变量之间的**数据依赖**关系传播。

#### **隐式流分析**

污点传播分析中的隐式流分析是分析污点标记如何随程序中变量之间的**控制依赖**关系传播,也就是分析污点标记如何从条件指令传播到其所控制的语句。

#### 过污染

过污染是指在污染分析过程中将与污点源没有数据和控制依赖关系的数据变量标记为污点变量，即产生误报

#### 欠污染

在污染分析过程中将与污点源存在数据或控制依赖关系的程 序变量标记为非污点变量，即产生漏报

### 🌰

```python
def func1():
    var1 = input("input data:")
    var3 = 10
    var4 = var1 * var3
    var7 = int(var4)
    return var7

def func2():
    var2 = input("input data:")
    var5 = 0
    if(var2 == 5):
        var5 = 3
    var6 = var2 * var5
    return var6

print(func1())
print(func2())
```

在func1中，污点源为var1。var4，var7为污点变量，污点汇聚点为 `return var7` ，var7并未进行无害化处理。

在func2中，污点源为var2。var5，var6为污点变量，其中var5由于控制依赖被var2隐式传播为污点变量，在var2不等于5的情况下，var5的值为0，任何数乘以0都为零，可以将var5抽象为无害化处理，因此在var2等于5的情况下func2满足信息流策略。



> 根据程序是否需要运行又可分为静态分析和动态分析

### 静态污点分析

静态污点分析主要通过词法和语法分析等方法离线分析变量间数据和控制依赖关系，以检测污点数据能否从污点源传播到污点汇聚点，在此过程中既不运行目标程序，也无需修改代码。静态污点分析的对象是程序代码或中间表示。可以将对污点传播中显式流的静态分析问题转化为对程序中静态数据依赖的分析：

* 首先,根据程序中的函数调用关系构建调用图(call graph,简称CG)
* 然后,在函数内或者函数间根据不同的程序特性进行具体的数据流传播分析.常见的显式流污点传播方式包括直接赋值传播、通过函数(过程)调用传播以及通过别名(指针)传播



### 动态污点分析

动态污点分析是指在程序运行过程中,通过实时监控程序的污点数据在系统程序中的传播来检测数据能否从污点源传播到污点汇聚点

* 基于硬件

基于硬件的污点传播分析需要定制的硬件支持,一般需要在原有体系结构上为寄存器或者内存扩展一个标记位,用来存储污点标记

* 基于软件

基于软件的污点传播分析通过插桩、重写等方式来修改程序的二进制代码来进行污点标记位的存储与传播
