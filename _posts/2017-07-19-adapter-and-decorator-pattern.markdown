---
layout: post
title:  "适配器VS装饰器模式"
date:   2017-07-19 13:26:48 +0800
---

本文目的是比较适配器与装饰器.

适配器模式:

    在设计模式中，适配器模式（英语：adapter pattern）有时候也称包装样式或者包装(wrapper)。
    将一个类的接口转接成用户所期待的。一个适配使得因接口不兼容而不能在一起工作的类能在一起工作，
    做法是将类自己的接口包裹在一个已存在的类中。

装饰器模式:

    修饰模式，是面向对象编程领域中，一种动态地往一个类中添加新的行为的设计模式。就功能而言，
    修饰模式相比生成子类更为灵活，这样可以给某个对象而不是整个类添加一些功能。
    
以上是维基百科对这三种模式的定义,现在想想,可能是因为我过早的对定义做了归纳.
因为他们都是一个类在无法接入(适配器)或不具备某功能(装饰器)或不想直接接入(代理)的情况下,
引入一个中间层或者说被包装成另一个类才能完成任务.问题肯定不会这么简单,所以要继续往下看.

![image](https://github.com/tingplay/tingplay.github.io/blob/master/pic/adapter.png?raw=true "适配器模式")

图1.适配器结构

![image](https://github.com/tingplay/tingplay.github.io/blob/master/pic/decorator.png?raw=true "装饰器模式")

图2.装饰器结构

从 UML 结构图上能很明显看出三种模式的不同.但是结构图只说出了 "how" 并不能说出 "why"(这算不算 UML 的缺陷).
要知道为什么,还是要深入去看信息流的传递,方法的调用有什么不一样,因为在 UML 中屏蔽了这些细节.而且从另一个层面也说明了,
他们虽然都是一层封装,但所处的情景,发挥的功效是完全不同的.

## IO 中的适配器与装饰器

在上面的结构图中适配器模式与装饰器模式是很好区分的.然后在读了<<深入分析Java Web技术内幕>>中的关于 I/O 设计模式的总结后,反而使我迷惑了,
这也是我当时无法区分适配器与装饰器的原因,实际的应用场景比列子更复杂,所以我就拿书中的实际例子出来具体分析.

![image](https://github.com/tingplay/tingplay.github.io/blob/master/pic/adapter_in_IO.png?raw=true "实际场景中的适配器")

图4.IO 中的适配器

![image](https://github.com/tingplay/tingplay.github.io/blob/master/pic/decorator_in_IO.png?raw=true "实际场景中的装饰器")

图5.IO 中的装饰器

在不知道对应的设计模式的时候,其实也挺懵逼的,现在提前知道了模式,可以尝试去一一对应.
装饰器还好说,可以与原来的结构一一对应.

1.component--InputStream,抽象组件角色

2.ConcreteComponent--FileInputStream,抽象组件现有实现类

3.Decorator--FilterInputStream,装饰者

4.ConcreteDecorator--BufferedInputStream,装饰器实现

但是适配器就没那么明显了,首先是结构不一样.即使在已知 InputStreamReader 是适配器的前提下,
我也无法在现有的结构上把他归纳为适配器模式.首先要降低要求,适配器不一定需要实现源接口,也可以持有源接口对象.

列1.适配器使用

    File filename = new File(pathname);
    InputStreamReader reader = new InputStreamReader(new FileInputStream(filename));
    BufferedReader br = new BufferedReader(reader);

这是一般的 InputStreamReader 用法.接收 InputStream 转化成 Reader.从代码上看 InputStreamReader 确实起到适配作用.
在这里看起来好像没有 StreamDecoder 什么事情.但其实它才是处理 InputStream 的主角.并且在读取的时候 InputStreamReader.read() 实际
也是调用 StreamDecoder.read().所以在图4中是有一个 StreamDecoder 结构的.不过这里有一个回答不了的问题"既然 StreamDecoder 才是真正的处理者,为什么他不是适配器呢?"
也许他确实是适配器,而 InputStreamReader 也是适配器,只不过适配是从 InputStreamReader 开始发生的.

再来看看 IO 中的装饰器.虽然确实知道了这就是装饰器模式,但为什么他是这样的呢?

列2.装饰器使用

    File filename = new File(pathname);
    BufferedInputStream bf = new BufferedInputStream(new FileInputStream(bFile));
    
这段代码和上面那段适配器的很像有没有.更为重要的不同是作为装饰器,这两行代码的功能已经完成了, BufferedInputStream 作为装饰器已经为 FileInputStream 增加了自己的功能.
装饰器还有一点值得注意,就是重叠装饰.

列3.装饰器多次装饰
    
    DataInputStream dos = new DataInputStream(new BufferedInputStream(new FileInputStream("temp.tmp")));
                                
我简写了代码,BufferedInputStream 装饰了 FileInputStream ,DataInputStream 又装饰了 BufferedInputStream 了.
这里重叠装饰增加了他的功能,缓存的功能,格式化的功能.重叠装饰在<Head First 设计模式中>有另一个列子的展示,计算饮料的价格.在饮料不停增加"佐料"(装饰器)的时候,他的价格也是累加的.这个使得重叠的效果更加直观.
价格累加能直观的表现出重叠效果的原因在于,你在拿到饮料之前并不能区分出一杯纯咖啡或者咖啡加牛奶加糖的区别,因为你只下了单,但是你付的钱增加了.这种附带的效应给了我深刻的映像.

这里其实我已经产生了一种错觉,适配器也可以看做装饰器模式. InputStreamReader 装饰了 FileInputStream 使他具备了处理字符流的功能.
这里需要将上文的一些内容做梳理了.先下定论列1中 InputStreamReader 只能是适配器.

* 适配器的可以实现源接口,也可以持有源接口
* 装饰器为了能让装饰一直持续,必须实现统一的接口

基于这两点实际分析列1.假设 InputStreamReader 是装饰器,那么他必须具备与被装饰者相同的接口,但是 FileInputStream 继承了 InputStream,而 InputStreamReader 并没有继承 InputStream.
另外, InputStreamReader 的装饰也没有重叠效应.既然他不是 InputStream 的实现类,自然也不能被其他 InputStream 的实现类装饰.
而且即使有一另个列子适配器将源接口和目标接口都实现了,也仍然不一定能满足重叠装饰的要求.因为实现源接口并不是必须的,适配器只要能达到接入源接口的目的就可以了,接入的方式有多种,不是非实现不可的.
所以虽然他们在列1,列2的行为很相似,但是他们的目的是不一样的,内部的接口也不同.在<<深入分析Java Web技术内幕>>最后也做了言简意赅的总结.

参考:
<<深入分析Java Web技术内幕>>,<<Head First设计模式>>