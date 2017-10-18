---
layout: post
title: NDK 增量更新
date: 2017-9-19
categories: blog
tags: [总结,知识管理]
description: NDK 增量更新步奏
---

## 处理流程:
1. 将应用的旧版本Apk与新版本Apk做差分,得到差分包(更新补丁)  xxx.datch;
2. 在用户下载了差分包后,在手机端组合起来;
3. 校验新合成的Apk文件是否完整,MD5或SHA1是否正确,正确则引导用户安装

#### 工具
1. 服务器端: 利用bsdiff得到差分包(更新补丁)
cmd命令格式:
  file.../bsdiff 旧版本apk包 新版本apk包 差分包.patch
2. 电脑: 
cmd命令格式:
  file.../bspatch 旧版本apk包 新版本apk包 差分包.patch
3. 手机端: 利用JNI合成完整apk

#### 1. 生成so文件
* 注: 
模拟器:x86 & x86_64  
真机: armeabi & armeabi-v7a & armeabi-v8a ...
1. 在GitHub找到SmartAppUpdates,在ApkPatchLibrary/jni 目录下获得c代码;
https://github.com/cundong/SmartAppUpdates
2. 在代码中定义native方法,
````
public class PatchUtils {
    static {
        System.loadLibrary("xxx"); //加载生成的so文件
    }
    public static native int patch(String oldApkPath,String newApkPath,String patchPath);
}
````
通过javah命令生成相应的.h文件
cmd命令格式:file.../javah 包名.xxx  (xxx需为.class文件)
3. 生成相应的c文件,把ApkPatchLibrary/jin目录下获得c文件进行修改:
文件名称修改 com_cundong_utils_PatchUtils.c
修改为 xxx_xxx_xxx_xxx.c
需修改xxx_xxx_xxx_xxx .c文件夹内的
18行
* include "com_cundong_utils_PatchUtils.h"
改为相对应的 include "xxx_xxx_xxx_xxx.h"
199行 
* jint JNICALL Java_com_cundong_utils_PatchUtils_patch 
改为xxx_xxx_xxx_xxx.h中15行中相对应的 jint JNICALL xxx_xxx_xxx_xxx_patch

4. 修改 Android.mk文件
把:
LOCAL_MODULE     := ApkPatchLibrary
LOCAL_SRC_FILES  := com_cundong_utils_PatchUtils.c
修改为:
LOCAL_MODULE     := xxx  
LOCAL_SRC_FILES  := xxx_xxx_xxx_xxx.c
* xxx 为在代码中写的xxx System.loadLibrary("xxx");
*  xxx_xxx_xxx_xxx.c 为前面生成的xxx_xxx_xxx_xxx.c文件名

5. 使用NDK生成so文件
smd命令格式:
file.../ndk-build

#### 2. 使用JNI(Android Studio 2.3.3)
1. 把xxx.so连同父文件夹复制到项目main目录下的jniLibs下 ,然后build项目;

![image.png](http://upload-images.jianshu.io/upload_images/6012361-0c2052a17664957d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 把旧版本apk和差分包放到sd卡中:

3. 现即可在项目中调用之前代码中的native方法了
````
String oldApkPath = Environment.getExternalStorageDirectory()+"/patchdemo/patchdemo_1.apk";
String newApkPath = Environment.getExternalStorageDirectory()+"/patchdemo/patchdemo_2.apk";
String patchPath = Environment.getExternalStorageDirectory()+"/patchdemo/patchdemo.patch";
//result为0则为成功,否则为失败
int result = PatchUtils.patch(oldApkPath,newApkPath,patchPath);
````







