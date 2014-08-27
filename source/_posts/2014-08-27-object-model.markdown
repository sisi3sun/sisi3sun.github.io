---
layout: post
title: "Object Model"
date: 2014-08-27 16:49:55 +0800
comments: true
categories: objective-c
---
# objective-c对象模型
这次研究objective-c的对象模型，之后会找一些C++和Java的资料进行研究。
关于对象模型，这一篇讲得很好。 [Object, Class and Meta Class in Objective-C](http://blog.eddie.com.tw/2013/12/05/object-class-and-meta-class-in-objective-c/)
对象模型的基础是object和class，下面看看oc中是如何实现的。 `object`定义如下：

    ///Represents an instance of a class. 
    struct objc_object {     
        Class isa OBJC_ISA_AVAILABILITY; 
    } ; 

`object`实际上只是一个结构体，仅包含一个`isa`的指针，指向自己所属的`class`。 而`class`定义如下：

    struct objc_class {
        Class isa  OBJC_ISA_AVAILABILITY;
    
    #if !__OBJC2__
    Class super_class                                     OBJC2_UNAVAILABLE;
    const char *name                                      OBJC2_UNAVAILABLE;
    long version                                          OBJC2_UNAVAILABLE;
    long info                                             OBJC2_UNAVAILABLE;
    long instance_size                                    OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                          OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                 OBJC2_UNAVAILABLE;
    struct objc_cache *cache                              OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                  OBJC2_UNAVAILABLE;
    #endif
    
    }

可以看出`class`也是一个结构体，isa指向了它所属的`class`。
## OC的消息发送
推荐的那篇文章中译为“讯息传递”。 事实上，调用任何方法，都是通过给对象 发送消息 来执行。 如下

    objective-c SpeakingBird *redBird = [[SpeakingBird alloc] init]; 
    [redBird saySomething:@"hello"]; 

第一行创建了一个`SpeakingBird`类型的`objectredBird`，第二行的意思是，通知`redBird`以`hello`字符串为参数，执行`saySomething`方法。方法执行时，将方法和参数以消息形式发送给`redBird`，它的`isa`指针指向了`Class SpeakingBird`，于是在`SpeakingBird`的`Methodist`中找到`saySomething:`方法，执行它。
## 为什么要存在meta-class
上面介绍，`class`也是一个结构体，`isa`指向了它所属的`class`，那`class`的class是什么呢？就是`meta-class`。 在声明方法的时候我们知道。

    - (void)saySomething:(NSString *)words;

 
这是一个对象方法，由object来调用。 还有一类方法

    + (SpeakingBird *)copyWithBird:(SpeakingBird *)bird;

 
这是一个类方法，调用方式为：

    SpeakingBird *redBird2 = [SpeakingBird copyWithBird:redBird]; 

这个`copyWithBird:`消息，是直接发送给`SpeakingBird`的，而不是发送给某一个`object`。那么必须要有一个结构来接受这个消息，这就是`meta-class`了。 下面是`meta-class`和`class\`的关系，本图来自文章开头提到的链接。
![Alt text](./1406183850034.png)

更深入一些的对象模型和runtime的文章http://blog.devtang.com/blog/2013/10/15/objective-c-object-model/