---
title: "Android NDK开发——基本概念"
description: "Android NDk开发——基本概念  "

date: 2022-03-06T11:30:40+08:00

author: "hongui"

categories:
 - NDK
tags:
 - Android
 - JNI
 - C++
 - NDK

draft: false
---
在Android开发中,有时候出于安全，性能，代码共用的考虑，需要使用C/C++编写的库。虽然在现代化工具链的支持下，这个工作的难度已经大大降低，但是毕竟万事开头难，初学者往往还是会遇到很多不可预测的问题。本篇就是基于此背景下写的一份简陋指南，希望能对刚开始编写C/C++库的读者有所帮助。同时为了尽可能减少认知断层，本篇将试着从一个最简单的功能开始，逐步添加工具链，直到实现最终功能，真正做到知其然且之所以然。

## 目标
本篇的目标很简单，就是能在Android应用中调用到C/C++的函数——接收两个整型值，返回两者相加后的值，暂定这个函数为`plus`。

## 从C++源文件开始
为了从我们最熟悉的地方开始,我们先不用复杂工具,先从最原始的C++源文件开始.
打开你喜欢的任何一个文本编辑器，VS Code，Notpad++，记事本都行，新建一个文本文件，并另存为`math.cpp`。接下来,就可以在这个文件中编写代码了.
前面我们的目标已经说得很清楚,实现个`plus`函数，接收两个整型值，返回两者之和，所以它可能是下面这样

```c
int plus(int left,int right)
{
    return left + right;
}
```
我们的源文件就这样完成了，是不是很简单。
但是仅仅有源文件是不够的，因为这个只是给人看的，机器看不懂。所以我们就需要第一个工具——编译器。编译器能帮我们把人看得懂的转化成机器也能看得懂的东西。
## 编译器
编译器是个复杂工程，但是都是服务于两个基本功能
1. 理解源文件的内容（人能看懂的）——检查出源文件中的语法错误
2. 理解二进制的内容（机器能看懂的）——生成二进制的机器码。
基于这两个朴素的功能，编译器却是挠断了头。难点在于功能2。基于这个难点编译器分成了很多种，常见的像Windows平台的VS，Linux平台的G++,Apple的Clang。而对于Android来说，情况略有不同，前面这些编译器都是运行在特定系统上的，编译出来的程序通常也只能运行在对应的系统上。以我现在的机器为例，我现在是在Deepin上写的C++代码，但是我们的目标是让代码跑在Android手机上，是两个不同的平台。更悲观的是，目前为止，还没有一款可以在手机上运行的编译器。那我们是不是就不能在手机上运行C++代码了？当然不是，因为有交叉编译。
交叉编译就是在一个平台上将代码生成另一个平台可执行对象的技术。它和普通编译最大的不同是在链接上。因为一般的链接直接可以去系统库找到合适的库文件，而交叉编译不行，因为当前的平台不是最终运行代码的平台。所以交叉编译还需要有目标平台的常用库。当然，这些Google都替我们准备好了，称为NDK。
## NDK
NDK全称是Native Development Kit，里面有很多工具，编译器，链接器，标准库，共享库。这些都是交叉编译必不可少的部分。为了理解方便，我们首先来看看它的文件结构。以我这台机器上的版本为例——`/home/Andy/Android/Sdk/ndk/21.4.7075529`（Windows上默认位置则是`c:\Users\xxx\AppData\Local\Android\Sdk\`）。 NDK就保存在Sdk目录下，以`ndk`命名，并且使用版本号作为该版本的根目录，如示例中，我安装的NDK版本就是`21.4.7075529`。同时该示例还是`ANDROID_NDK`这个环境变量的值。也就是说，在确定环境变量前，我们需要先确定选用的NDK版本，并且路径的值取到版本号目录。

了解了它的存储位置，接下来我们需要认识两个重要的目录

- `build/cmake/`，这个文件夹，稍后我们再展开。
- `toolchains/llvm/prebuild/linux-x86_64`，最后的`linux-x86_64`根据平台不同，名称也不同，如Windows平台上就是以Windows开头，但是一般不会找错，因为这个路径下就一个文件夹，并且前面都是一样的。这里有我们心心念念的编译器，链接器，库，文件头等。如编译器就存在这个路径下的`bin`目录里，它们都是以`clang`和`clang++`结尾的，如`aarch64-linux-android21-clang++`

1.  `aarch64`代表着这个编译器能生成用在`arm64`架构机器上的二进制文件，其他对应的还有`armv7a`，`x86_64`等。不同的平台要使用相匹配的编译器。它就是交叉编译中所说的目标平台。 
2.  `linux`代表我们执行编译这个操作发生在`linux`机器上，它就是交叉编译中所说的主机平台。 
3.  `android21`这个显然就是目标系统版本了 
4.  `clang++`代表它是个C++编译器，对应的C编译器是`clang`。 

可以看到，对于Android来说，不同的主机，不同的指令集，不同的Android版本，都对应着一个编译器。  
了解了这么多，终于到激动人性的时刻啦，接下来，我们来编译一下前面的C++文件看看。

## 编译

通过`aarch64-linux-android21-clang++ --help`查看参数，会发现它有很多参数和选项，现在我们只想验证下我们的C++源文件有没有语法错误，所以就不管那些复杂的东西，直接一个`aarch64-linux-android21-clang++ -c math.cpp`执行编译。

命令执行完后，假如一切顺利，就会在`math.cpp`相同目录下生成`math.o`对象文件，说明我们的源码没有语法错误，可进行到下一步的链接。

不过，在此之前，先打断一下。通常我们的项目会包含很多源文件，引用一些第三方库，每次都用手工的形式编译，链接显然是低效且容易出错的。在工具已经很成熟的现在，我们应该尽量使用成熟的工具，将重心放在我们的业务逻辑上来，`CMake`就是这样的一个工具。

## CMake

CMake是个跨平台的项目构建工具。怎么理解呢？编写C++代码时，有时候需要引用其他目录的文件头，但是在编译阶段，编译器是不知道该去哪里查找文件头的，所以需要一种配置告诉编译器文件头的查找位置。再者，分布在不同目录的源码，需要根据一定的需求打包成不同的库。又或者，项目中引用了第三方库，需要在链接阶段告诉链接器从哪个位置查找库，种种这些都是需要配置的东西。

而不同的系统，不同的IDE对于上述配置的支持是不尽相同的，如Windows上的Visual Studio就是需要在项目的属性里面配置。在开发者使用同样的工具时，问题还不是很大。但是一旦涉及到多平台，多IDE的情况，协同开发就会花费大把的时间在配置上。CMake就是为了解决这些问题应运而生的。

CMake的配置信息都是写在名为`CMakeLists.txt`的文件中。如前面提到头文件引用，源码依赖，库依赖等等，只需要在`CmakeLists.txt`中写一次，就可以在Windows，MacOS，Linux平台上的主流IDE上无缝使用。如我在Windows的Visual Studio上创建了一个CMake的项目，配置好了依赖信息,传给同事。同事用MacOS开发，他可以在一点不修改的情况下，马上完成编译，打包，测试等工作。这就是CMake跨平台的威力——简洁，高效，灵活。

## 使用CMake管理项目
### 建CMake项目
我们前面已经有了`math.cpp`，又有了CMake，现在就把他们结合一下。
怎样建立一个CMake项目呢？一共分三步：

1. 建一个文件夹。示例中我们就建一个`math`的文件夹吧。
2.  在新建的文件夹里新建`CMakeLists.txt`文本文件。注意，这里的文件名不能变。 
3.  在新建的`CMakeLists.txt`文件里配置项目信息。

最简单的CMake项目信息需要包括至少三个东西  
- 支持的最低CMake版本 

```cmake
cmake_minimum_required(VERSION 3.18.1)
```
- 项目名称
```cmake
project(math)
```

- 生成物——生成物可能是可执行文件，也可能是库。因为我们要生成Android上的库，所以这里是的生成物是库。
```cmake
add_library(${PROJECT_NAME} SHARED math.cpp)
```
经过这三步，CMake项目就建成了。下一步我们来试试用CMake来编译项目。

### 编译CMake项目
在执行真正的编译前，CMake有个准备阶段，这个阶段CMake会收集必要的信息，然后生成满足条件的工程项目，然后才能执行编译。
那么什么是必要的信息呢？CMake为了尽可能降低复杂性，会自己猜测收集一些信息。

如我们在Windows上执行生成操作，CMake会默认目标平台就是Windows，默认生成VS的工程，所以在Windows上编译Windows上的库就几乎是零配置的。

1.  在`math`目录下新建一个`build`的目录，然后把工作目录切换到`build`目录。 

```shell
cd build
cmake ..
```

在命令执行之后，就能在`build`目录下找到VS的工程，可以直接使用VS打开，无错误地完成编译。当然，更快的方法还是直接使用CMake编译. 

2.  使用CMake编译 

```shell
cmake --build .
```

注意前面的`..`代表父目录，也就是`CMakeLists.txt`文件存在的`math`目录，而`.`则代表当前目录，即`build`这个目录。假如这两步都顺利执行了，我们就能在build目录下收获一个库文件。Windows平台上可能叫`math.dll`，而Linux平台上可能叫`math.so`，但是都是动态库，因为我们在`CMakelists.txt`文件里配置的就是动态库。 

从上面的流程来看，CMake的工作流程不复杂。但是我们使用的是默认配置，也就是最终生成的库只能用在编译的平台上。要使用CMake编译Android库，我们就需要在生成工程时，手动告诉CMake一些配置，而不是让CMake去猜。

## CMake的交叉编译
### 配置参数从哪来
虽然我们不知道完成交叉编译的最少配置是什么，但是我们可以猜一下。

首先要完成源码的编译，编译器和链接器少不了，前面也知道了,Android平台上有专门的编译器和链接器，所以至少有个配置应该是告诉CMake用哪一个编译器和链接器。

其次Android的系统版本和架构也是必不可少的，毕竟对于Android开发来说，这个对于Android应用都很重要。

还能想到其他参数吗，好像想不到了。不过，好消息是，Google替我们想好了，那就是直接使用`CMAKE——TOOLCHAIIIN_FILE`。这个选项是CMake 提供的，使用的时候把配置文件路径设置为它的值就可以了，CMake会通过这个路径查找到目标文件，使用目标文件里面的配置代替它的自己靠猜的参数。而这个配置文件，就是刚才提到过的两个重要文件夹之一的`build/camke`,我们的配置文件就是该文件夹下面的`android.toolchain.cmake`。

### Google的CMake配置文件

`android.toolchain.cmake`扮演了一个包装器的作用，它会利用提供给它的参数，和默认的配置，共同完成CMake的配置工作。其实这个文件还是个很好的CMake学习资料，可以学到很多CMake的技巧。现在，我们先不学CMake相关的，先来看看我们可用的参数有哪些。在文件的开头，Google就把可配置的参数都列举出来了

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
这些参数其实不是CMake的参数，在配置文件被执行的过程中，这些参数会被转换成真正的CMake参数。我们可以通过指定这些参数的值，让CMake完成不同的构建需求。假如都不指定，则会使用默认值，不同的NDK版本，默认值可能会不一样。

我们来着重看看最关键的`ANDROID_ABI`和`ANDROID_PLATFORM`。前面这个是指当前构建的包运行的CPU指令集是哪一个，可选的值有`arneabi-v7a`，`arn64-v8a`，`x86`，`x86_64`，`mips`，`mips64`。后一个则是指构建包的Android版本。它的值有两种形式，一种就是直接`android-[version]`的形式`[version]`在使用时替换成具体的系统版本，如`android-23`，代表最低支持的系统版本是Android 23。另一种形式是字符串`latest`。这个值就如这个单词的意思一样，用最新的。

那么我们怎么知道哪个参数可以取哪些值呢，有个简单方法：先在文件头确定要查看的参数，然后全局搜索，看`set`和`if`相关的语句就能确定它支持的参数形式了。

### 使用配置文件完成交叉编译
说了那么一大堆，回到最开始的例子上来。现在我们有了`CMakelists.txt`，还有了`math.cpp`，又找到了针对Android的配置文件`android.toolchin.cmake`。那么怎样才能把三者结合起来呢，这就不得不提到CMake的参数配置了。

在前面，我们直接使用

```shell
cmake ..
```
就完成了工程文件的生成配置，但是其实它是可以传递参数的。_**CMake的参数都是以`-D`开头，用空白符分割的键值对。**而CMake缺省的参数都是以`CMAKE`为开头的，所以大部分情况下参数的形式都是`-DCMAKE_XXX`这种。如给CMake传递`toolchain`文件的形式就是
```shell
cmake -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake
```
这个参数的意思就是告诉CMake，使用`=`后面指定的文件来配置CMake的参数。
然而，完成交叉编译，我们还少一个选项——`-G`。这个选项是交叉编译必需的。因为交叉编译CMake不知道该生成什么形式的工程，所以需要使用这个选项指定生成工程的类型。一种是传统形式的Make工程，指定形式是

```shell
cmake -G "Unix Makefiles"
```
可以看出，这种形式是基于Unix平台下的Make工程的，它使用`make`作为构建工具，所以指定这种形式以后，还需要指定`make`的路径，工程才能顺利完成编译。而另一种Google推荐的方式是`Ninja`，这种方式更简单，因为不需要单独指定`Ninja`的路径，它默认就随CMake安装在同一个目录下，所以可以减少一个传参。`Ninja`也是一种构建工具，但是专注速度，所以我们这一次就使用`Ninja`。它的指定方式是这样的
```shell
cmake -G Ninja
```
结合以上两个参数，就可以得到最终的编译命令

```shell
cmake -GNinja -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake ..
```
生成工程后再执行编译
```shell
cmake --build .
```
我们就得到了最终能运行在Android上的动态库了。用我这个NDK版本编译出来的动态库支持的Android版本是21,指令集是armeabi-v7a。当然根据前面的描述我们可以像前面传递`toolchain`文件一下传递期望的参数，如以最新版的Android版本构建`x86`的库，就可以这样写
```shell
cmake -GNinja -DCMAKE_TOOLCHAIN_FILE=/home/Andy/Android/Sdk/ndk/21.4.7075529/build/cmake/android.toolchain.cmake -DANDROID_PLATFORM=latest -DANDROID_ABI=x86 ..
```
这就给我们个思路，假如有些第三方库没有提供编译指南，但是是用CMake管理的，我们就可以直接套用上面的公式来编译这个第三方库。
## JNI
前面在CMake的帮助下，我们已经得到了`libmath.so`动态库,但是这个库还是不能被Android应用直接使用，因为Android应用是用Java（Kotlin）语言开发的，而它们都是JVM语言，代码都是跑在JVM上的。要想使用这个库，还需要想办法让库加载到JVM中，然后才有可能访问得到。它碰巧的是，JVM还真有这个能力，它就是JNI。
### JNI基本思想
JNI能提供Java到C/C++的双向访问，也就是可以在Java代码里访问C/C++的方法或者数据，反过来也一样支持，这过程中JVM功不可没。所以要理解JNI技术，需要我们以JVM的角度思考问题。
JVM好比一个货物集散中心，无论是去哪个地方的货物都需要先来到这个集散中心，再通过它把货物分发到目的地。这里的货物就可以是Java方法或者C/C++函数。但是和普通的快递不一样的是，这里的货物不知道自己的目的地是哪里，需要集散中心自己去找。那么找的依据从哪里来呢，也就是怎样保证集散中心查找结果的唯一性呢，最简单的方法当然就是货物自己标识自己，并且保证它的唯一性。
显然对于Java来说，这个问题很好解决。Java有着层层保证唯一性的机制。

1. 包名可以保证类名的唯一性；
2. 类名可以保证同一包名下类的唯一性；
3. 同一个类下可以用方法名保证唯一性；
4. 方法发生重载的时候可以用参数类型和个数确定类的唯一性。

而对于C/C++来说，没有包名和类名，那么用方法名和方法参数可以确定唯一性吗？答案是可以，只要我们把包名和类名作为一种限定条件。

而添加限定条件的方式有两种，一种就是简单粗暴，直接把包名类名作为函数名的一部分，这样JVM也不用看其他的东西，直接粗暴地将包名，类名，函数名和参数这些对应起来就能确定对端对应的方法了。这种方法叫做静态注册。其实这和Android里面的广播特别像：广播的静态注册就是直接粗暴地在`AndroidManifest`文件中写死了，不用在代码里配置，一写了就生效。对应于静态注册，肯定还有个动态注册的方法。动态注册就是用写代码的方式告诉JVM函数间的对应关系，而不是让它在函数调用时再去查找。显然这种方式的优势就是调用速度更快一点，毕竟我们只需要一次注册，就可以在后续调用中直接访问到对端，不再需要查找操作。但是同样和Android中广播的动态注册一样，动态注册要繁琐得多，而且动态注册还要注意把握好注册时机，不然容易造成调用失败。我们继续以前面的`libmath.so`为例讲解。

### Java使用本地库
Java端访问C/C++函数很简单，一共分三步：

1.  Java调用`System.loadLibrary()`方法载入库 

```java
System.loadlibrary("math.so");
```

这里有个值得注意的地方，CMake生成的动态库是`libmath.so`，但是这里只写了`math.so`，也就是说不需要传递`lib`这个前缀。这一步执行完后，JVM就知道有个`plus`函数了。 

2.  Java声明一个和C++函数对应的`native`方法。这里对应指的是参数列表和返回值要保持一致，方法名则可以不一致。 

```java
public native int nativePlus(int left,int right);
```

通常，习惯将`native`方法添加`native`的前缀。 

3.  在需要的地方直接调用这个`native`方法。调用方法和普通的Java方法是一致的，传递匹配的参数，用匹配的类型接收返回值。 

把这几布融合到一个类里面就是这样

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

### C/C++端引入JNI
JNI其实对于C/C++来说是一层适配层，在这一层主要做函数转换的工作，不做具体的功能实现，所以，通常来说我们会新建一个源文件，用来专门处理JNI层的问题，而JNI层最主要的问题当然就是前面提到的方法注册问题了。

#### 静态注册
静态注册的基本思路就是根据现有的Java `native`方法写一个与之对应的C/C++函数签名，具体来说分四步。
1. 先写出和Java `native`函数一模一样的函数签名


```c
int nativePlus(int left,int right)
```

2. 在函数名前面添加包名和类名。因为包名在Java中是用`.`分割的，而C/C++中点通常是用作函数调用，为了避免编译错误，需要把`.`替换成`_`。
```c
hongui_me_MainActivity_nativePlus(int left,int right)
```

3. 转换函数参数。前面提到过所有的操作都是基于JVM的，在Java中，这些是自然而然的，但是在C/C++中就没有JVM环境，提供JVM环境的形式就只能是添加参数。

为了达到这个目的，任何JNI的函数都要在参数列表开头添加两个参数。而Java里面的最小环境是线程，所以第一个参数就是代表调用这个函数时，调用方的线程环境对象`JNIEnv`，这个对象是C/C++访问Java的唯一通道。第二个则是调用对象。因为Java中不能直接调用方法，需要通过类名或者某个类来调用方法，第二个参数就代表那个对象或者那个类，它的类型是`jobjet`。从第三个参数开始，参数列表就和Java端一一对应了，但是也只是对应，毕竟有些类型在C/C++端是没有的，这就是JNI中的类型系统了，对于我们当前的例子来说Java里面的`int`值对应着JNI里面的`jint`,所以后两个参数都是`jint`类型。这一步至关重要，任何一个参数转换失败都可能造成程序崩溃。
```c
hongui_me_MainActivity_nativePlus(
        JNIEnv* env,
        jobject /* this */,
        jint left,
        jint right)
```

4. 添加必要前缀。这一步会很容易被忽略，因为这一部分不是那么自然而然。首先我们的函数名还得加一个前缀`Java`,现在的函数名变成了这样`Java_hongui_me_MainActivity_nativePlus`。其次在返回值两头需要添加`JNIEXPORT`和`JNICALL`，这里返回值是`jint`，所以添加完这两个宏之后是这样`JNIEXPORT jint JNICALL`。最后还要在最开头添加`extern "C"`的兼容指令。至于为啥要添加这一步，感兴趣的读者可以去详细了解，简单概括就是这是JNI的规范。

经过这四步，最终静态方法找函数的C/C++函数签名变成了这样

```c
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
注意到，这里我把前面的`math.cpp`改成了`math.h`，并在JNI适配文件（文件名是`native_jni.cpp`）中调用了这个函数。所以现在有两个源文件了，需要更新一下`CMakeList.txt`。
```cmake
cmake_minimum_required(VERSION 3.18。1)

project(math)

add_library(${PROJECT_NAME} SHARED native_jni.cpp)
```
可以看到这里我们只把最后一行的文件名改了，因为`CMakeLists.txt`当前所在的目录也是`include`的查找目录，所以不需要给它单独设置值，假如需要添加其他位置的头文件则可以使用`include_directories(dir)`添加。
现在使用CMake重新编译，生成动态库，这次Java就能直接不报错运行了。
#### 动态注册
前面提到过动态注册需要注意注册时机，那么什么算是好时机呢？在前面Java使用本地库这一节，我们知道，要想使用库，必须先载入，载入成功后就可以调用JNI方法了。那么动态注册必然要发生在载入之后，使用之前。JNI很人性化的想到了这一点，在库载入完成以后会马上调用`jint JNI_OnLoad(JavaVM *vm, void *reserved)`这个函数，这个方法还提供了一个关键的`JavaVM`对象，简直就是动态注册的最佳入口了。确定了注册时机，现在我们来实操一下。_**注意：动态注册和静态注册都是C/C++端实现JNI函数的一种方式，同一个函数一般只采用一种注册方式。**_所以，接下来的步骤是和静态注册平行的，并不是先后关系。

动态注册分六步
1. 新建`native_jni.cpp`文件，添加`JNI_OnLoad()`函数的实现。

```c
extern "C" JNIEXPORT jint JNICALL
JNI_OnLoad(JavaVM *vm, void *reserved) {

   return JNI_VERSION_1_6;
}
```

这就是这个函数的标准形式和实现，前面那一串都是JNI函数的标准形式，关键点在于函数名和参数以及返回值。要想这个函数在库载入后自动调用，函数名必须是这个，而且参数形式也不能变，并且用最后的返回值告诉JVM当前JNI的版本。也就是说，这些都是模板，直接搬就行。


2. 得到`JNIEnv`对象

前面提到过，所有的JNI相关的操作都是通过`JNIEnv`对象完成的，但是现在我们只有个`JavaVM`对象，显然秘诀就在`JavaVM`身上。  
通过它的`GetEnv`方法就可以得到`JNIEnv`对象
```c
JNIEnv *env = nullptr;
vm->GetEnv(env, JNI_VERSION_1_6);
```
3. 找到目标类

前面说过，动态注册和静态注册都是要有包名和类名最限定的，只是使用方式不一样而已。所以动态注册我们也还是要使用到包名和类名，不过这次的形式又不一样了。静态注册包名类名用`_`代替`.`，这一次要用`/`代替`.`。所以我们最终的类形式是`hongui/me/MainActivity`。这是一个字符串形式，怎样将它转换成JNI中的`jclass`类型呢，这就该第二步的`JNIEnv`出场了。

```c
jclass cls=env->FindClass("hongui/me/MainActivity");
```
这个`cls`对象就和Java里面那个`MainActivity`是一一对应的了。有了类对象下一步当然就是方法了。

4. 生成JNI函数对象数组。

因为动态注册可以同时注册一个类的多个方法，所以注册参数是数组形式的，而数组的类型是`JNINativeMethod`。这个类型的作用就是把Java端的`native`方法和JNI方法联系在一起，怎么做的呢，看它结构。

```c
typedef struct {
const char* name;
const char* signature;
void* fnPtr;
} JNINativeMethod;
```

- `name`对应Java端那个`native`的方法名，所以这个值应该是`nativePlus`。
- `signature`对应着这个`native`方法的参数列表外加函数类型的签名。

什么是签名呢，就是类型简写。在Java中有八大基本类型，还有方法，对象，类。数组等，这些东西都有一套对应的字符串形式，好比是一张哈希表，键是类型的字符串表示，值是对应的Java类型。如`jint`是真正的JNI类型，它的类型签名是`I`，也就是`int`的首字母大写。

函数也有自己的类型签名`(paramType)returnType`这里的`paramType`和`returnType`都需要是JNI类型签名，类型间不需要任何分隔符。

综上，`nativePlus`的类型签名是`(II)I`。两个整型参数，返回另一个整型。



- `fnPtr`正如它名字一样，它是一个函数指针，值就是我们真正的`nativePlus`实现了（这里我们还没有实现，所以先假定是`jni_plus`）。

综上，最终函数对象数组应该是下面这样

```c
 JNINativeMethod methods[] = {
    {"nativePlus","(II)I",reinterpret_cast<void *>(jni_plus)}
 };
```
5. 注册

现在有了代表类的`jclass`对象，还有了代表方法的`JNINativeMethod`数组，还有`JNIEnv`对象，把它们结合起来就可以完成注册了

```c
env->RegisterNatives(cls,methods,sizeof(methods)/sizeof(methods[0]));
```
这里第三个参数是代表方法的个数，我们使用了`sizeof`操作法得出了所有的`methods`的大小，再用`sizeof`得出第一个元素的大小，就可以得到`methods`的个数。当然，这里直接手动填入1也是可以的。

6. 实现JNI函数
在第4步，我们用了个`jni_plus`来代表`nativePlus`的本地实现，但是这个函数实际上还没有创建，我们需要在源文件中定义。现在这个函数名就可以随便起了，不用像静态注册那样那么长还不能随便命名，只要保持最终的函数名和注册时用的那个名字一致就可以了。但是这里还是要加上`extern "C"`的前缀，避免编译器对函数名进行特殊处理。参数列表和静态注册完全一致。所以，我们最终的函数实现如下。

```c
#include "math.h"

extern "C" jint jni_plus(
        JNIEnv* env,
        jobject /* this */,
        jint left,
        jint right){
           return plus(left,right);
        }
```
好了，动态注册的实现形式也完成了，CMake编译后你会发现结果和静态注册完全一致。所以这两种注册方式完全取决于个人喜好和需求，当需要频繁调用`native`方法时，我觉得动态注册是有优势的，但是假如调用次数很少，完全可以直接用静态注册，查找消耗完全可以忽略不记。
## One more thing
前面我提到CMake是管理C/C++项目的高手，但是对于Android开发来说，Gradle才是YYDS。这一点Google也意识到了，所以gradle的插件上直接提供了CMake和Gradle无缝衔接的丝滑配置。在`android`这个构建块下，可以直接配置`CMakeLists.txt`的路径和版本信息。

```groovy
externalNativeBuild {
        cmake {
            path file('src/main/cpp/CMakeLists.txt')
            version '3.20.5'
        }
    }
```
这样，后面无论是修改了C/C++代码，还是修改了Java代码，都可以直接点击运行，gradle会帮助我们编译好相应的库并拷贝到最终目录里，完全不再需要我们手动编译和拷贝库文件了。当然假如你对它的默认行为还不满意，还可以通过`defaultConfig`配置默认行为，它的大概配置可以是这样
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
这里`cppFlags`是指定C++相关参数的，对应的还有个`cFlags`用来指定C相关参数。`arguments`则是指定CMake的编译参数，最后一个就是我们熟悉的库最终要编译生成几个架构包了，我们这里只是生成两个。

有了这些配置，Android Studio开发NDK完全就像开发Java一样，都有智能提示，都可以即时编译，即时运行，纵享丝滑。

## 总结
NDK开发其实应该分为两部分，C++开发和JNI开发。  

C++开发和PC上的C++开发完全一致，可以使用标准库，可以引用第三方库，随着项目规模的扩大，引入了CMake来管理项目，这对于跨平台项目来说优势明显，还可以无缝衔接到Gradle中。  

而JNI开发则更多的是关注C/C++端和Java端的对应关系，每一个Java端的`native`方法都要有一个对应的C/C++函数与之对应，JNI提供静态注册和动态注册两种方式来完成这一工作，但其核心都是利用包名，类名，函数名，参数列表来确定唯一性。静态注册将包名，类名体现在函数名上，动态注册则是使用类对象，本地方法对象，`JNIENV`的注册方法来实现唯一性。  
NDK则是后面的大BOSS，它提供编译器，链接器等工具完成交叉编译，还有一些系统自带的库，如`log`,`z`,`opengl`等等供我们直接使用。