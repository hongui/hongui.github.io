---
title: "Introduction to Android NDK-basic concepts"
description: "Introduction to Android NDK-basic concepts"
isCJKLanguage: false

lastmod: 2022-03-06T11:30:40+08:00
publishDate: 2022-03-06T11:30:40+08:00

author: "hongui"

categories:
 - NDK
tags:
 - Android
 - JNI
 - C/C++
 - NDK

toc: true
draft: false
---

Sometimes it is necessary to use libraries written in C/C++ for security, performance, and code sharing considerations during the development. Although with the support of modern tool chains, the difficulty of this work has been greatly reduced, after all, everything is difficult at the beginning, and beginners often still encounter many unpredictable problems. This article is a simple guide written based on this background. I hope it will be helpful to readers who have just started writing C/C++ libraries. At the same time, in order to reduce cognitive gaps as much as possible, this article will try to start with the simplest function and gradually add tool chains until the final function is achieved, truly knowing what is happening and why.

### Target
The goal of this article is very simple, which is to call C/C++ functions in Android applications - receiving two integer values ​​and returning the value after adding the two. This function is tentatively named `plus`.

### Begin with the C++ source file
In order to start from where we are most familiar, we'll start with the original C++ source files without the use of sophisticated tools.

Open any text editor you like, VS Code, Notpad++, Notepad, create a new text file and save it as `math.cpp`. Next, you can write code in this file.

Our goal, as stated earlier, is to implement a `plus` function that takes two integer values and returns the sum of the two, so it might look like below

```c++
int plus(int left,int right)
{
    return left + right;
}
```

Out work is done, isn't it simple.

But just having the source file is not enough, because this is just for humans, machines can't read it. So we need the first tool - a compiler. A compiler helps us to convert what is human-readable into something that is machine-readable.

### The compiler
The compiler is a complex project, but the two main functions are as follows
1. to understand the content of the source file (human-readable) - to check for syntax errors in the source file
2. to understand the content of the binary (machine-readable) - to generate binary machine code.

Around these two main functions, the compiler needs to complete a lot of work, especially function 2. Based on this difficulty, compilers are divided into a variety of common compilers, such as VS for Windows platform, G++ for Linux platform, Apple's Clang, and for Android, the situation is slightly different, the previous compilers are running on a specific system, compiled programs usually can only run on the corresponding system. The compiled program usually only runs on the corresponding system. Taking my current machine as an example, I'm writing C++ code on Deepin right now, but the goal is to have the code run on an Android phone, two different platforms. More pessimistically, so far, there is no compiler that will run on a phone. Does that mean we can't run C++ code on a phone? Of course not, because there is cross-compilation.

Cross-compilation is the technique of generating code on one platform into executable objects on another. The biggest difference between cross-compilation and normal compilation is in linking. Because the general link directly to the system library to find the appropriate library files, while cross-compilation can not, because the current platform is not the final platform to run the code. So cross-compile also need to have the common libraries of the target platform. Of course, Google has prepared all these for us, called NDK.

### NDK

NDK full name is Native Development Kit, there are many tools, compilers, linkers, standard libraries, shared libraries. These are all essential parts of cross-compilation. In order to understand the convenience, we first take a look at its file structure. Take the version on my machine as an example - `/home/Andy/Android/Sdk/ndk/21.4.7075529` (the default location on Windows is `c:\Users\xxx\AppData\Local\Android\\). Sdk\`). The NDK is stored in the Sdk directory, named `ndk`, and the version number is used as the root directory for that version, as in the example, the version of NDK I installed is `21.4.7075529`. The example is also the value of the `ANDROID_NDK` environment variable. In other words, before determining the environment variable, we need to determine the version of the NDK to use, and the path value is taken to the version number directory.

Knowing where it is stored, we next need to recognize two important directories

- `build/cmake/`, a folder that we'll expand on later.
- `toolchains/llvm/prebuild/linux-x86_64`, the last `linux-x86_64` has a different name depending on the platform, e.g. it starts with Windows on Windows platforms, but you can't go wrong with it because it's just one folder under the path and it's preceded by the same one. There are compilers, linkers, libraries, headers and so on. For example, the compilers are in the `bin` directory in this path, and they all end in `clang` and `clang++`, like `aarch64-linux-android21-clang++`.

1. `aarch64` means that this compiler can generate binaries for use on `arm64` architecture machines, the other equivalents are `armv7a`, `x86_64`, etc. Different platforms use matching compilers. It is the target platform that is referred to in cross-compilation.

2. `linux` means that we perform the compilation operation on a `linux` machine, which is the host platform in cross-compilation.

3. `android21` is obviously the target system version.

4. `clang++` means that it is a C++ compiler, and the corresponding C compiler is `clang`.

As you can see, for Android, different hosts, different instruction sets, different Android versions, all correspond to a compiler.
After learning so much, it's finally time to get excited about the human nature. Next, let's compile the C++ file in front of us.

### Compile

Looking at the parameters via `aarch64-linux-android21-clang++ --help`, you'll see that it has a lot of parameters and options, and now we just want to verify that our C++ source file doesn't have any syntax errors, so we'll just ignore all that complexity, and just a `aarch64-linux-android21- clang++ -c math.cpp`.

After the command is executed, if all goes well, a `math.o` object file will be generated in the same directory as `math.cpp`, which means that our source code has no syntax errors and we can proceed to the next step of linking.

But before that, a quick interruption. Often our projects contain many source files, referencing some third-party libraries, and each time we compile them manually, linking is obviously inefficient and error-prone. Nowadays, when tools are mature, we should try to use mature tools and focus on our business logic, `CMake` is one such tool.

### CMake
CMake is a cross-platform project builder. How to understand it? When writing C++ code, sometimes you need to refer to file headers in other directories, but in the compilation stage, the compiler doesn't know where to look for the headers, so you need a configuration to tell the compiler where to look for the headers. Furthermore, source code distributed in different directories needs to be packaged into different libraries according to certain needs. Or, if the project references third-party libraries, you need to tell the linker where to look for the libraries during the linking phase, and all of these are things that need to be configured.

Different systems and different IDEs have different support for these configurations, such as Visual Studio on Windows, which needs to be configured in the project's properties. When developers use the same tools, the problem is not so big. But once involved in the case of multi-platform, multi-IDE, collaborative development will spend a lot of time in the configuration of the CMake is to solve these problems came into being.

CMake configuration information is written in a file called `CMakeLists.txt`. As I mentioned earlier, header file references, source code dependencies, library dependencies, etc., only need to be written once in `CmakeLists.txt`, and can be used seamlessly on all major IDEs on Windows, MacOS, and Linux platforms. For example, I created a CMake project on Visual Studio for Windows, configured the dependencies, and passed it to a coworker. When my colleague develops on MacOS, he can immediately finish compiling, packaging, testing, etc. without any modification. This is the power of CMake cross-platform - simple, efficient, flexible.

### Manage project with CMake

#### Create a CMake project
We already have `math.cpp` and CMake above, so let's combine them now.

How do we create a CMake project? There are three steps:

1. Create a folder

In the example, let's create a folder `math`.

2. Create a new `CMakeLists.txt` text file in the new folder. Note that the name of the file cannot be changed.

3. Configure the project information in the new `CMakeLists.txt` file.
The simplest CMake project needs to include at least three infomation
1) Minimum CMake version supported
```cmake
cmake_minimum_required(VERSION 3.18.1)
```
2) Project name

```cmake
project(math)
```

3) Product - The product could be executables or libraries. Since we are need a libraries on Android, so the product is a library.

```cmake
add_library(${PROJECT_NAME} SHARED math.cpp)
```

After these three steps, the CMake project is built. Let's try compiling the project with CMake in the next step.

#### Compile the CMake project

Before executing the real compilation, CMake has a preparation phase, in which CMake collects the necessary information and then generates a project that meets the conditions before it can execute the compilation.

What is the necessary information? CMake will collect some information by guessing in order to minimize the complexity.

For example, if we perform the generation operation on Windows, CMake will default to Windows as the target platform and generate the VS project by default, so compiling Windows libraries on Windows is almost zero configuration.

1. Create a new `build` directory in the `math` directory and switch the working directory to the `build` directory.
   
   ```shell
   cd build
   cmake ..
   ```
   
After the command is executed, you will find the VS project in the `build` directory, and you can open it directly with VS and compile it without errors. Of course, the faster way is to compile directly with CMake.

2. Compile
   
   ```shell
   cmake --build .
   ```
   
   Note that the preceding `..` represents the parent directory, the `math` directory where the `CMakeLists.txt` file exists, and `. ` represents the current directory, `build`. If both of these steps are executed successfully, we will be able to harvest a library file in the build directory, which may be called `math.dll` on Windows platforms and `math.so` on Linux platforms, but both are dynamic libraries, because that is what we configured in the `CMakelists.txt` file.

From the above process, CMake's workflow is not complicated. But we are using the default configuration, which means that the final generated library can only be used on the compiled platform. To use CMake to compile Android libraries, we need to manually tell CMake some configurations when generating the project, instead of letting CMake guess.

### Cross-compilation of CMake

#### Where do the configuration parameters come from?

Although we do not know what is the minimum configuration to complete the cross-compilation, but we can guess.

First of all, to complete the compilation of the source code, compiler and linker is indispensable, we also know that the Android platform has a special compiler and linker, so at least one configuration should be to tell CMake with which compiler and linker.

Secondly, Android's system version and architecture is also essential, after all, for Android development, this is very important for Android applications.

Can you think of any other parameter, I can't seem to think of any. However, the good news is that Google has done it for us, and that is to use `CMAKE--TOOLCHAIIIN_FILE` directly. This option is provided by CMake, just set the configuration file path to its value, CMake will find the target file through this path, and use the configuration inside the target file instead of its own guessing parameters. The configuration file is `build/camke`, one of the two important folders mentioned earlier, and our configuration file is `android.toolchain.cmake` under that folder.

#### The CMake of Google

`android.toolchain.cmake` plays the role of a wrapper that will work together to configure CMake using the parameters provided to it, and the default configuration. In fact, this file is a good source for learning about CMake, and you can learn a lot of CMake tricks. Now, let's not learn CMake-related first, let's see what parameters we have available. In the beginning of the file, Google will be configurable parameters are listed out

```cmake
ANDROID_TOOLCHAIN
ANDROID_ABI
ANDROID_PLATFORM
ANDROID_STL
ANDROID_PIE
ANDROID_CPP_FEATURES
ANDROID_ALLOW_UNDEFINED_SYMBOLS
ANDROID_ARM_MODE
ANDROID_ARM_NEON
ANDROID_DISABLE_FORMAT_STRING_CHECKS
ANDROID_CCACHE
```
These parameters are not actually CMake parameters; they are converted to real CMake parameters as the configuration file is executed. We can specify the values of these parameters to allow CMake to fulfill different build requirements. If you don't specify any of them, the default values will be used, which may be different for different NDK versions.

Let's focus on the most critical `ANDROID_ABI` and `ANDROID_PLATFORM`. The first one refers to which CPU instruction set the currently built package is running on, the available values are `arneabi-v7a`, `arn64-v8a`, `x86`, `x86_64`, `mips`, `mips64`. The latter one refers to the Android version of the build package. Its value takes two forms, one is the direct `android-[version]` of the form `[version]` which is replaced with the specific system version when used, e.g., `android-23`, which means that the minimum supported system version is Android 23. The other form is the string `latest`. This value is as the word implies, use the latest.

So how do we know which parameter can take which values? There's an easy way: first identify the parameter you want to see in the header of the file, then search globally and look at the `set` and `if` related statements to determine the parameter forms it supports.

#### Complete cross-compilation using configuration files

With that out of the way, let's go back to the original example. Now we have `CMakelists.txt`, we have `math.cpp`, and we have found the configuration file `android.toolchin.cmake` for Android. So how do you combine the three, which brings us to CMake's parameter configuration.

In the previous section, we completed the configuration of the project file generation by directly using the following command

```shell
cmake ..
```

But it is actually possible to pass parameters, ***CMake's parameters are all key-value pairs that start with `-D` and are separated by whitespace.*** And CMake's default parameters all start with `CMAKE`, so most of the time the parameters are of the form `-DCMAKE_XXX`For example, passing a `toolchain` file to CMake would look like this

```shell
cmake -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake
```

The point of this parameter is to tell CMake to use the file specified after `=` to configure CMake's parameters

However, to complete the cross-compilation, we are missing one more option - `-G`. This option is required for cross-compilation. Because cross-compiling CMake does not know what form of project to generate, this option is needed to specify the type of project to generate. One type of project is the traditional Make project, which is specified as follows.

```shell
cmake -G "Unix Makefiles"
```

As you can see, this form is based on the Unix Make project, which uses `make` as the build tool, so after specifying this form, you also need to specify the path to `make` for the project to be compiled successfully. The other Google-recommended way is `Ninja`, which is simpler because you don't need to specify the path to `Ninja` separately, and it is installed in the same directory as CMake by default, so you can reduce the number of passing parameters. `Ninja` is also a build tool, but focuses on speed, so we'll use `Ninja` this time. It's specified like this

```shell
cmake -GNinja
```

Combining the above two parameters gives you the final compilation command

```shell
cmake -GNinja -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake ..
```

Generate a project and then compile it

```shell
cmake --build .
```

We've got a dynamic library that will eventually run on Android. The dynamic library compiled with my version of the NDK supports Android version 21, and the instruction set is armeabi-v7a. Of course, based on the previous description, we can pass the desired parameters as we did earlier with the `toolchain` file, e.g., to build the `x86` library with the latest version of Android, you can write something like this

```shell
cmake -GNinja -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake -DANDROID_PLATFORM=latest -DANDROID_ABI=x86 ..
```
This gives us the idea that if some third-party libraries don't provide a compilation guide, but are managed by CMake, we can just apply the above formula to compile the third-party libraries.

### JNI

With the help of CMake, we have got the `libmath.so` dynamic library, but this library still can't be used directly by Android apps, because Android apps are developed in Java (Kotlin) language, and they are all JVM languages, the code is running on the JVM. To use the library, you also need to find a way to get the library loaded into the JVM and then you can access it. It happens that the JVM really does have this capability, it is JNI.

#### JNI basic concepts

JNI can provide bi-directional access from Java to C/C++, that is, you can access C/C++ methods or data in Java code, and vice versa, the same support, which is the process of the JVM can not be ignored. So to understand JNI technology, we need to think in terms of JVM.

JVM is like a goods distribution center, no matter where the goods need to come to this distribution center, and then through it to distribute the goods to the destination. The goods here can be Java methods or C/C++ functions. But unlike ordinary courier is that the goods here do not know where their destination is, you need to find the distribution center itself. Then find the basis from where it is, that is, how to ensure that the distribution center to find the results of the uniqueness of it, the simplest way is, of course, the goods themselves to identify their own, and to ensure its uniqueness.

Obviously this is a good problem for Java, which has layers of mechanisms to guarantee uniqueness.

1. the package name guarantees the uniqueness of the class name;
2. the class name can guarantee the uniqueness of the class under the same package name; 3. the method name can guarantee the uniqueness under the same class; and
3. method names can be used to guarantee uniqueness under the same class. 4;
4. method overloading can be used to determine the uniqueness of the class by the type and number of parameters.

For C/C++, there is no package name and class name, so can we determine the uniqueness with method name and method parameters? The answer is yes, as long as we use the package name and class name as a kind of qualification.

There are two ways to add qualifications, one is simple and crude, directly to the package name class name as part of the function name, so that the JVM does not have to look at other things, directly crude package name, class name, function name and parameters of these correspond to determine the corresponding method on the other end. This method is called static registration. In fact, it's very similar to broadcasting in Android: static registration for broadcasting is just brute-force writing in the `AndroidManifest` file, so you don't have to configure it in the code, and it takes effect as soon as it's written. In contrast to static registration, there must be a dynamic registration method. Dynamic registration is writing code that tells the JVM what functions correspond to each other, rather than having it look them up when the function is called. Obviously the advantage of this approach is that the call is a little faster, after all, we only need to register once, you can in subsequent calls directly access to the counterpart, no longer need to find the operation. However, the same and Android broadcast dynamic registration, dynamic registration is much more cumbersome, and dynamic registration should also pay attention to grasp the timing of registration, otherwise it is easy to cause the call to fail. We continue to `libmath.so` as an example.

#### Use local library on the Java

Accessing C/C++ functions on the Java side is simple, in three steps:

1. Java calls `System.loadLibrary()` method to load the library.
   
   ```java
   System.loadlibrary("math.so");
   ```
   
   It's worth noting here that CMake generates a dynamic library called `libmath.so`, but here it's just `math.so`, which means that you don't need to pass the `lib` prefix. After this step, the JVM knows that there is a `plus` function.

2. Java declares a `native` method that corresponds to a C++ function. Correspondence means that the parameter list and return value should be the same, but the method name can be different.
   
   ```java
   public native int nativePlus(int left,int right);
   ```
   
   Often, it is customary to prefix `native` methods with `native`.

3. Call this `native` method directly where needed. Calling the method is the same as a normal Java method, passing matching parameters and receiving the return value with the matching type.

Combining these steps into a single class looks like this
```java
package hongui.me;

import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import hongui.me.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    static {
        System.loadLibrary("me");
    }

    ActivityMainBinding binding;

    private native int nativePlus(int left,int right);

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // Example of a call to a native method
        binding.sampleText.setText("1 + 1 = "+nativePlus(1,1));
    }
}
```

#### Introducing JNI on the C/C++ side
JNI is actually for C/C++ is a layer of adaptation layer, in this layer mainly do the work of function conversion, do not do the implementation of specific functions, so, in general, we will create a new source file, used to deal with the JNI layer of the problem, and the JNI layer of the most important problem is, of course, the method of registration problems mentioned earlier.
##### Static registration
The basic idea of static registration is to write a C/C++ function signature corresponding to an existing Java `native` method, specifically in four steps.

1. Start by writing the exact same function signature as the Java `native` function
```c++
int nativePlus(int left,int right)
```

2. Add the package name and class name in front of the function name. Because package names are split in Java with `. ` split, whereas in C/C++ dots are usually used as function calls, to avoid compilation errors, you need to replace `. ` is replaced with `_`.
```c++
hongui_me_MainActivity_nativePlus(int left,int right)
```

3. Converting function parameters. As mentioned earlier all operations are JVM based, and in Java these are natural, but in C/C++ there is no JVM environment, and providing a JVM
environment would have to be in the form of adding parameters. In order to do this, any JNI function has to add two parameters at the beginning of the parameter list. The smallest environment inside Java is a thread, so the first parameter is the thread environment object `JNIEnv`, which represents the caller's thread environment when calling this function, and this object is the only way for C/C++ to access Java. The second is the calling object. Since you can't call methods directly in Java, you need to call them through a class name or a class, the second argument represents that object or class, which is of type `jobjet`. Starting from the third parameter, the parameter list corresponds to the Java side, but only just, after all, there are some types that are not available in the C/C++ side, which is the type system in JNI, for our current example the `int` value in Java corresponds to the `jint` value in JNI, so the last two parameters are of type `jint`. This is a critical step, as failure to convert any of the parameters can cause the program to crash.
```c++
hongui_me_MainActivity_nativePlus(
        JNIEnv* env,
        jobject /* this */,
        jint left,
        jint right)
```

4. Add the necessary prefixes. This step can be easily overlooked because this part doesn't come so naturally. First we have to add a prefix `Java` to the function name, which now looks like this `Java_hongui_me_MainActivity_nativePlus`. Secondly, you need to add `JNIEXPORT` and `JNICALL` at the end of the return value, here the return value is `jint`, so after adding these two macros it looks like this `JNIEXPORT jint JNICALL`. Finally, you have to add the `extern "C" ` compatibility directive at the beginning. As to why this step is added, interested readers can go to the details, the simple summary is that this is the JNI specification.

After these four steps, the final version of C/C++ function signature looks like this
```c++
#include "math.h"

extern "C" JNIEXPORT jint JNICALL
Java_hongui_me_MainActivity_nativePlus(
        JNIEnv* env,
        jobject /* this */,
        jint left,
        jint right){
           return plus(left,right);
        }
```

Notice that here I changed the previous `math.cpp` to `math.h` and called the function in the JNI adaptation file (filename is `native_jni.cpp`). So now there are two source files, need to update `CMakeList.txt` a bit.
```cmake
cmake_minimum_required(VERSION 3.18。1)

project(math)

add_library(${PROJECT_NAME} SHARED native_jni.cpp)
```
You can see that we only change the last line of the filename, because `CMakeLists.txt` is currently located in the directory is also `include` lookup directory, so do not need to give it a separate value, if you need to add other locations of the header file can be used to `include_directories(dir)` to add.

Now use CMake to recompile and generate dynamic libraries, and this time Java will run directly without errors.

##### Dynamic registration
As mentioned earlier dynamic registration needs to pay attention to the timing of registration, so what is considered a good time? In the previous section of Java's use of local libraries, we know that in order to use the library, you must first be loaded, loaded after the success of the JNI methods can be called. Then dynamic registration must occur after loading, before use. JNI is very humane to think of this, in the library after the completion of loading will immediately call `jint JNI_OnLoad (JavaVM *vm, void *reserved)` function, this method also provides a key `JavaVM` object, it is simply the best entry point to the dynamic registration of the It's simply the best entry point for dynamic registration. Having determined the timing of the registration, let's now do it in practice. ***Note: Dynamic registration and static registration are both ways of implementing JNI functions on the C/C++ side, and there is generally only one registration method for the same function.*** So, the next steps are parallel to static registration, not sequential.

Dynamic registration in six steps

1. Create a new `native_jni.cpp` file and add the implementation of the `JNI_OnLoad()` function.
```c++
extern "C" JNIEXPORT jint JNICALL
JNI_OnLoad(JavaVM *vm, void *reserved) {

   return JNI_VERSION_1_6;
}
```
This is the standard form and implementation of this function, the previous string are the standard form of JNI function, the key point is the function name and parameters and return value. In order for this function to be called automatically after the library is loaded, the function name must be this, and the parameter form can not be changed, and the final return value to tell the JVM the current JNI version. In other words, these are templates, just copy.

2. Get `JNIEnv` object

As mentioned earlier, all JNI-related operations are done through the `JNIEnv` object, but now we only have a `JavaVM` object, so obviously the secret is in the `JavaVM`.
The secret lies in the `JavaVM`. You can get the `JNIEnv` object through its `GetEnv` method.
```c++
JNIEnv *env = nullptr;
vm->GetEnv(env, JNI_VERSION_1_6);
```

3. Find the target class

As I said earlier, both dynamic and static registration are most qualified by package and class names, just not in the same way. So for dynamic registration we still have to use the package name and class name, but this time in a different form. Static registration uses `_` instead of `. `, this time we're going to use `/` instead of `. `. So we end up with a class of the form `hongui/me/MainActivity`. This is a string form, so how do we convert it to a `jclass` type in JNI, which is where `JNIEnv` from step 2 comes in.
```c++
jclass cls=env->FindClass("hongui/me/MainActivity");
```
This `cls` object is a one-to-one correspondence with the `MainActivity` in Java. With the class object the next step is of course the methods.

4. Generate an array of JNI function objects.

Because dynamic registration can register multiple methods of a class at the same time, the registration parameters are in the form of an array, and the type of the array is `JNINativeMethod`. The purpose of this type is to associate the `native` method on the Java side with the JNI method, how it is done, look at its structure
```c
typedef struct {
const char* name;
const char* signature;
void* fnPtr;
} JNINativeMethod;
```
* `name` corresponds to the name of the `native` method on the Java side, so the value should be `nativePlus`.
* `signature` corresponds to the parameter list of the `native` method plus the signature of the function type.

What is a signature? It is a type shorthand. There are eight basic types in Java, as well as methods, objects, classes. Arrays and so on, all of these things have a set of corresponding string forms, as if it were a hash table, where the keys are string representations of the types, and the values are the corresponding Java types. For example, `jint` is a true JNI type, its type signature is `I`, which is the initial capitalization of `int`.

Functions also have their own type signature `(paramType)returnType` Here both `paramType` and `returnType` need to be JNI type signatures, with no separator between types.

In summary, the type signature of `nativePlus` is `(II)I`. Two integer arguments return another integer.

* `fnPtr` is, as its name suggests, a function pointer, and the value is our real `nativePlus` implementation (which we haven't implemented yet here, so let's assume it's `jni_plus` for now).

To summarize, the final array of function objects should look like this
```c++
 JNINativeMethod methods[] = {
    {"nativePlus","(II)I",reinterpret_cast<void *>(jni_plus)}
 };
```

5. Registration

Now that you have the `jclass` object representing the class, and the `JNINativeMethod` array representing the methods, and the `JNIEnv` object, combine them to complete the registration
```c++
env->RegisterNatives(cls,methods,sizeof(methods)/sizeof(methods[0]));
```
The third parameter represents the number of methods. We use the `sizeof` operation to get the size of all the `methods`, and then we use `sizeof` to get the size of the first element to get the number of `methods`. Of course, it's fine to just manually fill in 1 here.

6. Implementing JNI Functions

In step 4, we used a `jni_plus` to represent the native implementation of `nativePlus`, but this function hasn't actually been created yet, so we need to define it in the source file. Now the name of the function can be whatever you want, it doesn't have to be as long as the static registration, just keep the final function name the same as the one used in the registration. However, the prefix `extern "C"` should be added here, to avoid the compiler to do something special with the function name. The argument list is identical to the static registration. So, our final function implementation is as follows.
```c++
#include "math.h"

extern "C" jint jni_plus(
        JNIEnv* env,
        jobject /* this */,
        jint left,
        jint right){
           return plus(left,right);
        }
```
Well, the implementation of dynamic registration is now complete, and after CMake compiles it, you'll see that the result is exactly the same as static registration. So it's up to you to decide what you want and how you want to do it. When you need to call the `native` method a lot, I think dynamic registration is an advantage, but if you call it very rarely, you can just use static registration, and the lookup consumption is completely negligible.

### One more thing
I mentioned earlier that CMake is a master at managing C/C++ projects, but for Android development, Gradle is the way to go. Google realizes this too, so the gradle plugin provides a silky smooth configuration for CMake and Gradle to work seamlessly together directly. Under the `android` build block, you can directly configure the path and version information of `CMakeLists.txt`.
```groovy
externalNativeBuild {
        cmake {
            path file('src/main/cpp/CMakeLists.txt')
            version '3.20.5'
        }
    }
```
This way, if you change your C/C++ code or Java code, you can just click run and gradle will compile the libraries and copy them to the final directory, so you don't need to compile and copy the libraries manually anymore. Of course, if you are not satisfied with the default behavior, you can configure the default behavior via `defaultConfig`, which looks like this
```groovy
android {
    compileSdkVersion 29

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 29

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles 'consumer-rules.pro'

        externalNativeBuild {
            cmake {
                cppFlags += "-std=c++1z"
                arguments '-DANDROID_STL=c++_shared'
                abiFilters 'armeabi-v7a', 'arm64-v8a'
            }
        }
    }
}
```
Here, `cppFlags` specifies C++-related arguments, and there's a corresponding `cFlags` that specifies C-related arguments. `arguments` is to specify the compilation parameters of CMake, the last one is familiar with the library will eventually be compiled to generate a few architectural packages, we are here just to generate two.

With these configurations, Android Studio development NDK is exactly like the development of Java, there are intelligent prompts, can be compiled instantly, run instantly, enjoy the silky smooth.

### Sumary
NDK development should actually be divided into two parts, C++ development and JNI development.
C++ development is exactly the same as C++ development on PC, you can use standard libraries, you can refer to third-party libraries, with the expansion of the project scale, CMake was introduced to manage the project, which has obvious advantages for cross-platform projects, and can also be seamlessly integrated into Gradle.
JNI development is more concerned about the correspondence between the C/C++ side and the Java side, each `native` method on the Java side should have a corresponding C/C++ function to correspond to it, JNI provides
 JNI provides both static registration and dynamic registration to accomplish this work, but the core is to use the package name, class name, function name, and parameter list to determine the uniqueness. Static registration reflects the package name and class name in the function name, while dynamic registration uses the class object, local method object, and `JNIENV` registration method to achieve uniqueness.
NDK is the big boss behind, it provides compiler, linker and other tools to accomplish cross-compilation, and some system libraries, such as `log`, `z`, `opengl` and so on for us to use directly.