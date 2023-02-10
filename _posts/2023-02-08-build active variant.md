---
layout: post
title: Build Active Variant
subtitle: 
categories: android
tags: android 技巧
---

在Android Studio的左侧菜单栏有`Build Variants`的功能，主要是打不同的包。

## build.gradle

新增如下

```kotlin
    defaultConfig {}
    buildTyped {}
    
	//新增
    flavorDimensions 'lanier' //当未指定维度时,运行出错,即最少存在一个维度
    productFlavors {
        
        localA {}
        localB {}
    }
    ...
```

经过调试，若风味空实现，则继承自defaultConfig；

- 每个风味可以指定维度（即上述的`laneir`）；
- 只存在一个维度时，可不指定维度；
- 存在多个维度时，需要至少有一个风味使用该维度；

编译后的变种为

```
localADebug
localARelease
localBDebug
localBRelease
```

## 如何查看是否为变种

```kotlin
Log.i(TAG, "${BuildConfig.BUILD_TYPE} ${BuildConfig.FLAVOR}")
```

当选择当前变种为`localADebug`时，直接运行程序，执行上述log，输出为

```kotlin
17:42:56.159  I  debug localA
```

切换变种为`localBDebug`时，输出为

```kotlin
17:53:30.731  I  debug localB
```

切换变种为`xxRelease`时，提示需要签名，暂时不考虑；

调整gradle

```kotlin
    flavorDimensions 'lanier'
    productFlavors {

        localA {
            applicationId 'com.test.a'
            versionCode 5
            dimension 'lanier'
        }
        localB {
            applicationId 'com.test.b'
            versionCode 6
            dimension 'lanier'
        }
    }
```

调整输出log

```kotlin
Log.i(TAG, "${BuildConfig.BUILD_TYPE} ${BuildConfig.FLAVOR} ")
Log.i(TAG, "${BuildConfig.APPLICATION_ID} ${BuildConfig.VERSION_CODE} ")
```

选择变种`localADebug`，运行，提示需要安装（applicationId已改变）。输出如下

```kotlin
18:08:06.744  I  debug localA 
18:08:06.744  I  com.test.a 5 
```

同理变种`localBDebug`输出如下

```kotlin
18:10:29.885  I  debug localB 
18:10:29.885  I  com.test.b 6 
```

## 根据风味改应用配置

在`main`同级目录新建与风味对应的目录`localA`和`localB`，方便起见直接将`main`下面的`res`目录复制到新建的目录下，将`localA`和`localB`的`app_name`变更，运行后应用名即变更，若没有，则先编译一次再运行。同理可得资源替换（需目录及文件名一致）。

## 根据风味调整baseUrl

使用`buildConfigField`，参数为`类型`,`名称`,`值`

调整风味

```kotlin
    flavorDimensions 'lanier'
    productFlavors {

        localA {
            applicationId 'com.test.a'
            versionCode 5
            dimension 'lanier'
            buildConfigField "String", "baseUrl", '"https://test.a.com/"'
        }
        localB {
            applicationId 'com.test.b'
            versionCode 6
            dimension 'lanier'
            buildConfigField "String", "baseUrl", '"https://test.b.com/"'
        }
    }
```

编译后得

```kotlin
val baseUrl = BuildConfig.baseUrl
```





