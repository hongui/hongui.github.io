
---
title: "CMake Personal Understanding and Use"
description: "CMake personal understanding and use"
isCJKLanguage: false

lastmod: 2021-08-09T19:21:49+08:00
publishDate: 2021-08-09T19:21:49+08:00

author: hongui

categories.
 - C/C++
tags.
 - Android (operating system)
 - JNI
 - C/C++
 - CMake

toc: true
draft: false
url: post/CMake Personal Understanding and Use.html
---
# Preface
CMake is a build tool, through which you can easily create cross-platform projects. It is usually used to build a project in two steps: generating a project file from the source code, and building the target product (which may be a dynamic library, a static library, or an executable program) from the project file. One of the main advantages of using CMake is that in the multi-platform or multi-people collaborative projects, developers can make their own preferences to make the choice of IDE, not subject to the influence of other people's project configuration, it is a bit like a cross-platform IDE, through which the configuration of the relevant settings, you can be in multiple platforms seamlessly, improve development efficiency.
<! ---more-->
# Simplest CMake project
## Project setup
A project managed with CMake will usually contain a `CMakeLists.txt` file in the project root directory, but of course subdirectories may also be present, which we'll talk about later. Let's start with the simplest project. Here is an example of the simplest project:
``
CMakeProject
| CMakeLists.txt
| main.cpp
``
This is the complete runnable minimal project. In order, let's look at what's in the file

`CMakeLists.txt`
``
# Set the version number
cmake_minimum_required(VERSION 3.10)
# Set the project name
project(CMakeProject)
# Setting up product and source code associations
add_executable(${CMAKE_PROJECT_NAME} main.cpp)
``

Description:
* :: Commands in CMake are not case-sensitive
* :: Remarks beginning with `#'
* :: Referencing variable syntax `${variable name}`

So there are really only three lines of valid content in the document.
1. `cmake_minimum_required(VERSION 3.10)` sets the minimum version supported by CMake. `VERSION` is the parameter name, followed by the version number. **Note that the parameter name and parameters are separated by whitespace, not commas,** or you will get an error.
2. `project(CMakeProject)` CMake string can be with or without quotes, the effect is the same, this line is configured with the project name, such as the generation of Visual Studio project name is based on this name.
3. `add_executable(${CMAKE_PROJECT_NAME} main.cpp)`
is the real management of the source code and the target product of the place, here we use the reference variable writing, and the file does not define the variable, that this variable exists in CMake, in CMake there are a lot of predefined variables, we can be directly referenced in this way, the above writing is the name of the project is set to the product of the name, of course, you can also fill in the string directly, to take another name is fine. The latter `main.cpp` is used to generate the source path of the product, which is the most flexible part of CMake. This is the most flexible part of CMake. ** The source path can be various, it can be found out, written directly, relative path, absolute path, etc. ** If you have more than one source code, you can use it. ** If you have more than one source code, you can separate them with blanks and write them in order.
In the configuration file above, we configured its source file as `main.cpp`, through which we want to generate an executable program with the same simple content:.
``
#include <iostream>
int main()
{
std::cout<<"hello CMake"<<std::endl;
return 0;
}
``
## Project compilation and execution

Now that the preparations are done, we're going to use CMake to generate the executable.

The first step is of course to install CMake la, this is the download address [!Download](https://cmake.org/download/), according to their own platform to choose to download can be installed after the completion of the need to add it to the environment variables, so that we can easily use anywhere.
After installing CMake, open the command line tool and go to the root directory of the project you just created, i.e. to the directory where `CMakeLists.txt` and `main.cpp` are stored, and prepare to generate the project in the next step.

Usually in order not to affect and pollute the current working environment, we will choose to create a new directory to store the generated project files, the following I mainly Windows platform as the main platform to explain, other platforms are basically the same.

``
mkdir build # Create a folder to store the project files;
cd build #Switch the cmake working directory.
cmake ...                    # Generate project files;
``

After the execution of these three steps, we can see in the build folder has been generated inside a Visual Studio project, we can directly use Visual Studio to open the project, according to our habits to perform compilation and debugging. Of course, if you want to generate the fastest executable, I still recommend using CMake.

To perform a build with CMake, simply build on the previous step (i.e., you've already successfully performed the three steps above) by executing another command `cmake --build . ` and you're done. Remember to include the third period, which means that CMake builds in the current working directory.
If everything went well in the above four steps, then we can see the executable file named after the first parameter of `add_executable` in the `build/debug` directory (in this case `CMakeProject.exe`), and we can execute it by double-clicking or dragging it to the command line.

## Project extensions
In the previous example, to generate the project file, we used two commands, in fact, here it can be done directly with one command - `cmake build -S . -B build`. The meaning of this command is to use the current path as the working path and the `build` directory as the build directory to generate the project file, that is, we don't need to create the `build` folder manually. The `-S` parameter configures the source path and `-B` configures the build path.

In addition, since CMake does not have a cleanup method, every time we change CMake's configuration (i.e., add or remove code from `CMakeLists.txt`) and need to regenerate the project file, we need to manually clean up the generated directory to make sure that it's empty; if we don't do this, the project may fail to be generated or the new configuration may not work. If you only change the source code, you don't need to regenerate the project, just do the fourth step.
Although the above operation is simple enough, but considering the long-term modification and validation needs, it is still too cumbersome and boring, especially to repeatedly switch the working directory, which is still rather annoying. So I recommend using batch processing to complete these operations. Combining the steps of cleaning up the generated directory and switching working directories, the final batch file may look like this
``

@echo off 
rd /s /q build
mkdir build
cd build
cmake ...
cmake --build .
cd debug
CMakeProject
cd ... /...
``
Explain in order.

The first line turns off the command line's echo, because we don't want its echo to interfere with CMake's message output in a way that causes unnecessary confusion, and also because usually we only care if it ends up doing its job rather than looking at what it's doing.

The second line uses the Windows command for deleting folders (rmdir on Linux and MacOS), /s is to configure it to clear all the contents of the folder, including subfolders, without which the command will fail, and /q is to allow the command to execute the deletion directly without requiring us to manually confirm it, which is a very important parameter, as we would need to confirm the deletion of the folders one by one, completely defeating the purpose of automation. This parameter is very important, otherwise we need to confirm the deletion one by one, which completely defeats the purpose of automation. Then the next four sentences are what we talked about above, without further ado.

Coming all the way to the penultimate sentence, here I've written the name of the executable directly (you need to replace it with your own), in order to run the executable directly after the compilation is complete, which is useful for some applications that generate files.

At the end of the execution, then cut the directory back to the root directory of the project, this is the role of the last line, because we compile again has switched the directory to the generated directory, and the compiled executable file is in the generated directory of the subdirectory, so back to the root directory, we need to fall back twice, this is to ensure that the next time we can triumph in the execution of the batch processing key.

Save the above as a file ending in bat, and then next time you can just type the bat file name at the command line to finish generating and building at once, it's just awesome.
This is all we need to know about CMake projects. Of course the actual project is far more complex than this, next I will use the pitfalls I have stepped on as a basis to increase the complexity of the project one by one, and slowly develop an understanding of CMake's workflow.

# Multi-source projects

## Personal insights ##

Before I start, let me talk about my understanding of the CMake project or `CMakeLists.txt` file. **We can't understand a configuration in isolation, we need to categorize the commands and even distill its core working pattern. I am using the compilation and linking of c++ files as a clue to sort it out. ** We all know that for a c++ source file to generate executable code, it needs three steps
* :: Preprocessor processing, copying the contents of header files to source files, macro replacement, etc;
* The compiler compiles the source files into .o object files;
* The linker takes .o files and other libraries as input and links to generate executables.

It is much easier to understand CMake along these lines. If CMake reports an error, we can use the information to find out which phase of CMake is causing the problem, and then quickly find a solution. In addition, we can also use this information to categorize the CMake configuration, my own understanding of the rough categorization is as follows.
* :: Configure CMake basic information: `cmake_minimum_required`;
* :: Source code management: `file`, `aux_source_directory`;
* :: Library-managed: `find_libraray`;
* :: Header file management: `include_directories`;
* :: For linkbase management: `link_directories`;
* :: Sub-project management: `add_subdirectory`;
* :: For generator management: `add_executable`, `add_library`;

Of course, these are only a very small part of the picture, but they provide a better direction for understanding and searching for ideas to solve the problem.
## CMake manages subdirectories

Many times we introduce third-party packages to reduce duplicate coding efforts, and usually such code we need to put in other directories, so I created a new subdirectory to simulate the stored third-party code. For this case, we have two forms of inclusion - sub-modules and sub-directories.

Let's start with the simpler subdirectories. A subdirectory means that the third-party code is treated as part of our code and is merged and compiled together, in a way that makes our project look more compact. For example, the following project structure
``
CMakeProject

| auto.bat
| CMakeLists.txt //Modify
| main.cpp //modify
| 

\---3rd //Added
        lib.h     
``

I created a new subfolder to simulate the third party code, now let's introduce it into `main.cpp`, compile it, and we'll see that an error is reported, with the message `fatal error C1083: Unable to open include file: "lib.h": No such file or directory,` which is normal. Combine this with the example I gave above. This error message is related to header files. Looking at the CMake documentation, I found out that CMake has an `include_directories` directive, which means to add the directory of the file header in order for CMake to find the header files. So I added `include_directories(3rd)` to the `CMakeLists.txt` file and ran the compile again and the project ran correctly. Take a look at the `main.cpp` at this point.
``
#include <iostream>
#include <lib.h>

int main()
{
    int a=1,b=1;
std::cout<<"hello CMake"<<std::endl;
    std::cout<<"a + b = "<<sum(a,b)<<std::endl;
return 0;
}
``
Note: There is a one-to-one correspondence between `include_directories` and `include` in the cpp, i.e., if the directory configured in `include_directories` is . (the current directory, CMake does not add the current directory to the `include` path), then the `include` of the corresponding cpp should be written in the form of `3rd/lib.h`, which simply means that `include_directories` is set to the root of the `include` directory.
The other case is submodules.
## CMake management submodule
Submodule means that the module can be compiled separately and provided separately for other libraries to use, instead of being symbiotic with the main project, which applies to the case where the coupling with the main module is not too big. In order to satisfy this condition, we modify the directory structure to the following one
``
CMakeProject

| auto.bat
| CMakeLists.txt //Modify
| main.cpp        

| 
\-3rd
        CMakeLists.txt //added
        lib.cpp //added
        lib.h //modify
``
I changed the functions in `lib.h` to declarations, and the implementation is in the `lib.cpp` file. The biggest change was the creation of a new `CMakeLists.txt` file in the `3rd` directory, which manages all the source files in the `3rd` directory in a unified way (in case there are a lot of files, this is a simulation), and the use of `add_library` to package the `3rd` directory into sub-modules.
``
project(sum)
add_library(${PROJECT_NAME} lib.cpp)
``
`add_library` can also specify the build type between the name and the source code, the default is `STATIC`, which is a static library, if you want to build a dynamic library you need to manually specify `SHARED` (`add_library(${PROJECT_NAME} SHARED lib.cpp)`).

The important changes come from `CMakeLists.txt` in the main directory.
``
# Set the version number
cmake_minimum_required(VERSION 3.10)
# Set the project name
project(CMakeProject)
# Specify 3rd as the lookup directory for include
include_directories(3rd)
# Submodules
add_subdirectory(3rd)
# Setting up product and source code associations
add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} sum)
``
Added `add_subdirectory`, which is used to compile the source code from a specified directory as a module, provided there is a `CMakeLists.txt` file in that directory. Another change is the addition of `target_link_libraries`, which is used to link submodules into the main module; without it, linking would result in an error `error LNK2019: unresolved external symbol`. The name of the module needs to be the same as the first parameter of `add_library` in the submodule.
# Cross-compilation
While the complexity of the project in the previous example was shown in the multiple directories and source code, the main complexity of the project in the process of cross-compiling with CMake is shown in the environment configuration. Although CMake can be used to cross-compile with little or no modification to `CMakeLists.txt`, newcomers are often overwhelmed by unfamiliar configurations and try to find an easy way to configure them with a single click. It is true that there is no such shortcut for CMake, but once we understand the essence of ** cross-compilation is the process of configuring property values correctly. However, once we understand that **cross-compiling is the process of correctly configuring property values**, the problem becomes clear. So, the questions above become familiar - what properties need to be configured, what are the appropriate values for those properties, how are those values passed to CMake, etc., and that's what cross-compilation is all about. As mentioned before, CMake has a lot of preset variables, and we need to find some of these preset variables, set some values, and then let CMake do its job according to these configurations, and that's what we need to do next. Below I will illustrate this process with an example of cross-compiling Android for Windows.
## Prepare yourself ##
On Windows platform, Visual Studio will be used as the compiler for C, C++ by default, which may report errors for compiling libraries for Android. So you need to use ` -G "Unix Makefiles"` to change this behavior when executing `cmake` command. But that's not enough, because CMake compilation is required to specify the compiler. The C,C++ compiler on Android is usually provided in the form of NDK, so we need to download the NDK, which provides us with two tools at the same time, one is the compiler, and the other one is `android.toolchain.cmake`, which is also the file composed of CMake commands, which specifies a lot of preset values for cross-compilation, which can greatly simplify the compilation process. This is also a CMake file, which specifies a lot of presets for cross-compilation, which can greatly reduce our work.
## Write compilation scripts
As mentioned earlier, cross-compiling is all about changing the CMake preset, and there are two ways to change this preset that we're going to use in combination. One is through the `android.toolchain.cmake` file provided by the NDK. The `android.toolchain.cmake` file sets most of the values, but it is also very flexible and has a lot of room for configuration. Therefore, depending on the user's needs, we also need to pass some values dynamically when executing CMake commands in order for CMake to do its job correctly. This is another way - options. Passing options will start with `-D` followed by some CMake predefined variable Since there are a lot of options and most of them are complex, it's better to document and modify them through script files. Below are just a few of the options that need to be specified to compile Android code on the Windows platform, and I'll go over the necessary configurations one by one.
* `-DCMAKE_SYSTEM_NAME=Android` This configuration tells CMake that it needs to generate libraries for the Android platform, i.e., perform cross-compilation.
* The `-DANDROID_ABI=x86` configuration tells CMake to generate libraries for the applicable architectural platforms. Readers familiar with Android development should not be unfamiliar with the supported values will change according to the changes in the NDK, such as the early `armeabi` has been removed in the NDK r17, now there are four mainstream `armeabi-v7a`, `arm64-v8a`, `x86`, `x86_64`. Just replace the values as needed.
* `-DANDROID_PLATFORM=android-28`, this value is not really necessary as there are preset values, but it is necessary to specify one for controllability. It is used to determine the minimum system version supported by the library.
* `-DCMAKE_TOOLCHAIN_FILE=C:/Users/Leroene/AppData/Local/Android/Sdk/ndk/21.0.6113669/build/cmake/android.toolchain.cmake`, which is the above mentioned preset file. Note that there are multiple files with this name in the NDK, and if you specify them incorrectly, you may get CMake errors, so my experience has been to change the version number (`C:/Users/Leroene/AppData/Local/Android/Sdk/ndk/21.0.6113669`) and the paths in front of it, and leave the paths behind it unchanged. The latter stays the same.
* `-DCMAKE_MAKE_PROGRAM=C:/Users/Leroene/AppData/Local/Android/Sdk/ndk/21.0.6113669/prebuilt/windows-x86_64/bin/make` The last argument specifies the path to the `make ` program path, since we are specifying the code that generates the make project and Windows usually does not have a make executable, we need to let CMake find the make file in order to complete the compilation. My experience here is also to keep the later unchanged, modify the earlier, and keep the version consistent to avoid bugs.
* `-DCMAKE_BUILD_TYPE=Release` to specify the build type, which should be common enough.

At this point all the configurations for cross compiling Android libraries for Windows have been explained. Let's have a look at its complete example
``
@echo off
rd /s /q build
mkdir build
cd build
cmake -G "Unix Makefiles" ^
-DCMAKE_TOOLCHAIN_FILE=C:/Users/Leroene/AppData/Local/Android/Sdk/ndk/21.0.6113669/build/cmake/android.toolchain.cmake ^
-DCMAKE_MAKE_PROGRAM=C:/Users/Leroene/AppData/Local/Android/Sdk/ndk/21.0.6113669/prebuilt/windows-x86_64/bin/make ^
-DANDROID_PLATFORM=android-28 ^
-DCMAKE_SYSTEM_NAME=Android ^
-DANDROID_ABI=x86 ^
-DCMAKE_BUILD_TYPE=Release ^
... /3rd
cmake --build .
``
As you can see from the above, these options are all followed by a `^` symbol, which is not part of cmake, it is just written this way for our reading convenience, this is the command line break used for batch processing on Windows platform, its function is to tell the command parser that the command is not yet finished, and continue to be parsed down the line, this function is equivalent to `\` on Linux,MacOS This function corresponds to `\` on Linux and MacOS. Now that you have this configuration, how do you use it? It's quite easy, just store these commands in the `android.bat` file, switch to the current directory in CMD, execute this file and you'll find the static library file named `libsum.a` in the `build` directory. Next, let's try to run in the emulator with this library file.
# Using CMake in an Android project
In the Android platform, CMake is also used to manage jni projects, along with Gradle to complete the build. The biggest difference between this and a normal CMake project is that we usually need to reference multiple Android related libraries such as `log`, `android`, etc. These libraries are usually provided by NDK. These libraries are usually provided by the NDK, and we can just follow the default `CMakeLists.txt` file.
## Directory structure
Next, for descriptive purposes, let's take a look at the current directory structure (to avoid confusion, only the more representative files are listed here)
``
CMakeProject
│ android.bat
│ CMakeLists.txt
│ main.cpp
│  
├óΓé¼╦£3rd
│ CMakeLists.txt
│ lib.cpp
│ lib.h
│   
└─Android
    │ build.gradle
    │   
    ├─app
    │ │ build.gradle
    │ │   
    │ ├─libs
    │ └─src         
    │ ├─main
    │ │ │ AndroidManifest.xml
    │ │ │  
    │ │ ├─cpp
    │ │ │ CMakeLists.txt
    │ │ │ native-lib.cpp
    │ │ │   
    │ │ ├─java
    │ │ └─me
    │ │ └─hongui
    │ │ └─cmakesum
    │ │ │ MainActivity.kt
    │ │ │   
    │ │ ├─jniLibs
    │ │ └─x86
    │ │ │ libsum.a
``
A new `Android` subdirectory has been created under the root of the original directory, which is an Android C++ project, so it has an extra `cpp` directory compared to other normal Android projects, and the main modifications we'll make later on happen in that directory.

The original root directory, in order not to add complexity, exists only as a function of generating static libraries, so there are no modifications compared to the example above.
## Build static libraries
First, let's go back to the root directory. Use the `android.bat` batch in the root directory to generate static libraries that are available on Android, you can also modify the value of the `-DANDROID_ABI` option in the `android.bat` file to generate static libraries for other architectures, but this needs to correspond to the directories in the `jniLibs` directory, otherwise the link may fail. For example, the `libsum.a` file I generated is for the `x86` architecture. Then you need to create a new `x86` directory in the `jniLibs` directory, and then put `libsum.a` into that directory. This concludes the build of the static library.
## Using static libraries
After putting the static libraries in place, we need to configure the `build.gradle` in the `app` directory and the `CMakeLists.txt` file in the `cpp` directory to complete the introduction of the static libraries.
### Configuring Gradle
First, let's talk about `build.gradle`, which is mainly concerned with modifying the ABI, because if you don't specify it, the ABI generated by Gradle by default may not find the corresponding static library file to link to, which may lead to linking failure. The main changes in this file are as follows
``
android {
    defaultConfig {
        externalNativeBuild {
                cmake {
                    cppFlags ""
                    abiFilters "x86"
                }
            }
    }
}
``
That is, specify the value of `abiFilters` as the same value as the static library you just built.
### Configuring CMake
The `CMakeLists.txt` file is a bit more complicated, it needs to do two jobs, find the static libraries and static libraries' header files, and link the static libraries.
#### find header file
In the second part of the article we already knew about the `include_directories` command that lets CMake find header files, just set the parameter to the `3rd` directory. It's worth noting that CMake uses the current `CMakeLists.txt` file as its working directory, so to specify to the `3rd` file we need to go all the way back in the directory to the root project, and end up with `include_directories(... /... /... /... /... /3rd)` configuration. Try to use relative paths, you can collaborate with multiple people without having to change the configuration.
#### find static library
The next step is to get CMake to find our static library. When it comes to libraries, it's all related to `add_library`, the difference is just the parameters. When adding a library using source, we need to specify the name and source location of the library, whereas to reference a third-party library, we need to specify the name and type of the library, plus an `IMPORTED` indicator parameter to tell CMake that the library is imported. So there is a configuration like `add_library(addSum STATIC IMPORTED)`.

However, here we have only told CMake the name of the library, where the library is stored is not yet known, so we need another command to tell CMake where the library is stored. When it comes to configuration parameters, it's usually the `set_target_properties` command, which can be called multiple times to set multiple configurations. `set_target_properties(addSum PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/... /jniLibs/${ANDROID_ABI}/libsum.a)`, the first parameter and the first parameter of the previous entry are one-to-one and can be taken at will. In fact, `add_library` is equivalent to generating a target product, and the first parameter is used to refer to such a product, which is why our `set_target_properties` is allowed to find the appropriate target setting properties. The second parameter is the standard way to write a configuration property, the third represents the property variable, and the fourth is the value of the property. The variable that configures the library path is `IMPORTED_LOCATION`, and the value has a pitfall here, as CMake under Android restricts the value to an absolute path, not a relative path. This is contrary to the original purpose of using CMake, fortunately, we have a few preset values that we can use, `CMAKE_CURRENT_SOURCE_DIR` is one of them, it represents the absolute path of the current `CMakeLIsts.txt` file, with this, and the directory fallback function, we can find any appropriate directory. At this point, a second problem arises, when there are multiple architectures with static libraries to configure, we introduce different directories, and there is a lot of duplication of configurations. Luckily, it helps to have `ANDROID_ABI`, which refers to a certain architecture that is currently being compiled, and as the compilation progresses, this value is set to the appropriate value, and is a one-to-one correspondence with the architecture being compiled. So, even though they are a bit strange, this gives me flexibility and simplicity.
#### Linking static libraries
Now we have the header files and the libraries, but the compilation of C++ is divided into two steps, so far, our work has only done the compilation of things, not yet involved in the linking of things, of course, compared to the previous configuration, this is much simpler, undoubtedly is in the `target_link_libraries` command to add a parameter can be, such as
``
target_link_libraries( 
                       native-lib
                       ${log-lib}
                       addSum
        )
``
Just yo note that the name corresponds one to one with the name configured during `add_library`.
### Use in source code
After a long wait, we are now able to introduce the `addsum` header file in the `native-lib.cpp` file and use the functions inside to get the job done. I intend for the function to return a string containing the result of the addition operation. The final implementation is as follows
``
#include <jni.h>
#include <string>
#include <lib.h>

extern "C" JNIEXPORT jstring JNICALL
Java_me_hongui_cmakesum_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = std::to_string(sum(1,1));
    return env->NewStringUTF(hello.c_str());
}
``
At this point, click the `run` button on the toolbar and we can finally see the results of our static library work in the Android emulator.
## Extension ##
In fact, besides the way of referencing static libraries, we can also directly reference the source code by configuring the `CMakeLists.txt` file, which allows us to customize the source code anytime, anywhere, but it also slows down the compilation speed and may increase the complexity of `CMakeLists.txt`. So I still recommend the direct static library approach.
# Summarize
CMake in fact, there are many, many commands, we are involved here is only a very small part. However, I think that understanding CMake has these contents almost on it, the subsequent need to target learning on the line. Learning a technology, we must not be greedy, greedy for details. First of all, we must grasp the main, clear vein, the details of the latter is a matter of water to the drain. For CMake, I think it is the C++ code compiled into a binary process as the main trunk is enough. Where the source code comes from, where the header files are, where the library files are, how to organize the compilation, what libraries are involved in linking, what products are generated, and some general operations to complete these tasks, copying files ah, directory information ah, etc., a collection of these operations constitute the main body of CMake. In addition, CMake is really just a build tool, it is not a compiler or linker, and some problems may involve not only cmake, but also the compiler and linker. Of course, these are all issues that you may encounter later on, when you know more about it.
