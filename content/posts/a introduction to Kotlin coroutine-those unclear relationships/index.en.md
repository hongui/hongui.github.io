---
title: "A introduction to Kotlin coroutine-those unclear relationships"
description: "A introduction to Kotlin coroutine-those unclear relationships"
isCJKLanguage: false

lastmod: 2021-03-26T18:50:29+08:00
publishDate: 2021-03-26T18:50:29+08:00

author: hongui

categories:
 - Coroutine
tags:
 - Android
 - Kotlin
 - Coroutine

toc: true
draft: false
url: post/A introduction to Kotlin coroutine-those unclear relationships.html
---

> Kotlin's coroutines have been increasingly sought after by Android developers since its introduction. On the other hand, due to its huge API, it has also shut out a considerable number of developers. This article tries to start from a few important concepts of coroutines, in the complex API to restore its original face, to bring readers into the Kotlin coroutine world with a new perspective.

### What is coroutine
In many articles about coroutines, describing them is usually done with this one sentence description - coroutines are more lightweight than threads and are cancelable. There is nothing wrong with this statement, both are advantages of coroutines, but they are not characteristics, and it doesn't explain what coroutines are. So what are the characteristics of a coroutine, I think we can start with an analogy with threads, ***The best way to explain a concept is by analogy.*** Instead of using a scientifically rigorous description, I'd like to give thread my own definition - a thread is an execution unit that is available for CPU scheduling, it has its own execution block and can execute logic independently. I've deliberately listed three key characteristics of a thread:

- Threads are scheduled by the CPU
- Threads have their own blocks of code
- Code blocks are required in order to schedule execution

This is my intuition about threads and that these characteristics are able to be reflected in the code. For example, `Thread` in Java is a thread and at the same time an entity object that can be recognized and used through the API. And what makes it special are the three things I listed above, all of which are thread-specific. I'll try to give my own definition of a coroutine along the same lines - a coroutine is an execution unit that is scheduled for execution by a thread, and it also has its own execution block that can execute logic independently or collaboratively. In fact, Kotlin's coroutines also have their own corresponding entities and operation APIs, and even threads are also very similar (such as the life cycle), but because Kotlin's encapsulation of coroutines is too thorough, many APIs are not exposed, so my understanding of coroutines has been in the state of blind men feeling the elephant. In addition, in fact, there are a variety of ways to implement the coroutine, the following my views only for Kotlin's implementation of the coroutine, may not be consistent with the implementation of other languages.

***A coroutine object in Kotlin is essentially an executable block of code,***and the code that is executed is passed in to create the coroutine. In addition to this, it has another great feature - a coroutine is not thread-bound. It can disconnect from the current thread at some point and then attach to another thread at another time. In other words, it's like a ping pong ball, and threads are like paddles that it can repeatedly hop across between. So, combining these two characteristics, the official definition of a coroutine is ***a suspend computational entity***. I don't think this definition is precise, it only reflects the concept of hanging, not the concept of recovery. I'll give it a definition of my own - ***a computational entity that can be scheduled***.

### A few key concepts in coroutine
Understanding what a coroutine is isn't enough, because there's a lot more to Kotlin's coroutine, and there are a few key concepts to understand.

#### Suspend function
The mention of Kotlin's coroutines necessitates the mention of the suspend function, which is one of the most important concepts of Kotlin's coroutine implementation. Simply put, ***a suspend function is an asynchronous implementation solution that has the properties of pending and resuming on the premise that it is a normal function.*** It is a solution to time-consuming computation and result passing. So how does it compare to our common callback based scheme. In the callback-based scheme, the computation process and the result have no relationship, the result needs to be passed through another object, after the completion of the calculation by the computation process manually passed, and this process is possible to be repeatedly nested, which leads to, some of the code that should be serialized, was cut into several parts, scattered into different blocks of code. For example, the following code reads and writes a file
```kotlin
 // asynchronously read into `buf`, and when done run the lambda
 inChannel.read(buf) {
     // this lambda is executed when the reading completes
     bytesRead ->
     ...
     ...
     process(buf, bytesRead)
 
     // asynchronously write from `buf`, and when done run the lambda
    outChannel.write(buf) {
        // this lambda is executed when the writing completes
        ...
        ...
        outFile.close()          
    }
}
```
What can the same logic be written with `read` and `write` implemented as suspend functions?
```kotlin
 launch {
     // suspend while asynchronously reading
     val bytesRead = inChannel.aRead(buf) 
     // we only get to this line when reading completes
     ...
     ...
     process(buf, bytesRead)
     // suspend while asynchronously writing   
     outChannel.aWrite(buf)
    // we only get to this line when writing completes  
    ...
    ...
    outFile.close()
}
```
This is an official example given by Kotlin, and you can see that the implementation of the suspend function is very intuitive, is consistent with the thought process, and also reduces a lot of nesting.

To better explain the suspend function, I also need to introduce a new concept - the suspend point.
A suspend point is a demarcation point that represents the point after which the execution process may move to execute elsewhere and then, at some point, resume from that point and continue down the line. The current thread is not blocked during this process. So ***the suspend function actually implements an asynchronous non-blocking communication model. ***

In a nutshell, a suspend function is a function that does not block the current thread and returns the result of an asynchronous computation.

#### Concurrent Creators
The previously mentioned pending function is good, but there is a limitation. Ordinary methods cannot call pending functions, they can only be called through a pending function. So the question arises whether the chicken or the egg comes first. The solution to this problem is the coroutine creator. `launch`, `future`, `sequence` are all coroutine creators. As the name suggests, coroutines are used to create coroutine objects and are otherwise indistinguishable from normal functions. They are the gateway to the world of coroutines and hung functions. Inside this gateway, we can use pending functions to our heart's content, simplifying our computations. Of course, none of this is fixed, and these functions have several configuration parameters, the most important of which is `CoroutineContext`.

#### `CoroutineContext`
The role of `CoroutineContext` is to provide various configuration information for the coroutine, ***essentially a container (Set)*** that holds non-duplicated elements that can be fetched based on a Key (e.g., a scheduler), called an Element. Here, I couldn't resist putting up its interface definition because it's just so beautiful.
```kotlin
 interface CoroutineContext {
     operator fun <E : Element> get(key: Key<E>): E?
     fun <R> fold(initial: R, operation: (R, Element) -> R): R
     operator fun plus(context: CoroutineContext): CoroutineContext
     fun minusKey(key: Key<*>): CoroutineContext
 
     interface Element : CoroutineContext {
         val key: Key<*>
     }

    interface Key<E : Element>
}
```
These are the definitions of `CoroutineContext` in Kotlin, and each of these APIs has its own clever use, which is breathtaking!

- The purpose of `get` is to get the corresponding object based on the Key, and the quirky thing about this method is the query parameter. This method is used to determine if `CoroutineContext` has a certain configuration object before performing an operation, thus enabling a kind of privilege authentication.

- `fold` is actually an iterative algorithm that checks for all elements.

- `plus` which is interesting in that it allows two objects to be merged and uses the right object to overwrite the left object when the `key` is the same. This is definitely the most flexible API in our use of coroutines. We can use + to replace the original scheduler with our given scheduler, and it looks so natural.

- `minusKey` returns a `context` that doesn't contain the specified `key`, which is equivalent to an inverse operation, which can be very useful in certain contexts.

I think this should be considered the ultimate embodiment of abstraction, this interface with a simple API abstraction of the four operations of add, delete, change and check, and retain a strong scalability. At the very beginning of my contact with coroutines, I often had deep doubts about the complex working mechanism and simple parameter configuration of coroutines, until I saw this definition, I realized how powerful it really is, not only can it use the system's default working configuration to complete the work, but also allows the user to implement their own `CoroutineContext` to replace the default configuration at any time, to complete their own customized tasks.

#### `Continuation`
`Continuation` is not a Kotlin-specific concept; it is explained on the wiki as an abstract representation of control state. In Kotlin, it is an abstraction of the state of a coroutine at the starting point of a hang, which may not be well understood, and we can concretize this concept with a specific API.
```kotlin
interface Continuation<in T> {
   val context: CoroutineContext
   fun resumeWith(result: Result<T>)
}
```
It has a key function, resumeWith, which indicates that the state object is resumed from its original position by this state object at some point after the hung state. This is the key to the implementation of the resume function. The state of control here is represented by the parameters, success or failure, so it has two more extension methods:
```kotlin
fun <T> Continuation<T>.resume(value: T)
fun <T> Continuation<T>.resumeWithException(exception: Throwable)
```
### Summary
A coroutine is a schedulable computational entity that can be created by a coroutine creator. A pending function can be used in a coroutine's code block, which can hang when necessary and then resume when conditions are met, completing the serialized programming of asynchronous code.

The above is to understand the key concepts of coroutines, in the actual use of coroutines in the process may not be used a lot, but it will help us to understand the process of its operation, but also to write a standard coroutine code is the key to the Kotlin coroutines do not have a lot of black magic, just for the application of a variety of different scenarios, there is a huge API, this article is a general explanation of these APIs, and later on, will be for a variety of scenarios in detail. This post is a general explanation of these APIs, later will be for a variety of scenarios and then combed in detail, I hope you enjoy.