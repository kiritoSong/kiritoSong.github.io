---
layout: post
title: Runtime的初步认识
category: 学习
tags: runtime
keywords: Objective-C, runtime
---

#Runtime的初步认识

[TOC]

##Runtime介绍

学习一个东西至少要先知道它是个啥，你一定听说过“运行时是 Objective-C 的一个特色”，这里的“运行时”就是指 runtime 了。

runtime是在自 iOS 平台开放并基于 Objective-C 语言开发后的一个编程语言上的高级技术。

学习runtime的目的并不是为了开发，而是让你更好的理解 Objective-C 的工作原理，从而熟练的运用 Objective-C 来开发应用。当然，你也可以利用 runtime 的理论在面试的时候给自己加分。

Runtime 很强大，很强大，很强大，强大的让你无言以对。甚至因为 Runtime，让 Objective-C 与别的面向对象的语言有着极大的不同。

曾经在论坛上看到这样的一段话“**如果这世界上没有 OC ，一个会熟练使用 Runtime 的人就可以写出一个 OC。**”其实苹果就是利用runtime制造了现在的Objective-C 语言。runtime只是给C语言插上了面向对象的翅膀，然后他就可以上天与各路面向对象语言一起飞了︿(￣︶￣)︿。

##类与结构体的关系

任何一个可执行的编程语言，最终编译后都会变成汇编，然而 OC 到汇编并不是一步的，其中还经历了 C，甚至还有可能是 C++。

但是 C 语言是最接近汇编语言的，因为 C 语言有直接的内存操作。而 C 语言又是用英语写，人易看懂的语言。OC 在变成汇编前，它先变成了 C，然后才是汇编。

OC 是一个面向对象的语言，C 语言是面向过程的。OC 和 C 最大的区别就是 C 没有 “类”。那么，OC 变成 C 语言后，面向对象特有的类变成了什么，**类是怎么存在面向过程语言中的？** 到这个问题的时候，runtime的作用就展现出来了。

Runtime用面向过程语言(C) 中的结构体(struct) 做出了类(class) 在面向对象语言(OC) 中的实现。其实我们可以看见这个Class在C中变成结构体之后是什么样子的。

我们可以在文件中导入头文件
```
#import <objc/runtime.h>
```
然后利用`command`键和`鼠标左键`进入 runtime 的头文件定义，你会发现一个如下的结构体
```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```
这个结构体的名字叫`objc_class`，其实他就是我们平时在OC中常用的`Class`。到这里我们又会疑问它们俩是怎么发生关系的呢。

继续导入头文件
```
#import <objc/objc.h>
```
点进去就会看到这几行
```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
如果你有 C 语言基础，你一定知道`typedef`的语法以及含义

苹果重新定义`objc_class`这个结构体，并取名为`*Class`，也就是说`Class`是一个 `objc_class`的一个指针。同样`id`也是`objc_object`的一个指针。

非常的显然了，平时我们这样编写代码：

```
id obj = [[NSObject alloc] init];
```

`obj`是一个`NSObject`的一个对象，而本质上`obj`是一个`id`，而`id`又是一个`objc_object`结构体的一个指针，所以你其实得到的只是一个**结构体的指针**而已。

那么同样，类其实也是有对象的，因为你可以这样打：

```
Class c = [obj class];
```

这样 c 就是 objc 的类型了，然后你可以直接通过这个 c 来创建对象：

```
id other_obj = [[c alloc] init];
```

如果你细心的话，你会发现 

`Class` 保存的一个类对象，但本质是`objc_class`的指针

`id` 保存一个对象，但本质是`objc_object`的指针

而我们所做的所有面向对象编程，对对象的操作，最终都会落实到这些结构体上。这些结构体保存着一个对象的所有信息。

##结构体解析

`objc/objc.h`和`objc/runtime.h`两个头文件的内容，让大家好看一些：

```
typedef struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} *Class OBJC2_UNAVAILABLE;
//以上是一个类的结构体

typedef struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
} *id ;
//以上是一个对象的结构体

```
我们从对象结构体开始介绍(objc_object)。

当我们获取到一个对象的时候，说白了就是一个对象的引用，也就是一个`id`。上面那个结构体被重定义为`*id`，那么`id`就是一个`objc_object`的指针。

这个结构体里有一个成员，是类型为`Class`的一个`isa`。我们可以这么理解，我们实例化出来的对象，它实际上只保存一个`类的结构体`的指针，也就为了说明这个对象是由什么类实例化出来的。

现在来看看类的结构体(objc_class)。

对象被实例化出来后，他就保存了那个创造自己的类的信息，这个信息也就是一个`objc_class`结构体，那么我们来看看，这个结构体到底保存了那些关于一个类的信息

* `Class isa` 
	* 保存一个`objc_class`的一个指针，叫做`isa`
	* 这个`isa`和`objc_object`中的重名，具体内容和区别之后分析
* `Class super_class`
	* 保存一个`objc_class`的一个指针，叫做`super_class`
	* 显而易见，这里就是一个指向父类的指针，为了保存自己是从哪里集成来的
* `char *name`
	* 保存这个类名的字符串指针
	* 这个也不用多解释，NSObject类里这里的值就是"NSObject"
* `long version`
* `long info`
* `long instance_size`
	* 以上三个成员不多解释，用不到
* `struct objc_ivar_list *ivars` 
	* 从定义可以看出，是一个结构体指针，保存着这个类的实例变量列表
* `struct objc_method_list **methodLists`
	* 从定义可以看出，是一个结构体指针，保存着这个类的方法列表
* `struct objc_cache *cache` 
	* 从定义可以看出，是一个结构体指针，保存缓存
	* 什么东西可以保存到缓存，这个下文会解释
* `struct objc_protocol_list *protocols`
	* 从定义可以看出，是一个结构体指针，保存协议列表

我们写的类，实际上也就是这些东西，一些实例变量、一些方法、实现一些协议，还给类取了个名字，拥有父类的所有特性。

现在我们终于知道我们写的类最终会变成了一个什么样的结构体。

## 结构体的作用

结构体，就是一块内存，用于统一保存大小不同且不变的集合。

**在 C 语言中，允许我们去创建与修改结构体**，也就是动态的去修改OC中的类。

我们可以动态的修改这些结构体内的内容，甚至你在一个对象创建后，你可以真真切切地修改他的类型和父类。你甚至可以为一个类动态的增加方法，而不是在 .m 文件中去写方法。同样，你甚至可以去**修改苹果写的类**。这也是runtime如此强大的地方。先介绍到这里。近期会再更新篇关于runtime应用的文章。