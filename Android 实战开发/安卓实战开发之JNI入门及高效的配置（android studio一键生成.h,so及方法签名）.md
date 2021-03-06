##前言
以前也讲过NDK开发，但是开始是抱着好玩的感觉去开始的，然后呢会helloWord就觉得大大的满足，现在静下来想这NDK开发到底是干什么呢？
> NDK开发，其实是为了项目需要调用底层的一些C/C++的一些东西；另外就是为了效率更加高效些但是在java与C相互调用时平白又增大了开销（其实效率不见得有所提高）,然后呢，基于安全性的考虑也是为了防止代码被反编译我们为了安全起见，使用C语言来编写这些重要的部分来增大系统的安全性，最后呢生成so库便于给人提供方便。    

好了，我们来看一下qq的结构，我们就能理解任何有效的代码混淆对于会smail语法反编译你apk是分分钟的事，即使你加壳也不能幸免高手的攻击，当然你的apk没有什么机密和交易信息就没有人去做这事了。

***分析qq的apk架构：***

1.使用ClassyShark.jar来打开qq.apk
![](http://i.imgur.com/d1kWaZk.png)

2.点开Archive我们来查看架构

![](http://i.imgur.com/wOWhbeN.png)

从上图我们可以看出qq里面是一堆的so库是吗，所以呢so库可见比代码混淆安全系数高的多。
##JNI与NDK的关系
* NDK:

NDK是一系列工具的集合。它提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。这些工具对开发者的帮助是巨大的。它集成了交叉编译器，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so。它可以自动地将so和Java应用一起打包，极大地减轻了开发人员的打包工作。

* JNI:

JavaNative Interface (JNI)标准是java平台的一部分，JNI是Java语言提供的Java和C/C++相互沟通的机制，Java可以通过JNI调用本地的C/C++代码，本地的C/C++的代码也可以调用java代码。JNI 是本地编程接口，Java和C/C++互相通过的接口。Java通过C/C++使用本地的代码的一个关键性原因在于C/C++代码的高效性。

现在明白了吧，NDK就是为我们生成了c/c++的动态链接库而已，jni呢只不过是java和c沟通而已，两者与android没有半毛钱关系，只因为安卓是java程序开发然后jni又能与c沟通，所以使“Java+C”的开发方式终于转正。
> Android是JVM架设在Linux之上的架构。所以无论如何，在Linux OS层面，都应该可以跑C/C++程序。

>Android Native C就是使用C/C++程序直接跑到Linux OS层面上的程序。与其它平台类似，只需要交叉编译后。并得到Linux OS root权限，就可以直接跑起来了。
##android studio 中简单的jni开发
## Let’s Go！！！  ##

准备工作不再需要什么cgwin来编译ndk（太特么操蛋了）,现在只需要你下载一下NDK的库就ok了，然后你也可以去离线下载[http://www.androiddevtools.cn](http://www.androiddevtools.cn)最新版，这里吐槽一下android studio对NDK的支持还有待提高。
##效果
看下今天的效果：（安卓jni获取 apk的包名及签名信息）
![这里写图片描述](http://img.blog.csdn.net/20160717004326192)

###必须的步骤
1.配置你的ndk路径(local.properties)
> ndk.dir=E\:\\Android\\sdk\\android-ndk-r11b-windows-x86_64\\android-ndk-r11b 

2.grale配置使用ndk(gradle.properties)
> android.useDeprecatedNdk=true  

3.在module下的build.gradle添加ndk以及jni生成目录

>   ndk{
            moduleName "JNI_ANDROID"
            abiFilters "armeabi", "armeabi-v7a", "x86" //输出指定三种abi体系结构下的so库，目前可有可无。
        }
   sourceSets.main{
           jniLibs.srcDirs = ['libs']
        }  


准备工作做好了开始写代码：（jni实现获取应用的包名和签名信息）
###步骤1：先写要实现本地方法的类，及加载库（JNI_ANDROID也就是ndk 里面配的moduleName）
```
package com.losileeya.getapkinfo;

/**
 * User: Losileeya (847457332@qq.com)
 * Date: 2016-07-16
 * Time: 11:09
 * 类描述：
 *
 * @version :
 */
public class JNIUtils {
    /**
     * 获取应用的签名
     * @param o
     * @return
     */
    public static native String getSignature(Object o);

    /**
     * 获取应用的包名
     * @param o
     * @return
     */
    public static native String getPackname(Object o);

    /**
     * 加载so库或jni库
     */
    static {
        System.loadLibrary("JNI_ANDROID");
    }
}
```
注意我们 的加载c方法都加了native关键字，然后要使用jni下的c/c++文件就必须使用System.loadLibrary（）。
###步骤2：使用javah命令生成.h（头文件）
> javah -jni com.losileeya.getapkinfo.JNIUtils  

执行完之后你可以在module下文件夹app\build\intermediates\classes\debug下看见生成的 .h头文件为：
> com_losileeya_getapkinfo_JNIUtils.h     

```
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_losileeya_getapkinfo_JNIUtils */

#ifndef _Included_com_losileeya_getapkinfo_JNIUtils
#define _Included_com_losileeya_getapkinfo_JNIUtils
#ifdef __cplusplus
extern "C" {
#endif
JNIEXPORT jstring JNICALL Java_com_losileeya_getapkinfo_JNIUtils_getPackname(JNIEnv *, jobject, jobject);
JNIEXPORT jstring JNICALL Java_com_losileeya_getapkinfo_JNIUtils_getSignature(JNIEnv *, jobject, jobject);
#ifdef __cplusplus
}
#endif
#endif
```

在工程的main目录下新建一个名字为jni的目录，然后将刚才的.h文件剪切过来，当然文件名字是可以修改的
###步骤3：根据.h文件生成相应的c/cpp文件
```
//
// Created by Administrator on 2016/7/16.
//
#include <stdio.h>
#include <jni.h>
#include <stdlib.h>
#include "appinfo.h"
JNIEXPORT jstring JNICALL Java_com_losileeya_getapkinfo_JNIUtils_getPackname(JNIEnv *env, jobject clazz, jobject obj)
{
jclass native_class = env->GetObjectClass(obj);
jmethodID mId = env->GetMethodID(native_class, "getPackageName", "()Ljava/lang/String;");
jstring packName = static_cast<jstring>(env->CallObjectMethod(obj, mId));
return packName;
}

JNIEXPORT jstring JNICALL Java_com_losileeya_getapkinfo_JNIUtils_getSignature(JNIEnv *env, jobject clazz, jobject obj)
{
jclass native_class = env->GetObjectClass(obj);
jmethodID pm_id = env->GetMethodID(native_class, "getPackageManager", "()Landroid/content/pm/PackageManager;");
jobject pm_obj = env->CallObjectMethod(obj, pm_id);
jclass pm_clazz = env->GetObjectClass(pm_obj);
// 得到 getPackageInfo 方法的 ID
jmethodID package_info_id = env->GetMethodID(pm_clazz, "getPackageInfo","(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");
jstring pkg_str = Java_com_losileeya_getapkinfo_JNIUtils_getPackname(env, clazz, obj);
// 获得应用包的信息
jobject pi_obj = env->CallObjectMethod(pm_obj, package_info_id, pkg_str, 64);
// 获得 PackageInfo 类
jclass pi_clazz = env->GetObjectClass(pi_obj);
// 获得签名数组属性的 ID
jfieldID signatures_fieldId = env->GetFieldID(pi_clazz, "signatures", "[Landroid/content/pm/Signature;");
jobject signatures_obj = env->GetObjectField(pi_obj, signatures_fieldId);
jobjectArray signaturesArray = (jobjectArray)signatures_obj;
jsize size = env->GetArrayLength(signaturesArray);
jobject signature_obj = env->GetObjectArrayElement(signaturesArray, 0);
jclass signature_clazz = env->GetObjectClass(signature_obj);
jmethodID string_id = env->GetMethodID(signature_clazz, "toCharsString", "()Ljava/lang/String;");
jstring str = static_cast<jstring>(env->CallObjectMethod(signature_obj, string_id));
char *c_msg = (char*)env->GetStringUTFChars(str,0);

return str;
}
```
注意：要使用前得先声明，方法名直接从h文件考过来就好了，studio目前还是很操蛋的，对于jni的支持还是不很好。
###步骤4：给项目添加Android.mk和Application.mk
此步骤显然也是不必要的，如果你需要生成so库添加一下也好，为什么不呢考过去改一下就好了，如果你不写这2文件也是没有问题的，因为debug下也是有这些so库的。
好吧，勉强看一下这2货：
####Android.mk
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := JNI_ANDROID
LOCAL_SRC_FILES =: appinfo.cpp
include $(BUILD_SHARED_LIBRARY)
```
####Application.mk
```
APP_MODULES := JNI_ANDROID
APP_ABI := all
```
##android studio下External Tools的高级配置NDK一键javah,ndk生成so库
eclipse开发ndk的时候你可能就配置过javah,所以android studio也可以配置，是不是很兴奋：
Settings--->Tools---->External Tools就可以配置我们的终端命令了，别急一个一个来：

* javah -jni 命令的配置（一键生成h文件）

![](http://i.imgur.com/4Bf05WA.png)

我们先来看参数的配置：
> 
1.Program:$JDKPath$\bin\javah.exe 这里配置的是javah.exe的路径（基本一致）

> 2.Parametes: -classpath . -jni -d $ModuleFileDir$/src/main/jni $FileClass$这里指的是定位在Module的jni文件你指定的文件执行jni指令

> 3.Working:$ModuleFileDir$\src\main\java 

* ndk-build(一键生成so库)

![](http://i.imgur.com/AaGIW9k.png)

我们同样来看参数的配置：
> 
1.Program:E:\Android\sdk\android-ndk-r11b-windows-x86_64\android-ndk-r11b\ndk-build.cmd 这里配置的是ndk下的ndk-build.cmd的路径（自己去找下）

> 2.Working:$ModuleFileDir$\src\main\ 

* javap-s（此命令用于c掉java方法时方法的签名）

![](http://i.imgur.com/5p4qEW8.png)

我们同样来看参数的配置：
> 
1.Program:$JDKPath$\bin\javap.exe 这里配置的是javap.exe的路径（基本一致）

> 2.Parametes: -classpath $ModuleFileDir$/build/intermediates/classes/debug -s $FileClass$ 这里指的是定位到build的debug目录下执行 javap -s class文件

> 3.Working:$ModuleFileDir$   

这里介绍最常用的3个命令，对你的帮助应该还是很大的来看一下怎么使用： 

* javah -jni的使用：选中native文件--->右键--->External Tools--->javah -jni
效果如下：

![](http://i.imgur.com/tztfRxr.png)
![](http://i.imgur.com/b1hTJbI.png)

是不是自动生成了包名.类名的.h文件。

* ndk-build的使用：选中jni文件--->右键--->External Tools--->ndk-build
效果如下：

![](http://i.imgur.com/SR7qMav.png)
![](http://i.imgur.com/eoRsBSg.png)  

是不是一键生成了7种so库，你还想去debug目录下面去找吗

* javap-s的使用：选中native文件--->右键--->External Tools--->javap-s
效果如下：

![](http://i.imgur.com/wdh5pWT.png)

看见了每个方法下的descriptor属性的值就是你所要的方法签名。

> 3种一键生成的命令讲完了，以后你用到了什么命令都可以这样设置，是不是很给力。
##新实验版Gradle插件与AS下NDK开发
近期的 AS 与 Gradle 版本的快速更新对 NDK 开发又有了更加牛叉的体验，因为它完全支持使用 GDB 和 LLDB （不清楚这两是啥的请自行脑部Unix编程基础）来 GUI 化 debug 我们得 native 代码了（以前真的好蛋疼，命令行巴拉巴拉的，泪奔啊！）。
总之现在的 AS 和 Gradle 已经趋于实验完善 NDK 开发了，主要表现在如下方面：

* AS 完全支持 GUI 模式的 GDB/LLDB 调试 native 代码（LLDB调试引擎需要gradle-experimental plugin的支持）。
* AS 可以很好的直接编写 native 代码。
* Java 层代码声明好以后 AS 可以自动帮我们生成 JNI 接口规范代码。
*推出了几乎针对 NDK 的实验版 Gradle 插件。
可以看见，现在 NDK 开发已经渐渐的变得越来越方便了，牛叉的一逼！
因为是实验版本，所以我就试了下，不作为今后开发的主要方向，但是还是需要了解下。区别如下：

 1.情况1
```
//Project的build.gradle文件
buildscript {
    repositories {
       jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle-experimental:0.7.0-alpha1'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```
2.情况2
```
//Module的build.gradle文件
apply plugin: 'com.android.model.application'

model {
    android {
        compileSdkVersion = 23
        buildToolsVersion = "23.0.2"

        defaultConfig.with {
            applicationId = "com.losileeya.getapkinfo"
            minSdkVersion.apiLevel = 8
            targetSdkVersion.apiLevel = 23
        }
    }

    /*
     * native build settings
     */
    android.ndk {
        moduleName = "JNI_ANDROID"
        /*
         * Other ndk flags configurable here are
         * cppFlags.add("-fno-rtti")
         * cppFlags.add("-fno-exceptions")
         * ldLibs.addAll(["android", "log"])
         * stl       = "system"
         */
    }
  
    android.productFlavors {
        // for detailed abiFilter descriptions, refer to "Supported ABIs" @
        // https://developer.android.com/ndk/guides/abis.html#sa
        create("arm") {
            ndk.abiFilters.add("armeabi")
        }
        create("arm7") {
            ndk.abiFilters.add("armeabi-v7a")
        }
        create("arm8") {
            ndk.abiFilters.add("arm64-v8a")
        }
        create("x86") {
            ndk.abiFilters.add("x86")
        }
        create("x86-64") {
            ndk.abiFilters.add("x86_64")
        }
        create("mips") {
            ndk.abiFilters.add("mips")
        }
        create("mips-64") {
            ndk.abiFilters.add("mips64")
        }
        // To include all cpu architectures, leaves abiFilters empty
        create("all")
    }
```
可以明显感觉到 Project 和 Module 的 build.gradle 文件编写闭包都有了变化。
入门就讲完了，你也可以删掉jni目录，把so库放入jniLibs下，效果还是一模一样的，很晚了，睡觉。
##总结
初步使用ndk的技巧已经说完了，后续还会介绍jni中c调用java以及java调用c和相关一系列ndk开发中所要注意的。

***注意事项***
>1.jni调用前记得申明，比如：#include  stdio.h，#include   jni.h，#include stdlib.h，方法被调用者写前面或者头文件里面
>
>2.c中env调方法时（*env）->但是cpp中就得这样env->,原因是cpp中是一级指针，所以指针特别注意  


demo 传送门：[GetApkInfo.rar](http://download.csdn.net/detail/u013278099/9578062)