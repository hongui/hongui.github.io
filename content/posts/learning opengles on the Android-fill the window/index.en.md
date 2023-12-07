---
title: "Android OpenGLES learning-draw a color"
description: "Android OpenGLES learning-draw a color"

date: 2023-05-09T22:14:38+08:00
lastmod: 2023-05-09T22:14:38+08:00
publishDate: 2023-05-09T22:14:38+08:00

author: hongui

categories:
 - OpenGLES
tags:
 - study
 - Android
 - OpenGLES

toc: true
draft: false
url: post/learning opengles on the android-fill the window.html
---
> We know that the screen displays content by emitting RGB lamp beads one by one, and the brightness of the lamp beads is determined by a memory area. By writing data to this memory area, we can observe the data display effect on the screen. This is a complex and flexible job. In order to facilitate this work, the pioneers developed the OpenGL standard, and our story will start from here.

### OpenGL ES
OpenGL ES is a streamlined version of OpenGL. The Android platform has provided support for OpenGL ES since its release. However, different versions support different OpenGL ES versions. The current mainstream versions are still 2.0 and 3.0. OpenGL ES is a set of APIs that provide developers with the ability to configure data, transfer data, and draw content. Its work is strictly related to drawing, so OpenGL ES alone will not cause a big obstacle to understanding. The problem lies in the configuration environment for configuring OpenGL ES. Why should we separate the OpenGL ES API and the configuration environment? Because OpenGL ES is a cross-platform API, but it needs to be bound to a specific platform, such as Android, for actual operation. The conditions required to prepare the OpenGL ES environment are different between platforms. In order to ensure the cross-platform capabilities of OpenGL ES, it is necessary to separate the configuration environment and bind it to a specific platform. On Android, this configuration environment is EGL. Clarifying the difference between the OpenGL ES API and configuring the OpenGL ES environment is not only very helpful in understanding these two key concepts, but also greatly helps in debugging the code and troubleshooting later.

### Workflow
After clarifying some basic concepts, our next most important task is to clarify the workflow of OpenGL ES. Many tutorials start by listing a lot of nouns or giving examples directly, which I think is inappropriate. Only by becoming familiar with the workflow can we know what to do when writing code and locate problems faster and more accurately during debugging.
#### Prepare environment
OpenGL ES is composed of a series of APIs, but it does not mean that these APIs can be called at any time. Instead, it requires some settings in the running environment, which is the preparation environment. Preparing the environment usually involves doing some video memory allocation and window configuration work, which is very tedious but essential.
#### Prepare shader
Shaders are very important, but for beginners, you don’t need to pay too much attention to them. Many effects can be directly found in ready-made codes online, but how to assemble these codes into a complete runnable program is not necessarily known. . We just need to be clear that shaders are an important part of OpenGL ES development, and this is where the magic happens.
#### Prepare program
Although shaders are important, they cannot run independently and need to be managed by a program. The program in question is an OpenGL ES object that is responsible for putting the shaders together. This object is required before running most OpenGL ES APIs.
#### rendering
The rendering process is actually preparing data. We need to assign some data used in the shader, and then call the drawing API to complete the final drawing work. The GPU will pass the data to the shader, and the shader will go through the pipeline to convert the data into final display data and store it in the video memory.
#### Display
Rendering does not mean that the data is displayed, but that the data is calculated. To see the calculated data on the screen, you may need to call a function in the OpenGL ES environment configuration tool, such as exchanging buffers or switching display objects.
#### Clean up
Like memory, we will also apply for some resources when using the OpenGL ES API. After rendering, we should actively release the resources for subsequent program use. Many times when our normal application for resources fails, it may be because the previous resources have not been released.

The above is the general process for developing OpenGL ES applications. Since OpenGL ES development is difficult to debug, the most effective way to locate problems when discovering them is to determine the error link and then deal with it in a targeted manner. So it’s important to be familiar with the process.

### Get started with examples
Since there are many concepts related to OpenGLES, in order to minimize the interference of related concepts, this article intends to focus on only the first step in the above process. At the same time, use the knowledge points involved to implement a minimal example-dye the window red.

Let’s start with the first concept—EGL.

### EGL
OpenGL ES is only an abstraction of drawing and does not provide an abstraction of the running environment. If you want to apply for video memory, you need to specify where the video is stored. After the image is calculated, you also need to specify where it will be displayed. EGL is a collection of these environmental abstractions. In order to explain the relevant concepts in a popular way, we can play a role-playing role-what should we do if we are asked to design relevant standards.

The first thing that is easy to think of is that we need a monitor, because OpenGL ES will eventually generate a set of color data. If we want to see these colors, we definitely need a monitor to display these color data. At the same time, we know that monitors also have many specifications and features. In order to be compatible with various low-end to high-end monitors, it is definitely necessary to make a layer of abstraction for it and provide some methods for setting properties. This is `EGLDisplay`'s task.

After determining the display, we will find that we can only select the entire display or not use it every time. When we actually use it, there must be situations where only one area is displayed, or multiple areas are displayed at the same time. In order to meet this usage scenario , it is necessary to further divide the display so that it can support the simultaneous operation of multiple areas, and the person responsible for this layer of abstraction task is `EGLSurface` .

Since they all support multiple areas, it must be possible to configure the areas. Maybe on the same monitor, one area only needs to display black and white pixels, and another area needs to display high-definition pictures. In order to make these configurations effective and independent of each other, An abstraction is definitely needed, which must be able to save the display configuration and isolate the OpenGL ES environment so that API calls to OpenGL ES in one area will not affect another area. This is `EGLContext` .

The above are the three core concepts of EGL, which are the abstraction of the display, display area, and display configuration.

The above concepts are all scattered, and we definitely need to connect all the parts together in actual work, so it is necessary to make a summary of their work processes. First we need to obtain a `EGLDisplay` to determine the final display device, and then configure a display area `EGLSurface` according to the configuration supported by the display device. Finally, use `EGLContext` to connect `EGLDisplay` and `EGLSurface` . Once the association is successful, it means that the OpenGL ES environment is ready. The next step is to create a shader, create a shader program, and prepare for drawing.

After sorting out the process, let's take a look at how to write the code. In order to minimize the barriers to understanding, this article will use the Java-side interface as an example.

### Prepare EGLDisplay
Learning any new skill requires an entrance, and the common entrances for OpenGLES and EGL are `EGLDisplay` . So the first step is to get a `EGLDisplay` object. We cannot create this object directly, but need to obtain an object through the `eglGetDisplay` method. This object is very important and is the first parameter of almost all subsequent EGL-related APIs. Therefore, it usually needs to be cached for subsequent use. Although the `EGLDisplay` object already exists, it cannot be used directly. Need to call `eglInitialize` for initialization. This phenomenon is also very common in many SDKs. After obtaining the object, it needs to be initialized to ensure that the internal state is restored to the initial state.
### Get configuration

After successfully calling the `eglInitialize` method, the `EGLDisplay` object is ready and the display area can be configured. But we don’t know which configuration information is valid and which configuration information is supported, because different hardware supports different features. If we directly hard-code the configuration regardless of the hardware features, the code may fail to run on a certain device. This is not What we want to see. Therefore, in order for the configuration to be valid on all devices, the effective way is not for us to specify the configuration, but for us to actively query the hardware to see if it is the configuration we want, that is, let the `EGLDisplay` object tell us.

`EGLDisplay` provides two methods to query the configuration supported by the hardware. One is to directly obtain all configuration information supported by the device `eglGetConfigs` , and the other is for the developer to list the desired configuration. Then actively query whether the device supports the listed configurations `eglChooseConfig` . Developers can choose any method to determine the configuration items of the display area. If the method call is successful, it is equivalent to determining the configuration items of the display area, and we can use these configuration items to configure `EGLSurface` .

### Configure display area
`Surface` is used on the Android platform to represent the display area, but usually we do not deal directly with `Surface` , but use `SurfaceView` . However, there are limitations to using `SurfaceView` . `Surface` only valid after the `surfaceCreated` and before `surfaceDestroyed` of the `SurfaceHolder` in the  `SurafceView`.That is to say, the operation of configuring the display area can only be performed after receiving the `surfaceCreated` callback.

To configure the display area, you need to use the `eglCreateWindowSurface` method. The first two parameters are the objects we obtained in the above two steps. The third parameter is a `Surface` related parameter, which can be `Surface` , `SurfaceHolder` . In addition, you can also use the fourth parameter to pass some configuration information about `Surface` . After the function call is successful, we obtain a `EGLSurface` object.

### Connect them together
So far, the `EGLDisplay` object and the `EGLSurface` object are still independent. The latter only obtains some configuration information from the former, and has no other connection. In order to associate the two together, we need to borrow the `EGLContext` object. Similarly, creating a `EGLContext` object requires passing the `eglCreateContext` function. The first two parameters are the `EGLDisplay` and `EGLConfig` obtained in the previous step. , the special one is the third parameter. The third parameter is `EGLContext` , usually `EGL_NO_CONTEXT` is passed, which means creating an independent `EGLContext` object. Another situation is that when two rendering environments want to share resources, `EGL_NO_CONTEXT` should be passed normally when creating the first rendering environment. When creating the second rendering environment, you need to pass the `EGLContext` object is passed in, then the second rendering environment can use the textures, shaders, shader programs, and buffer class objects created in the first rendering environment, which are shared by the two rendering environments. got some data. At this point, three important objects have appeared, but they are not connected to each other yet, so the `eglMakeCurrent` function is needed to complete this work. This function will bind the `EGLContext` object to the current thread and also bind the `EGLContext` object to `EGLSurface` . After the binding is completed, the three major The objects are connected and the OpenGL ES environment is ready.

Before entering the world of OpenGL ES, we finally review the previous EGL world in code.
```kotlin
val display = EGL14.eglGetDisplay(EGL14.EGL_DEFAULT_DISPLAY)
if (EGL14.EGL_NO_DISPLAY == display) {
     log()
     return
}
val versions = IntArray(2)
var flag = EGL14.eglInitialize(display, versions, 0, versions, 1)
if (!flag) {
     log()
     return
}
Log.i(TAG, "EGL version:major = ${versions[0]}, minor = ${versions[1]}")

val attr = intArrayOf(
       EGL14.EGL_RED_SIZE, 8,
       EGL14.EGL_GREEN_SIZE, 8,
       EGL14.EGL_BLUE_SIZE, 8,
       EGL14.EGL_NONE
      )
val configs=Array<EGLConfig?>(1,{null})
val numConfig=IntArray(1)
flag = EGL14.eglChooseConfig(display, attr, 0, configs, 0, 1, numConfig, 0)
if (!flag) {
      log()
      return
}
val config=configs.first()
val eglSurface=EGL14.eglCreateWindowSurface(display,config,surface, intArrayOf(EGL14.EGL_NONE),0)
if (EGL14.EGL_NO_SURFACE == eglSurface) {
      log()
      return
}
val context=EGL14.eglCreateContext(display,config,EGL14.EGL_NO_CONTEXT, intArrayOf(EGL14.EGL_NONE),0)
if (EGL14.EGL_NO_CONTEXT == context) {
      log()
      return
}
flag = EGL14.eglMakeCurrent(display, eglSurface, eglSurface, context)
if (!flag) {
      log()
      return
}
```
### Enter the world of OpenGL ES
After a long preparation, we finally prepared the rendering environment and can use the OpenGL ES API normally. Typically, this is followed by creating shaders and shader programs. Of course, different rendering scenarios usually call different APIs. In this article, we want to dye the window red, so there is no need to create these things. We only need to call two APIs. `glClearColor` Set the clear screen color.`glClear` Set the clear screen bit.

Of course, these two functions alone are not enough. We haven't set the drawing area yet. Yes, you can specify the drawing area separately for each drawing. For example, for the first drawing, we draw in the upper left corner, and for the second drawing, we can draw in the lower right corner. You only need to specify the drawing area before drawing. The drawing area The specification will be valid until the next re-specification, and the function used is `glViewport` . The first two parameters of the function specify the starting position, and the last two parameters are the distance from the starting position.

With the help of these three functions, OpenGL ES will dye our dark black frame red. Let's take a look at the code
```kotlin
//We want to render the entire area, so the start point is the top left corner and the end point is the width and height of the view
GLES20.glViewport(0,0,width,height)
//Colour range is 0-1
GLES20.glClearColor(1f,0f,0f,1f)
GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT)

```
### Display
We have dyed the window red in the previous section, but after running the application, we will find that the display is still black. That is because we forgot the final screen operation. Because after entering the OpenGL ES world, the logical thing to do is to call the OpenGL ES API. However, the fact is to always remember that the OpenGL ES API is only responsible for drawing. For display-related issues, you have to contact EGL. After OpenGL ES drawing is completed, you need to use `eglSwapBuffers` to complete the screen operation.

### Summary
This article is the first in the OpenGL ES series. It focuses on my general understanding of EGL and OpenGL ES. The expression may not be so rigorous. It is intended to help readers build a channel to enter this field and understand some main concepts. Basic impression, we will go into each link one by one in the later stage, hoping to have the effect of attracting new ideas.

After reading this article, readers should have a simple impression of the process of developing OpenGL ES applications: EGL environment preparation, shaders, shader programs, rendering, screen loading, and cleanup. Of course, this article only focuses on the preparation of the EGL environment.

Regarding EGL environment preparation, we have three objects, which are the display, display area, and display context, corresponding to `EGLDisplay` , `EGLSurface` , and `EGLContext` . Environment preparation mainly starts from `EGLDisplay` , obtains and configures these three objects, and finally uses `eglMakeCurrent` to associate them. Of course, after using the OpenGL ES API to complete rendering, remember to use `eglSwapBuffers`` to complete the screen operation.

Please refer to [here](https://github.com/hongui/OpenGLES) for the source code.
