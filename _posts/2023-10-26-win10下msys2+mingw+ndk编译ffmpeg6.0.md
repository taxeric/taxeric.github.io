---
layout: post
title: win10下msys2+mingw+ndk编译ffmpeg6.0
subtitle: 
categories: ndk
tags: ffmpeg
---

## 下载并安装msys2

https://www.msys2.org/

若安装卡在50%，请点击**取消**停止安装，然后*（建议）*重启电脑重新安装。

## 安装mingw tool chain

打开msys2，执行

```shell
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

无输入回车执行，即安装全部，等待安装完成

## 下载ffmpeg源码

https://ffmpeg.org/download.html#build-windows

## 撰写脚本

脚本来自[Win10编译Android版本的FFmpeg](https://blog.csdn.net/TLuffy/article/details/128182904)，脚本路径位于**`ffmpeg路径下`**

xxx.sh

```shell
#!/bin/bash
# 用于编译android平台的脚本

# NDK所在目录
NDK_PATH=D:/Android/SDK/ndk/26.0.10792818 # tag1
# macOS 平台编译，其他平台看一下 $NDK_PATH/toolchains/llvm/prebuilt/ 下的文件夹名称
HOST_PLATFORM=windows-x86_64  #tag1
# minSdkVersion
API=32

TOOLCHAINS="$NDK_PATH/toolchains/llvm/prebuilt/$HOST_PLATFORM"
SYSROOT="$NDK_PATH/toolchains/llvm/prebuilt/$HOST_PLATFORM/sysroot"
# 生成 -fpic 与位置无关的代码
CFLAG="-D__ANDROID_API__=$API -Os -fPIC -DANDROID "
LDFLAG="-lc -lm -ldl -llog "

# 输出目录
PREFIX=`pwd`/android-build
# 日志输出目录
CONFIG_LOG_PATH=${PREFIX}/log
# 公共配置
COMMON_OPTIONS=
# 交叉配置
CONFIGURATION=

build() {
  APP_ABI=$1
  echo "======== > Start build $APP_ABI"
  case ${APP_ABI} in
  arm64-v8a)
    ARCH="aarch64"
    TARGET=$ARCH-linux-android
    CC="$TOOLCHAINS/bin/$TARGET$API-clang"
    CXX="$TOOLCHAINS/bin/$TARGET$API-clang++"
    LD="$TOOLCHAINS/bin/$TARGET$API-clang"
    CROSS_PREFIX="$TOOLCHAINS/bin/$TARGET-"
    EXTRA_CFLAGS="$CFLAG"
    EXTRA_LDFLAGS="$LDFLAG"
    EXTRA_OPTIONS=""
    ;;
  esac

  echo "-------- > Start clean workspace"
  make clean

  echo "-------- > Start build configuration"
  CONFIGURATION="$COMMON_OPTIONS"
  CONFIGURATION="$CONFIGURATION --logfile=$CONFIG_LOG_PATH/config_$APP_ABI.log"
  CONFIGURATION="$CONFIGURATION --prefix=$PREFIX"
  CONFIGURATION="$CONFIGURATION --libdir=$PREFIX/libs/$APP_ABI"
  CONFIGURATION="$CONFIGURATION --incdir=$PREFIX/includes/$APP_ABI"
  CONFIGURATION="$CONFIGURATION --pkgconfigdir=$PREFIX/pkgconfig/$APP_ABI"
  CONFIGURATION="$CONFIGURATION --cross-prefix=$CROSS_PREFIX"
  CONFIGURATION="$CONFIGURATION --arch=$ARCH"
  CONFIGURATION="$CONFIGURATION --sysroot=$SYSROOT"
  CONFIGURATION="$CONFIGURATION --cc=$CC"
  CONFIGURATION="$CONFIGURATION --cxx=$CXX"
  CONFIGURATION="$CONFIGURATION --ld=$LD"
  # nm 和 strip
  CONFIGURATION="$CONFIGURATION --nm=$TOOLCHAINS/bin/llvm-nm"
  CONFIGURATION="$CONFIGURATION --strip=$TOOLCHAINS/bin/llvm-strip"
  CONFIGURATION="$CONFIGURATION $EXTRA_OPTIONS"

  echo "-------- > Start config makefile with $CONFIGURATION --extra-cflags=${EXTRA_CFLAGS} --extra-ldflags=${EXTRA_LDFLAGS}"
  ./configure ${CONFIGURATION} \
  --extra-cflags="$EXTRA_CFLAGS" \
  --extra-ldflags="$EXTRA_LDFLAGS"

  echo "-------- > Start make $APP_ABI with -j1"
  make -j1

  echo "-------- > Start install $APP_ABI"
  make install
  echo "++++++++ > make and install $APP_ABI complete."

}

build_all() {
  #配置开源协议声明
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-gpl"
  #目标android平台
  COMMON_OPTIONS="$COMMON_OPTIONS --target-os=android"
  #取消默认的静态库
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-static"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-shared"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-protocols"
  #开启交叉编译
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-cross-compile"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-optimizations"
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-debug"
  #尽可能小
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-small"
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-doc"
  #不要命令（执行文件）
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-programs"  # do not build command line programs
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffmpeg"    # disable ffmpeg build
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffplay"    # disable ffplay build
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffprobe"   # disable ffprobe build
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-symver"
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-network"
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-x86asm"
  COMMON_OPTIONS="$COMMON_OPTIONS --disable-asm"
  #启用
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-pthreads"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-mediacodec"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-jni"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-zlib"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-pic"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-muxer=flv"
  #COMMON_OPTIONS="$COMMON_OPTIONS --enable-avresample"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=h264"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=mpeg4"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=mjpeg"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=png"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=vorbis"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=opus"
  COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=flac"
  echo "COMMON_OPTIONS=$COMMON_OPTIONS"
  echo "PREFIX=$PREFIX"
  echo "CONFIG_LOG_PATH=$CONFIG_LOG_PATH"
  mkdir -p ${CONFIG_LOG_PATH}
  build "arm64-v8a"
}

echo "-------- Start --------"
build_all
echo "-------- End --------"

```

该脚本只编译`arm64-v8a`版本，直接copy请修改ndk路径

## 执行脚本

打开MSYS2 MINGW64，跳转到脚本所在路径，执行

```
chmod +x xxx.sh
```

```
./ xxx.sh
```

等待编译完成

## err

若编译报错如下

```shell
vulkan.h fatal error:
'vulkan_beta.h' file not found
```

即缺少硬件加速相关，需要下载[VulkanSDK](https://vulkan.lunarg.com/sdk/home#windows)，下载完成后将`Include`目录下的**`vk_video、vulkan`**目录复制到ndk路径下**`toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/include`**，替换原有的vulkan文件夹，然后重新尝试编译

> 方案来自[NDK编译ffmpeg包含硬件加速vulkan和mediacodec](https://blog.csdn.net/flyfish1986/article/details/131555008)

