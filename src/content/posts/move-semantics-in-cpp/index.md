---
title: "C++中的移动语义"
description: "C++引以为傲的特点是零开销抽象和高性能，尤其是C++11发布的移动语义，更是让它的高性能更上一层楼"

date: 2023-04-10T13:14:00+08:00

author: "hongui"

categories:
 - C++
tags:
 - C++

cover: "cover.webp"
draft: false
---

> C++引以为傲的特点是零开销抽象和高性能，尤其是C++11发布的移动语义，更是让它的高性能更上一层楼。

C++是一门支持多种编程范式的语言，其中之一就是面向对象编程。面向对象编程的一个重要特征就是封装，即将数据和操作数据的函数组织在一起，形成一个类。类可以定义自己的构造函数、拷贝构造函数、赋值运算符和析构函数，来控制对象的创建、复制、赋值和销毁过程。

## 拷贝语义
在C++11之前，类的拷贝构造函数和赋值运算符都是基于拷贝语义（copy semantics）的，即在复制或赋值一个对象时，会创建一个新的对象，并将原对象的所有成员变量逐一复制到新对象中，它可以保证对象之间互不影响，符合直觉和逻辑。这样做有时候是必要和合理的，比如当我们想要保留原对象的状态，或者当原对象和新对象有不同的生命周期时。但是，在某些情况下，我们并不需要保留原对象的状态，或者原对象本身就是一个临时的、即将被销毁的对象。比如：

```cpp
std::string s1 = "Hello";
std::string s2 = s1 + " World"; // s1 + " World" 是一个临时字符串
```

在这个例子中，我们把两个字符串相加得到一个新的字符串，并赋值给s2。按照传统的拷贝语义，这个过程会涉及三次拷贝：

+ 首先，在运算符+中，会创建一个临时字符串，并把s1和" World"复制到其中；
+ 然后，在赋值运算符=中，会创建s2，并把临时字符串复制到其中；
+ 最后，在表达式结束后，会销毁临时字符串。

可以看到，在这个过程中，有两次拷贝其实是没有必要的：第一次是把s1和" World"复制到临时字符串中；第二次是把临时字符串复制到s2中。因为在这两次拷贝之后，原来的数据就没有用处了：s1和" World"并没有改变；临时字符串马上就要被销毁。如果我们能够直接把原来的数据转移到目标对象中，而不需要创建新的副本，那么就可以节省内存空间和时间开销。这就是移动语义要做的事情。

## 移动语义
问题是找到了，在拷贝语义下，虽然解决了对象共存的问题，却引入了内存浪费的问题，所以C++11引入了移动语义（move semantics），它的主要目标就是实现最少的内存浪费。即在复制或赋值一个对象时，可以选择将原对象的资源“移动”到新对象中，而不是进行逐一复制。这样做有两个优点： 

+ 提高效率，因为移动操作通常只涉及简单地交换指针或句柄等轻量级的操作；
+  符合直觉，因为在某些情况下（例如临时变量、返回值等），我们确实只关心新对象的状态，而不在乎原对象是否被修改或销毁。

当然要实现移动语义，就不得不引入新的工具。首先需要一个新的类型来表示直接量和临时值，即右值引用（rvalue reference），而直接量和临时值统称为右值。另一个工具是针对自定义类型的，为了使自定义类也能实现移动语义，C++新增了移动构造函数（move constructor）和移动赋值运算符（move assignment operator）。它们和拷贝构造函数和拷贝复制运算符的差别就在于前者使用右值引用（rvalue reference）作为参数类型。

为了说明白移动语义的这些工具，我们需要从C++11以前开始说起。

## 右值引用的由来
为了说明白移动语义的必要性，我们可以先从一个函数开始说起。

```cpp
struct Config {
	std::string api;
	std::string token;
};

void config(Config config) {
	std::cout << "Use config( api = " << config.api <<" , token = "<<config.token<<")" << std::endl;
}

int main() {
	Config c{ "api","token" };
	config(c); //复制了一个Config
	return 0;
}
```

假设我们需要一个`config`函数，它只有一个`Config`的参数，用来表示配置信息，看起来很简单吧。但是我们很容易就能发现传递给`config`函数会复制一个对象，这个对象会一直占用内存，直到出`config`的调用作用域。这种内存占用显然不合理，我们于是开始着手重构。

### 第一次重构
为了避免生成多余的对象，我们首先想到的肯定是指针，于是我们改成以下版本

```cpp
void config(Config* config)
```

这个版本很好地解决了效率的问题，但是也引入了新问题。指针虽好，但是对于函数调用者无法直观地看出该参数是否可为空，这对于api来说是不友好的。另一方面，对于函数开发者来说，也需要检测空指针的情况。所以指针的方案也不完美。

### 第二次重构
结合不生成多余对象和不能为空的需求，我们很快想到使用引用。于是有了以下版本

```cpp
void config(Config& config)
```

这个版本解决了所有问题吗？并没有。由于是左值引用，调用者每次调用函数的时候，都需要有一个变量来作为参数，不能用临时对象，甚至不能用直接量。作为一个配置api，这种限制显然也不合理。

### 第三次重构
走到这里，相信大家都觉得没所谓了，兜兜转转还是得回到最开始的那个版本，这点内存消耗，反正也不大。但是假如配置项很多，或者有些配置项占用内存很高呢，显然我们早晚还是要解决这个问题，但是似乎已经没有合适的方案了。不过当你试一试在第二版参数前面加个`const`呢，你就会发现无论你传变量，还是临时对象，甚至直接量都可以了。发生了什么魔法呢，我们暂时按下不表，我们来看一看api还有没有问题。目前，对于调用者来说，已经很明确很方便了，但是对于api的实现者呢。实现者假如需要对`Config`参数进行修改，实现者就不得不重新生成变量，在调用者处省略的内存，又被占回来了。那有没有一种方式可以同时解决修改，临时对象的问题呢，这就是右值引用。它的形式如下

```cpp
void config(Config&& config)
```

上面的例子就是从拷贝语义逐步改造成移动语义的过程，可以看到，在拷贝语义时，内存占用分成两份，我们只使用了拷贝出来的新对象，老对象虽然外部也没用了，但是还是需要等到它出作用域后才能销毁，这就是拷贝语义下的对象行为。而在移动语义下呢，函数参数可以从外部移动到函数内部，并且内部修还可以当作普通左值一样修改，非常方便。

## 左值和右值
虽然我们解释了右值引用的由来，但是还没有解释右值。为了解释右值，我们先来看看什么是值。

我们知道程序是由代码和数据组成的，它们在程序运行的时候都需要存储在内存中。为了能让开发者读取或者修改数据，需要知道数据保存在哪里，同时，为了能解释数据，我们还需要知道数据占用的大小和可取值范围，它们共同组成了类型。一旦类型和地址确定了，我们就能从内存的字符串中还原出期望的内容，可能是数字，也可能是字符串，而我们称还原出来的内容为值。更确切的，某某类型的值。

从上面的情景我们可以看出，任何值都有个地址和类型，有的值只是暂时的，很快就会被丢弃，而有的值存活得更久，可以被反复读取和修改。暂时的这种值很可能是用来初始化某个变量的，也可能是某个计算的中间值，它们的生命周期很短，要么等着复制到其他位置，要么等着被回收，我们统称这种值为右值。另一种开发者用一种叫变量的东西记住了值的地址和类型，通过变量就可以定位到那一块固定内存，从而对它进行修改或者读取，这种类型的值，我们称为左值。如开发者在源代码里面写了字符串`abc`，当程序运行起来后，`abc`肯定也被加载进了内存里，但是我们不知道`abc`被存在哪里，所以为了能后面能继续使用`abc`，我们需要用变量来记住它。加载进来的那个`abc`就是一种右值，当它被复制到变量指定的位置后，就再也没法找到它了，而开发者可以通过变量找到被初始化为`abc`的值，即左值。由此，我们可以看出右值是即将消完的值，而左值生存周期被绑定在变量上，直到变量销毁前，都能通过变量找到它，它都存在。左值和右值的明显差别是左值可直接寻址，右值是某个中间值或者直接量，等待被复制到新的变量位置或者被系统回收。

## 右值引用
在上面的例子中，有仔细的读者可能已经发现了，既然左值和右值都存在了内存里，而且都是一样的值，那为啥不直接使用右值呢。因为右值不能直接寻址，所以我们需要一种工具来操作右值，这个工具就是右值引用。右值引用延长了右值的生命周期，它把右值偷来做自己的值，所以它既有左值的特点，还避免了一次内存可以操作。

右值引用相比左值引用多了一个`&`，但是不仅仅是语法上的不同，右值引用可以接受临时值和直接量，并且在函数内部还可以修改，是提高内存效率的强力工具。

我们先来看看左值和右值的语法

```cpp
int x = 10; // x 是一个左值
int& lref = x; // lref 是一个左值引用
int&& rref1 = 20; // rref1 是一个右值引用
int&& rref2 = x + y; // rref2 是一个右值引用
int&& rref3 = x; // 错误！x 是一个左值
```

可以看到右值引用是一种新的类型，直接量，临时值都可以是右值引用的值。

对于上面的例子，下面这些都是有效的写法

```cpp
Config readConfig(){
	Config c;
    return c;
}
config({ "api","token" });
config(readConfig());
```

既然左值右值都在内存里，那么有办法让左值转化成右值呢？答案是可以，但是有副作用。当**左值转换成右值引用后，原来的变量依然有效，但是状态是不确定的**。也就是还是可以调用它的`api`，但是对调用结果不做保证。如把`std::string`对象转换为右值引用后，用这个左值对象调用`size()`方法后，可能得到的是0，也可能还是原来的大小。

这就给我们怎样使用右值引用提供了指导思路。既然右值引用可以接收匿名对象，那么我们就可以直接使用匿名对象做参数，在不得不用左值存储中间值并确定变量不需要再时候后，直接用`std::move`来将左值转换为右值。

## 自定义类添加右值引用支持
上面的实例是针对函数的，对于自定义类，C++同样提供了新工具来支持移动语义，即添加了新的移动构造函数，移动赋值函数，其实现即是在拷贝语义的基础上，将左值引用替换为右值引用。

```cpp
class MyString {
public:
	MyString() {
		std::cout << "Default Constructor" << std::endl;
	}

	MyString(const MyString& origin) :base_{ origin.base_ } {
		std::cout << "Copy Constructor" << std::endl;
	}

	MyString(MyString&& origin) :base_{ std::move(origin.base_) } {
		std::cout << "Move Constructor" << std::endl;
	}
private:
	std::string base_;
};

int main() {   
	MyString str1;  					// 输出 Default Constructor
	MyString str2{ str1 };				// 输出 Copy Constructor
	MyString str3{ std::move(str2) };	// 输出 Move Constructor
	return 0;
}
```

上例是一个实现了移动构造函数的示例，像传递函数参数一样，我们把右值引用的数据直接偷来初始化自己的成员变量。

```cpp
// 新增
// 拷贝赋值运算符
MyString& operator=(const MyString& origin) {
    base_ = origin.base_;
    std::cout << "Assign operator" << std::endl;
    return *this;
}

// 移动赋值运算符
MyString& operator=(MyString&& origin) {
    base_={ std::move(origin.base_) };
    std::cout << "Move assign operator" << std::endl;
    return *this;
}

int main() {   
	MyString str1;  					// 输出 Default Constructor
	MyString str2{ str1 };				// 输出 Copy Constructor
	MyString str3{ std::move(str2) };	// 输出 Move Constructor
    str2 = str3;						// 输出 Assign operator
	str1 = std::move(str3);				// 输出 Move assign operator
	return 0;
}
```

可以看到，移动语义并不是取代拷贝语义的工具，而是对拷贝语义的完善，开发者可以根据实际业务自行取舍。

## 总结
移动语义能提高内存使用效率，极大地增加了开发者对内存的控制能力，在C++ 的发展中越来越重要。在C++14及后面的版本中，编译器更是将返回值默认优化成移动语义的形式，足以见移动语义的成功。

右值引用给开发者提供了操作右值的能力，拷贝构造函数和拷贝移动函数给开发者提供了移动资源的能力，开发者只需要在合适的时候选择移动或者拷贝就行，而对于现有代码，也可以通过`std::move`来完成右值转换。

咱们青山不改，绿水长流，下期见！