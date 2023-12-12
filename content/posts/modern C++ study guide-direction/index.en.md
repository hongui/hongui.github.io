---
title: "Modern C++ Study Guide-roadmap"
description: "Modern C++ Study Guide-roadmap"
isCJKLanguage: false
lastmod: 2022-06-25T08:51:01+08:00
publishDate: 2022-06-25T08:51:01+08:00 
author: hongui
categories:
 - C++
tags:
 - Study guide
 - C++ 
toc: true
draft: false
url: post/Modern C++ Study Guide-roadmap.html
---
C++ is a forty-year-old language that has gone through four major version upgrades (Birth, 98, 11, 17(20), 14 counts as a minor upgrade). Each upgrade was a trade-off between many problems and solutions. Understanding this history can better help us to clarify the development of the language. So next I will borrow its development history, talk about my understanding of it, and finally give a guide to what I think is a more reasonable learning route.

### C++0——be born
C++ was created to solve two main problems - performance and abstraction. Performance refers to the ability to have C-like underlying access and execution efficiency, while abstraction is meant to provide the ability to describe problems and ways of thinking at the language level. This is the foundation of C++ and the reason why C++ has endured. The solution that Bjarne Stroustrup came up with for both of these goals was to leverage existing C techniques and tools, and then provide classes to solve the abstraction problem. Based on this premise, we can see that classes are the first hurdle on the road to learning C++.
C++ considers classes to be a form of abstract thinking, and the relevant features of classes are provided in the service of abstraction. So classes in C++ offer more capabilities than other object-oriented classes, and so have more complexity. In order to describe this complexity, it is necessary to mention two features of C++, static type safety, and resource management.
Static type safety helps developers to define custom classes that are more reasonable and legal, e.g., through operator overloading, custom classes can be written with the same concise code as basic types. Runtime errors caused by implicit type conversions can be avoided through constructors, and one can stop one's classes from being abused by explicitly blocking certain operations. All autonomy is up to the developer. So if we are users of the library, we don't have to worry about these details, we just have to write the code as we would in a normal language, and if it doesn't make sense, the compiler will just tell us, and we don't have to worry that these problems will be hidden at some point in the program's runtime.

Resource management, on the other hand, can help developers provide guidance and support for resource management. There are many kinds of resources, and most of the resources in a computer are limited, and must be borrowed and returned, and the borrowing and returning must be one-to-one correspondence, otherwise it is a memory leak. In the C era, resource management relied on the developer's global control of resources, and no better support was provided at the language level. In order to better support resource management, C++ proposes constructors and destructors, which can correspond to the acquisition and recovery of resources. However, in many cases, resources are not only for your own use, but also need to be made available for external use. In order to support this kind of resource transfer, C++ provides move and copy operations.
To summarize, C++ classes provide many features, but not all of them are needed by developers. ```The main question a developer needs to consider when defining a class is what support is provided for the class, and then choosing the appropriate syntactic features to implement among those provided.``` Constructors and destructors provide good one-to-one operations, moves and copies provide how resources can be shared among objects, and operator overloading allows classes to be used more concisely and elegantly.

### C++98——standardized
The biggest upgrades to C++98 are templates and exceptions, and they are paired with a good standard library.
The place of templates in C++ cannot be overemphasized. It is another abstraction mechanism. Classes in C++ address the abstraction of similar concepts, focusing more on the similarity between concepts, whereas templates address generic problems. While templates solve the abstraction of generic problems, focusing more on the generality of concepts. Together, they form the two cornerstones of abstraction in C++. We've 

already talked about classes, so let's focus on templates.
Thanks to C++'s strong static type safety, templates are easy to write, and they can be used to write ordinary functions in any way you like, just by replacing specific types with generic ones. But, on the other hand, templates can do much more. A template can support multiple parameters, multiple arguments, qualified parameters, and be type-safe. What's even more impressive is that it can also specify values. Using types and values together wisely basically solves most problems.

Speaking of exceptions. They don't have much appeal to the average developer. This is because the main problem that exceptions solve is how to tell the caller that an error has occurred, what the error is, and transfer the ability to execute to the caller's side. And most of the time we develop business code, we know what happened, how to solve, most of the time is not much need for exceptions. Of course, it's not that exceptions are useless; they are exceptionally important to the library developer. The library developer needs to be able to tell the caller that an error has occurred and that the operation is not going to work when the exception occurs. Often, however, the library developer doesn't know what the caller should do with the error, whether to ignore it or clean up the mess. The exception mechanism provides both exception throwing and exception catching to support library developers and users.

For newbies, you may not like the standard library much and tend to write your own. This is not a good idea. Standard libraries are industry-tested code that will work in most situations, whereas writing your own code by hand gives you a better sense of accomplishment, but is more likely to carry bugs.Early standard libraries offered limited functionality, with only `string`, input/output streams, bitwise arithmetic, the three main containers, and a few small algorithms. However, these are sufficient for our daily use, especially now that the standard library has become more and more complete, and most programming scenarios can be accomplished with the right tools, it is possible to give up writing specific code by hand.

C++98 was more about standardization, templates were a standard, and the standard library was a standard. Since then, the three pillars of C++ have been completed: classes, templates, and standard libraries. Each of them brings unlimited possibilities and vitality to C++.
### C++11——new language
The changes in C++11 are revolutionary, but retaining incredible compatibility is not easy. We won't go into specific features and details here, just a general overview of the general direction.

The first intuitive change is in the type system, which C++11 standardizes and unifies as much as possible.
- C++11 has standardized and unified the type system as much as possible. It has standardized the form of initialization of objects by agreeing to initialize them;
- The form of type declarations is simplified by `auto`;
- Null pointers are standardized through `nullptr`;
- Static type-safe enumerations are provided through `enum class`;
- simplified type writing through aliases;
- And much, much more.

Improvements to the type system mean that developers can write cleaner, more standardized, and safer code, but the challenges to compilers are huge, so for a long time, C++11 was not well supported, and also hindered the development of C++.
In addition to the type system, another big improvement is the provision of threading support. the standard library of C++11 provides threads, conditional objects, locks, and other threading-related tools, which is revolutionary for library developers. Cross-platform threading support is provided with almost no loss of performance, which greatly improves the stability and performance of the library and saves a lot of time in platform testing, which has to be top notch.

Another important upgrade is resource management. The standard library provides `unique_ptr`, `shared_ptr` to assist in resource management. Right-valued references and move semantics have also been introduced for better performance. Right references and move semantics may sound high end, but they actually solve the problem of avoiding the repetitive creation and destruction of large objects in favor of less expensive moves. The underlying idea is twofold: for direct quantities, right references are provided to increase their lifetime, allowing them to be passed through parameters like normal variables. And for variables, move semantics are provided to transfer resources managed by objects that no longer need to be used to another pair of imagines. Also added move constructs, copy constructs way to optimize the return value of the function. It can be said to drain every inch of memory from the computer.

C++11 is undoubtedly a landmark update to C++, and it carries on the role of cleaning up the historical legacy while leading the way for the rest of C++'s development. Improvements to the type system undoubtedly make up for some of the shortcomings inherited from C at the very beginning. It also took into account the development of modern computers and introduced threading support. It also took memory management to the next level, introducing smart pointers, move semantics, and right-valued references. It basically throws off the historical constraints, but still does not forget its mission, and still runs towards ``better static type support, more autonomy, more efficient resource management, and more restrained feature support``.
### C++17，20——newborn
C++17 and C++20 are supposed to complement each other, and the vast majority of features are already supported and improved. However, due to compiler limitations, there are fewer features that I use. one of the more anticipated features of C++17 is the cross-platform filesystem support, which is certainly exciting and delightful for most application developers. Another feature I like is structured binding, which I use a lot in Python, but of course it's supported by basically all modern languages now.

And for C++20 it's much less used, more of an example nature. I'm more concerned about modules and concurrency, but I won't go into details since I don't know much about them.
### What are the fundamentals of C++
From the first few chapters it is easy to see that I have emphasized boasting about C++ classes, templates, standard library, and type system. These are the more important aspects of learning C++ in my opinion. But for beginners, I think ``The type system and the standard library`` are enough.

The type system is the smallest unit of a language. In C++, it includes type declarations, object initialization, function passing, and function return values. It's a lie how many features you learn at the beginning of the learning process, but in reality you need to start with the smallest unit of the language. For example, when declaring a variable, what type of variable should it be, can it be a pointer, can it be a reference. When defining a function, how to determine the parameter list, what is the return value, how to make the function pass parameters efficiently, how to prevent and avoid useless parameter checking, what type of return value should be, and so on, these are in the actual project need to directly face the problem. So learning about the type system is the first and most important step in writing efficient and usable code. The more in-depth and comprehensive consideration of the issue, the greater the return.

The standard library, on the other hand, provides good algorithmic support and container support that can help us write more robust code. Learning about the standard library interfaces promotes awareness of the type system on the one hand, and on the other hand, it is a place to build up good habits.

With these two skills in place, I feel like I've been able to write great applications. But for library designers, writing great libraries also requires a deeper understanding of classes and templates.
A well-defined class needs to have tight control over the object lifecycle, construction, transfer, and destruction. For operations that need to be supported, the class designer should provide the most convenient and efficient support possible, and for operations that are prohibited by the class, the class designer should explicitly prohibit them to prevent misuse or hidden bugs. so for the class, the focus needs to be on the construction of the resources, as well as the transfer and sharing of resources among multiple objects. Problems are likely to occur in the function passing and return value, especially the function of the layer call, efficiency and safety is a must consider, so this is back to the type system mentioned earlier, only a more in-depth understanding of it, in order to design a better class.

Templates are the other side of the class, it and the class concept is different, but the idea is similar. Templates are similar to generics in Java, but are more flexible and important, and are the same height as classes. Templates need to consider the question of what algorithms to provide, what objects can use this algorithm, how to avoid and prevent the misuse of the wrong object, in the process of use how to avoid run-time errors by using compilation errors as much as possible. So it is a further abstraction than a class, and has higher requirements for the developer than a class.

### C++ Learning Roadmap
From the previous section, you can see that my recommended learning path is the type system, to the standard library, to classes, and finally to templates. The other details of the language are not unimportant, but will be integrated into the learning process while learning the four main sections, there is no need to learn and understand separately, after all, the details are complex and scattered, will not increase the mastery of the language, but will disrupt the learning rhythm and distraction.
The learning of the type system can in turn be carried out in the following steps
- Variable declarations (constants and compile-time constants)
- Initialization (uniform initialization, assignment)
- Function definitions, function parameter definitions, return values (use of references, pointers)
- Simple class definitions, not involving memory management, resource management
Standard libraries can be performed in the following steps
- Smart pointers (`shared_ptr`, `unique_ptr`, etc.)
- Strings
- Container class objects (`list`,`map`, etc.).
- Standard input and output usage
- Threaded library usage
- Generic algorithms (`sort`, `find`, etc.)
Classes can proceed as follows
- Class constructors, move constructors, copy constructors
- Class operator overloading
- Inheritance
- Virtual functions
- Multiple Inheritance
Templating can be done in the following steps
- Template Functions
- Template classes
- Template recursion
- Template specialization
### Summary
C++ is full of details, beginners can easily dive into the details of the syntax without realizing it, and end up wasting a lot of time, not to mention, but also a serious blow to the motivation to learn. The main purpose of this article is to help beginners clear the main vein of this language, and provide me with a more scientific learning route, I hope to help beginners.

C++ is a general-purpose language with a long history of development. It has a lot of historical baggage, so it has been restrained in introducing language features and how to introduce them. However, in order to better serve modern hardware and simplify the work of developers, new features have had to be introduced and old ones left behind. For this reason, the language exhibits some complexity and clutter. But its core direction is clear: to better address efficiency and abstraction. By grasping these two core aspects and combining them with this guide, you can get a good grasp of most of the language by starting with the hard ones and then moving on to the easy ones, and then adding a little generalization and summarization. For the features outside the guide, it's not too late to learn them when you need them in a real project. After all, most of the time the features we use are only a small part of the language, so we should spend our energy on the most cost-effective parts.
