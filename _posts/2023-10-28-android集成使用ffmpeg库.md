---
layout: post
title: android集成使用ffmpeg库
subtitle: 
categories: ndk
tags: ffmpeg
---

## 导入so

将编译好的so文件copy到jniLibs目录下（jniLibs目录与java和cpp同级）

## 导入.h

将编译好的includes目录copy到cpp目录下（只包含头文件，无需架构）

## build.gradle配置

测试build.gradle文件为.kts

```kotlin
        externalNativeBuild{
            cmake {
                abiFilters("arm64-v8a")
            }
        }
```

## CMakeLists配置

修改如下

```cmake
include_directories(
        includes
)
set(LIBRARY_DIR "${CMAKE_SOURCE_DIR}/../jniLibs/${CMAKE_ANDROID_API}")
add_library(avcodec SHARED IMPORTED)
set_target_properties(avcodec PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libavcodec.so")
add_library(avdevice SHARED IMPORTED)
set_target_properties(avdevice PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libavdevice.so")
add_library(avfilter SHARED IMPORTED)
set_target_properties(avfilter PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libavfilter.so")
add_library(avformat SHARED IMPORTED)
set_target_properties(avformat PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libavformat.so")
add_library(avutil SHARED IMPORTED)
set_target_properties(avutil PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libavutil.so")
add_library(postproc SHARED IMPORTED)
set_target_properties(postproc PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libpostproc.so")
add_library(swreasample SHARED IMPORTED)
set_target_properties(swreasample PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libswreasample.so")
add_library(swscale SHARED IMPORTED)
set_target_properties(swscale PROPERTIES IMPORTED_LOCATION "${LIBRARY_DIR}/libswscale.so")

target_link_libraries(${CMAKE_PROJECT_NAME}
        # List libraries link to the target library
        android
        log
        avcodec
        avdevice
        avfilter
        avformat
        avutil
        postproc
        swreasample
        swscale)
```

## 调试

```c++
extern "C"
JNIEXPORT jstring JNICALL
Java_com_lanier_ffmpegbuild_MainActivity_ffmpegVersion(JNIEnv *env, jobject thiz) {
    char strBuffer[1024 * 4] = {0};
    strcat(strBuffer, "libavcodec : ");
    strcat(strBuffer, AV_STRINGIFY(LIBAVCODEC_VERSION));
    strcat(strBuffer, "\nlibavformat : ");
    strcat(strBuffer, AV_STRINGIFY(LIBAVFORMAT_VERSION));
    strcat(strBuffer, "\nlibavutil : ");
    strcat(strBuffer, AV_STRINGIFY(LIBAVUTIL_VERSION));
    strcat(strBuffer, "\nlibavfilter : ");
    strcat(strBuffer, AV_STRINGIFY(LIBAVFILTER_VERSION));
    strcat(strBuffer, "\nlibswresample : ");
    strcat(strBuffer, AV_STRINGIFY(LIBSWRESAMPLE_VERSION));
    strcat(strBuffer, "\nlibswscale : ");
    strcat(strBuffer, AV_STRINGIFY(LIBSWSCALE_VERSION));
    strcat(strBuffer, "\navcodec_config : ");
    strcat(strBuffer, avcodec_configuration());
    strcat(strBuffer, "\navcodec_license : ");
    strcat(strBuffer, avcodec_license());

    return env->NewStringUTF(strBuffer);
}
```

对应的kt方法为

```kotlin
external fun ffmpegVersion(): String
```