---
title: "现代C++学习指南-方向篇"
description: "现代C++学习指南-方向篇"
isCJKLanguage: false

lastmod: 2022-06-25T08:51:01+08:00
publishDate: 2022-06-25T08:51:01+08:00

author: hongui

categories:
 - C++
tags:
 - 学习指南
 - C++

toc: true
draft: false
url: post/现代C++学习指南-方向篇.html
---

C++是一门有着四十年历史的语言，先后经历过四次版本大升级（诞生、98、11、17（20），14算小升级）。每次升级都是很多问题和解决方案的取舍。了解这些历史，能更好地帮助我们理清语言的发展脉络。所以接下来我将借它的发展历程，谈一谈我对它的理解，最后给出我认为比较合理的学习路线指南。

<!--more-->

### C++0——诞生
C++诞生的目的是为了解决两个主要问题——性能和抽象。性能指的是拥有像C一样的底层访问能力和执行效率，抽象则意在语言层面提供对问题的描述能力和思考方法。这是C++的立命之本，也是C++经久不衰的原因。对于这两个目标，Bjarne Stroustrup想到的解决方法是充分利用现有的C的技术和工具，然后提供类来解决抽象问题。基于这个前提，我们就可以看出类是C++学习路上的第一个关卡。

C++认为类是一种抽象思维，类的相关特性都是为抽象提供服务的。所以C++中的类比其他面向对象的类提供了更多的能力，所以也具有更多的复杂性。为了描述这种复杂性，就不得不提到C++的两个特点，静态类型安全，资源管理。

静态类型安全可以帮助开发者定义出更合理合法的自定义类，如通过操作符重载，自定义类可以写出和基本类型一样的简洁代码。可以通过构造函数避免隐式类型转换而造成的运行时错误，也可以通过明确阻止某些操作阻止自己的类被滥用。所有的自主权都由开发者决定。所以假如我们是库的使用者，完全可以不用关心这些细节，我们只需要按照一般的语言一样写代码，遇到不合理的，编译器会直接告诉我们，不用担心这些问题会隐匿在程序运行时的某个时刻。

资源管理则可以帮助开发者提供资源管理的指导和支撑。资源有很多种，而在计算机中的资源大部分都是有限的，必须有借有还，而且借和还必须一一对应，不然就是内存泄漏。在C时代，资源管理靠的是开发者对资源的全局掌控力，语言层面没有提供更好的支持。为了更好地支持资源管理，C++提出了构造函数和析构函数，两者分别可以对应资源的获取和回收。但是很多时候资源不仅仅供自己使用，还需要提供给外部使用。为了配合这种资源的转移，C++又提供了移动和复制两种操作来支持。

综上，总结一下，C++的类提供了很多特性，但是不是所有的特性都是开发者需要的。```开发者在定义类的时候需要考虑的主要问题是，对这个类提供哪些支持，然后再在这些提供的功能中选择合适的语法特性来实现。```构造函数和析构函数可以提供很好的一一对应的操作，移动和复制则提供了资源在对象中怎么共享，操作符重载则可以让类使用更加简洁和优雅。


### C++98——标准化
C++98最大的升级是模板和异常，并且搭配了好用的标准库。

模板在C++中的地位怎么强调都不为过。它属于另一种抽象机制。所以它解决的也是抽象问题。C++中的类解决的是相似概念的抽象，更注重概念间的相似性。而模板解决的是通用问题的抽象，更注重概念的通用性。两者共同构成了C++的两大抽象基石。前面已经谈过了类，这里我们着重说一下模板。

得益于C++强大的静态类型安全，模板编写起来也很简单，普通的函数怎么写，它就可以怎么写，无非就是把特定类型换成泛型。但是，另一方面，模板还可以做得更多。模板可以支持多种参数，多个参数，限定参数，并且是类型安全的。更厉害的是，它还可以指定值。合理地配合使用类型和值，基本上就能解决大部分问题了。

说起异常。对于普通开发者没有多大吸引力。因为异常主要解决的问题是怎样告诉调用者发生错误了，是什么错误，并将执行能力转移到调用者一方。而我们大部分时间开发的都是业务代码，我们知道发生了什么，该怎样解决，大部分情况下是不太需要异常的。当然，并非说异常一无是处，异常对库开发者来说异常重要。对于库开发者来说，他需要在异常发生后，告诉调用者发生了错误，操作没有办法顺利执行。但是很多时候，库开发者并不知道调用者该怎样处理这个错误，是忽略呢，还是清理现场。异常机制提供了抛异常和异常捕获两种方式来支持库开发者和使用者。

对于新手来说，可能不太喜欢标准库，而倾向于自己写。这不是个好主意。标准库是经过工业级测试的代码，可以在绝大部分情况下正常工作，而自己手写虽然成就感更好，但是更可能携带BUG。早期的标准库提供的功能有限，只有`string`，输入输出流，位运算，三大容器，和一些小算法。不过，这些都足够我们日常使用了，尤其是现在标准库功能越来越完善了，大部分编程场景都能找到合适的工具来完成，完全可以放弃手写特定代码了。

C++98更多着眼于标准化，模板是一种标准，标准库也是一种标准。自此，```C++的三座大山算是构筑完成了，类，模板，标准库。```每一项都为C++带来了无限可能和旺盛生命力。

### C++11——全新语言
C++11的改动是革命性的，但是还保留着难以置信的兼容性，是非常不容易的。这里我们不细谈具体的特性和细节，只从大方向上来个笼统的概述。

首先直观的变化是在类型系统上，C++11将类型系统做了尽可能的规范化和统一化。
- 通过同意初始化规范了对象的初始化形式；
- 通过`auto`简化了类型声明的形式；
- 通过`nullptr`规范化了空指针的形式；
- 通过`enum class`提供了静态类型安全的枚举；
- 通过别名简化了类型书写的方式；
- 还有其他更多更多

类型系统的改进意味着开发者可以写出更简洁，更规范，也更安全的代码，但是对编译器的挑战却是巨大的，所以，很长时间内，C++11都没有得到很好的支持，同时也妨碍了C++的发展。

除了类型系统，另一项大改进就是提供了对线程的支持。C++11的标准库中提供了线程，条件对象，锁等线程相关的工具，这对库开发者来说是革命性的。在几乎不损失性能的情况下，提供了跨平台的线程支持，这极大地提高了库的稳定性和性能，也节省了很多平台测试时间，不得不说是顶呱呱。

另一个重要升级就是资源管理了。标准库提供了`unique_ptr`，`shared_ptr`来协助资源管理。同时为了更出色的性能，引入了右值引用和移动语义。右值引用和移动语义听起来很高端，实际上就是解决一个问题，避免大对象的反复销创建和销毁，转而使用代价更低的移动。根本思路就是两条，对于直接量提供了右值引用，以增加它的生存时间，使之可以像普通变量一样通过参数传递。而对于变量来说，提供了移动语义，将不再需要使用的对象管理的资源转移到另一个对想象中。同时增加了移动构造，复制构造方式来优化函数的返回值。可谓是榨干了计算机的每一寸内存。

C++11无疑是C++里程碑式的更新，在对历史遗留问题清理的同时，引领了接下来C++的发展方向，它的作用是承上启下的。对类型系统的改进无疑弥补了最开始从C继承来的一些缺陷。同时也充分考虑了现代计算机的发展，引入了线程支持。在内存管理上也是更上一层楼，引入了智能指针，移动语义，右值引用。它基本上抛开了历史束缚，但依旧是不忘使命，依旧是奔着```更好的静态类型支持，更多的自主性，更高效的资源管理，更克制的特性支持来展开的```。
### C++17，20——新生
C++17和C++20应该是相辅相成的，绝大部分特性都已经得到支持和完善了。但是由于编译器的限制，我用的特性比较少。C++17比较期待的是跨平台的文件系统支持，这对于大部分应用开发者来说无疑是激动和喜悦的。另一个我喜欢的特性是结构化绑定，这个特性我在Python里面用得很顺手，当然现在基本上所有现代语言都支持它了。

而对于C++20就用得更少了，更多的是示例性质的。我比较在意的是模块和协程，但是由于了解得不深入，就不详谈了。

### 什么是C++的基本面
从前几个章节不难看出，我着重夸了C++的类，模板，标准库，类型系统。这些都是我觉得学习C++比较重要的方面。但对于初学者来说，我觉得```类型系统和标准库```就足够了。

类型系统是一门语言最小的单元了，在C++中它包括类型声明，对象初始化，函数传参，函数返回值。在学习初期学多少特性都是骗人的，实际上手还是需要从这个最小的单元入手。比如声明一个变量，这个变量该是什么类型的，可以是指针吗，可以是引用吗。定义函数的时候，参数列表该怎样确定，返回值是什么，怎样才能让函数传参高效，怎样阻止和避免无用的参数检查，返回值该是什么类型，等等，这些都是在实际项目中需要直接面对的问题。所以对类型系统的学习，是写出高效可用代码的第一步，也是最重要的一步。考虑的问题越深入、全面，得到的回报就越大。

标准库则是提供了很好的算法支持和容器支持，可以帮助我们写更健壮的代码。对标准库接口的学习，一方面可以促进对类型系统的认识，另一方面也是积累好习惯的地方。

有了这两项技能的支持，我觉得已经能够写出很棒的应用程序了。但是对于库设计者来说，写出很好的库还需要对类和模板有着更深刻的理解。

一个定义良好的类需要对对象的生命周期进行严格的控制，构造，转移，销毁都是需要控制的。对于需要支持的操作，类设计者应该提供尽可能便捷和高效的支持，对于类禁止的操作，类设计者应该明确禁止，防止发生误用或者隐藏BUG。所以对于类，着重需要关注的是资源的构造，以及在多个对象间的传递和共享。容易发生问题的地方在于函数传参和返回值上，特别是层层调用的函数上，高效和安全就是必须要考虑的了，所以这就回到了前面提到的类型系统，只有对它有了比较深入的了解，才能设计出比较好的类。

模板则是类的另一方面，它和类的概念虽然是不同的，但是思路上却是相通的。模板和Java里面的泛型相似，却更加灵活和重要，是和类一样的高度。模板需要考虑的问题是，提供什么算法，什么对象可以使用这个算法，怎样避免和阻止错误对象的滥用，在使用过程中怎样尽可能利用编译错误来避免运行时错误。所以它是比类更进一步的抽象概念，对开发者有着比类更高的要求。

### C++学习路线图
从上一章节，可以看出我推荐的学习路线是类型系统，到标准库，到类，最后才到模板。其他的语言细节不是说不重要，而是在学习这四大板块的同时会融入到学习过程中，没必要单独去学习和理解，毕竟细节是繁杂而且散乱的，不会增加对语言的掌握，却会打乱学习节奏，分散注意力。

类型系统的学习又可以按以下步骤进行
- 变量声明（常量和编译时常量）
- 初始化（统一初始化，赋值）
- 函数定义，函数参数定义，返回值（引用，指针的使用）
- 简单类定义，不涉及到内存管理，资源管理

标准库可以按以下步骤进行
- 智能指针（`shared_ptr`,`unique_ptr`等）
- 字符串
- 容器类对象（`list`,`map`等）。
- 标准输入输出使用
- 线程库使用
- 通用算法（`sort`，`find`等）

类可以按以下步骤进行
- 类的构造函数，移动构造，复制构造
- 类的运算符重载
- 继承
- 虚函数
- 多继承

模板可以按以下步骤进行
- 模板函数
- 模板类
- 模板递归
- 模板特化

### 总结
C++细节繁多，初学者容易一头扎进语法细节而不自知，最终白白浪费了大把时间不算，还严重打击了学习积极性。本篇的主旨是在帮初学者理清这门语言的主要脉络，并提供我认为比较科学的学习路线，希望对初学者有所帮助。

C++语言是一门通用型语言，有着很长的发展历史。这导致了它有着不小的历史包袱，所以在引入语言特性和怎样引入的事情上一直保持着克制。但是为了更好地服务于现代硬件和简化开发者工作，又不得不引入新特性，遗弃一些老特性。基于这种原因，语言表现出了一定的复杂性和杂乱性。但是它的核心方向是明确的，就是为了更好地解决效率和抽象问题。抓住这两个核心，再结合这份指南，先难后易，抓大放小，再加上一点归纳和总结就能很好地掌握这门语言的大部分内容。对于指南外的特性，在实际项目中需要了再学习完全是来得及的，毕竟大部分时间我们用到的特性也是很少的一部分，应该把精力花在性价比最高的部分。