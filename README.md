ndk-patch
=========
The standard ndk dose not contains some useful libraries such binder,
cutils, utils, android_rumtime and so on. This patch do the job.

arch-x86/lib 
=========
from genymotion_vbox86p Samsung Galaxy Note 2 - 4.2.2 - API 17 - 720x1280 


Android.mk可能如下
=========
LOCAL_PATH := $(call my-dir)

#CLEAR_VARS                                             清除LOCAL_xxx变量
include $(CLEAR_VARS)

##LOCAL_MODULE    := inject


#LOCAL_ARM_MODE := arm  
#6 需要链接的系统默认库
#LOCAL_LDLIBS    += -lm -llog
##LOCAL_SRC_FILES := ffmpeg-build/$(TARGET_ARCH_ABI)/libffmpeg.so
##LOCAL_SRC_FILES  :=  \
        injector.c \
    ##    shellcode.s

LOCAL_MODULE    := rservice
LOCAL_SRC_FILES := rservice.cpp

### LOCAL_LDLIBS 可能也是有顺序的（添加没有自动链接到你的本地代码中模块）
#如使用android_runtime，没有这个会报如下错误（如果在eclipse里则在Problems窗口里报）：
#Description	Resource	Path	Location	Type
#undefined reference to 'android::AndroidRuntime::mJavaVM'	AInject		line 112, external location: F:\Android\android-ndk-r10\toolchains\arm-linux-androideabi-4.6\prebuilt\windows\arm-linux-androideabi\bin\ld.exe: .\obj\local\armeabi\objs\rservice\rservice.o: in function hook_entry:jni\rservice.cpp	C/C++ Problem
#模块诸如（前缀-l，链接模块）：-llog -lcutils -lutils -landroid_runtime -lnativehelper -lstdc++ -lz -lc -lstdc++ -lm
LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib -llog  -landroid_runtime

include $(BUILD_SHARED_LIBRARY)

##include $(LOCAL_PATH)/prebuilt/Android.mk

=========

#######原文：android-ndk-r5\docs\STABLE-APIS.html#############
一些“API的级别”定义。每个级别的AP对应一个给定的Android版本，目前支持：

    android-3 -> 官方Android 1.5  系统映像     （C、C++、Math、Log、Zlib）
    android-4 -> 官方Android 1.6  系统映像     （OpenGL ES 1.x）
    android-5 -> 官方Android 2.0  系统映像     （OpenGL ES 2.0）
    android-6 -> 官方Android 2.0.1 系统映像
    android-7 -> 官方Android 2.1  系统映像 
    android-8 -> 官方Android 2.2  系统映像     （jnigraphics）
    android-9 -> 官方Android 2.3  系统映像     （OpenSL ES、android）

请注意，android-6和android-7和android-5相同，即他们的本地API是完全一样的！

什么级别有什么头文件，在下面路径下可以找到：
   $NDK/platforms/android-<level>/arch-arm/usr/include

二、Android-3 Stable Native APIs:

下面列出的所有API是用于开发NDK代码，可用于运行在Android 1.5系统的及以后版本.

（1）C库：

C库的头文件，在Android 1.5中定义的可用通过他们标准名称找到(/<stdlib.h>，<stdio.h>中，等..).
如果一个头在构建时不存在，可能是在1.5版中中没有实现

编译系统自动链接C库到您的本机模块，您不必将它添加到LOCAL_LDLIBS.

请注意，机器人C库包含的pthread（<pthread.h>）的支持， 
所以“LOCAL_LIBS：= - lpthread”是没有必要的。

（2）数学库：

<math.h>是可用的，在构建时.数学库会自动链接到您的本地代码中,
所以不需要指定LOCAL_LDLIBS := -lm

（3）C++库：

C++非常小的一部分API是支持的.对于Android 1.5,仅允许下列头文件:

   <cstddef> 
   <new> 
   <utility> 
   <stl_pair.h>

他们不可能包含所有标准的定义.
值得注意的是,在Android 1.5中，C++异常和RTTI支持是不可用。

C++支持库(-lstdc++)是自动链接到你的本地代码中，
所以不需要指定LOCAL_LDLIBS

（4）Android Log：

<android/log.h>包含可用于从您的本机代码中发送各种定义日志的信息。
请您看其内容(build/platforms/android-3/common/include/android/log.h)，其中包含如何使用它.

您应该写一个宏来包装他们,以方便使用.

如果你使用它，你的本机模块应该链接/system/lib/liblog.so：

  LOCAL_LDLIBS：= -llog

（5）Zlib压缩库：

<zlib.h>和<zconf.h>可以使用,可用于使用了zlib压缩库。

如果你使用它，你的本机模块应该链接/system/lib/libz.so：

  LOCAL_LDLIBS := -lz

（6）动态链接库：

<dlfcn.h>可以使用，dlopen()/dlsym()/dlclose()
提供的Android动态链接器功能。您将需要链接/system/lib/libdl.so：

  LOCAL_LDLIBS := -ldl



三. Android-4 Stable Native APIs：

下面列出的所有API是用于开发本机代码，可用于运行在Android 1.6系统的及以后版本.

（1）OpenGL ES 1.x的库： 

标准的OpenGL ES的头文件<GLES/gl.h>和<GLES/glext.h>，

如果您使用它们，你的本机模块应该链接到/system/lib/libGLESv1_CM.so,如：

  LOCAL_LDLIBS := - lGLESv1_CM


'1.X'号是指OpenGL ES API的两个版本1.0和1.1。 
请注意：

   - OpenGL ES 1.0是支持所有的Android设备
   - OpenGL ES 1.1完全支持相应的图形处理器特定的设备

是因为Android1.0使用相对较少的设备.

开发人员应该查询OpenGL ES版本字符串和扩展名字符串，如果知道当前设备支持哪些东西。
参见glGetString()的说明，以了解如何做到这一点：

    http://www.khronos.org/opengles/sdk/1.1/docs/man/glGetString.xml

此外，开发者必须在manifest.xml中指明<uses-feature>标签，说明应用程序使用 
哪些OpenGL ES版本。有关详细信息，请查看下面链接的信息：

 http://developer.android.com/guide/topics/manifest/uses-feature-element.html

请注意，目前，本地的头文件和库在 EGL APIs是不可用.
EGL是用于执行平面创建和翻转(而不是渲染).
相应的操作必须要在您的应用程序而不是虚拟机。



四、Android-5 Stable Native APIs：

下面列出的所有API是用于开发本机代码，可用于运行在Android 2.0系统的及以后版本.

（1）OpenGL ES 2.0库： 

标准的OpenGL ES 2.0的头文件<GLES2/gl2.h>和<GLES2/gl2ext.h>，
在本地调用中，要包含声明需要执行从本地代码调用的OpenGL ES 2.0渲染
这包括定义顶点和片段着色器的使用的GLSL的语言。

如果您使用它们，你的本机模块应该链接到/system/lib/libGLESv2.so如:

  LOCAL_LDLIBS := -lGLESv2

并非所有设备都支持OpenGL ES 2.0,开发人员应该这样查询 
实现的版本和扩展字符串,见上文第三节详情。

请注意，目前，本地的头文件和库在 EGL APIs是不可用.
EGL是用于执行平面创建和翻转(而不是渲染).
相应的操作必须要在您的应用程序而不是虚拟机。

重要注意事项： 
    Android模拟器不支持OpenGL ES 2.0的硬件.
    如果运行和测试代码，使用这个API需要真实的设备.
    (意思使用这个OPenGL的库，必须真机运行或调试)



五、Android-8 Stable Native APIs:

下面列出的所有API是用于开发本机代码，可用于运行在Android 2.2系统的及以后版本.

（1）'jnigraphics'库：

这是一个很小的库，展示一个稳定的，基于C语言的，接口，使 
本机代码安全地访问Java对象的像素缓冲区的位图.

使用它，在你的源代码中包含<android/bitmap.h>，并链接库jnigraphics：

  LOCAL_LDLIBS + = -ljnigraphics

详细信息，请阅读bitmap.h：

    build/platforms/android-8/arch-arm/usr/include/android/bitmap.h

简单地说，典型的使用应该是这样的：

    1  根据JNI位图句柄，然后使用AndroidBitmap_getInfo()来检索有关信息 
       （例如它的宽度/高度/像素格式）

    2  使用AndroidBitmap_lockPixels()来锁定像素缓冲区和指针。
       直到AndroidBitmap_unlockPixels()被调用之前可确保像素
       不被移动???

    3  修改像素缓冲区，本地代码中根据其像素格式，宽度，步幅等

    4  调用AndroidBitmap_unlockPixels()来解锁缓冲区。



六、Android-9 Stable Native APIs:

下面列出的所有API是用于开发本机代码，可用于运行在Android 2.3系统的及以后版本.

（1）OpenSL ES的音频库：

使用它，在你的源代码中包含<SLES/OpenSLES.h>和<SLES/OpenSLES_Platform.h>.
包含需要从本地端的Android执行音频输入和输出的声明.

注意：尽管目前OpenSL ES的规范使用<OpenSLES.h>，
   目前Khronos的修改了,使用该文件建议包含<SLES/OpenSLES.h>，
      以后使用这个库使用Android的方案

这个API版本也提供Android的特定扩展，详情请参看
<SLES/OpenSLES_Android.h>和<SLES/OpenSLES_AndroidConfiguration.h>

该系统库命名为“libOpenSLES.so”，它在本地代码中实现了音频功能。
在你的模块使用以下链接:

    LOCAL_LDLIBS + = - lOpenSLES


（2）Android的本地应用程序的APIs：

从API 9起，有可能完全写一个Android本机代码应用程序(即没有任何Java).
这并不意味着您的代码不运行在虚拟机,
虽然,大部分的功能在该平台将仍然需要通过JNI访问.

更多的关于这个主题的信息,
请阅读docs/NATIVE-ACTIVITY.html(TODO：写文档)

下面的头文件是新增的本地API:

  <android/native_activity.h> 活动生命周期管理（和入口点）

  <android/looper.h> 
  <android/input.h> 
  <android/keycodes.h> 
  <android/sensor.h> 直接从本地代码监听和传感器的输入事件

  <android/rect.h> 
  <android/window.h> 
  <android/native_window.h> 
  <android/native_window_jni.h>  窗口管理，包括能够锁定/解锁像素，可在缓冲区中操作.

  <android/configuration.h> 
  <android/asset_manager.h> 
  <android/storage_manager.h> 
  <android/obb.h> 
        直接(只读)访问您的.apk文件嵌入资源或 
        在不透明的二进制(OBB)文件,允许分配应用程序
        大量的数据在.apk外.
        (对于开发游戏是非常有用的,例如) 
               

相应的功能函数在"libandroid.so"库中,
API 9才能使用.要使用它,使用以下配置:

    LOCAL_LDLIBS + = - landroid