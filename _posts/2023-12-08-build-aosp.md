---
show_subscribe: false	

title: 'MacOS-14.1 下载/编译/阅读 AOSP'
date: 2023-12-08
excerpt: '1：AOSP 下载/编译 2：将AOSP module 导入 AndroidStudio'
tags:
  - Android
  - AOSP
  - XCode 13.2.1
  - MacOS 14.1
  - android-12.0.0_r3
  - 整理分享
---


## 简介
> **[警告：自 2021 年 6 月 22 日起，我们将不再支持在 Windows 或 MacOS 上进行构建。
](https://source.android.google.cn/docs/setup/start/requirements?hl=zh-cn)**
> **针对 `Linux` 系统 Google 推出了 [平台版AndroidStudio](https://developer.android.com/studio/platform?hl=zh-cn)**
> 
> 文档版本：1.0.1
> 
> 设备信息:
> - Mac OS: 14.1.2
> - Chip: M1 
> - RAM: 16G
> - Core: 8
> - XCode: [13.2.1](https://download.developer.apple.com)
> - Android Version: android-12.0.0_r3 
> - 硬盘空间：我编译成功后占用空间大概244G，所以应准备大概300G左右用来编译。

## 编译环境准备

### 1. 安装旧版 XCode
因`AOSP`不再对 MAC 进行支持，所以我们使用最新版的`XCode`会遇到大量的问题，这些问题的产生是因为 XCode 包含的开发工具包的冲突导致的，其中尤以`Go` 和 `Rust`产生的问题最多。

我们的主要任务是编译AOSP而不是解决这些冲突，所以我建议你直接使用[XCode13.2.1](https://download.developer.apple.com)这样会帮助你避免很多不必要的麻烦。

### 2. 创建区分大小写的磁盘映像

通过 shell 使用以下命令创建磁盘映像：
```bash
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 1024g ~/android.dmg
```
这将创建一个 .dmg.sparseimage 文件，该文件在装载后可用作具有 Android 开发所需格式的存储卷。

如果您以后需要更大的存储卷，还可以使用以下命令来调整稀疏映像的大小：
```bash
hdiutil resize -size <new-size-you-want>g ~/android.dmg.sparseimage
```
对于存储在主目录下的名为 android.dmg 的磁盘映像，您可以向 `~/.bash_profile` 中添加辅助函数：

要在执行 mountAndroid 时装载磁盘映像，请运行以下命令：
```bash
function mountAndroid { hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android; }
```
要在执行 umountAndroid 时卸载磁盘映像，请运行以下命令：
```bash
function umountAndroid() { hdiutil detach /Volumes/android; }
```
## 初始化/下载/编译 AOSP，将模块导入AndroidStudio
**请提前下载好 `diffutils`工具，避免编译到90%时报`VNDK`相关Error.**
``` bash
brew install diffutils
```

[清华AOSP镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)文档建议了两种初始化方法，1. 下载初始化包 2. 直接 `repo init ...`。方法1的好处是能避免`repo init`失败的风险，但是我们现在要编译Android12，所以直接采取第二种方法通过`repo init`进行下载。

```bash
#1. repo 下载
mkdir ~/bin
PATH=~/bin:$PATH
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo

#2. 初始化 Android 12：
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-12.0.0_r3

#3. 同步源码树（以后只需执行这条命令来同步）：
repo sync -c j64

#4. 初始化环境
source build/envsetup.sh

#5. 避免 too many file open 的问题
ulimit -S -n 4096
 
#6. lunch target
lunch aosp_arm64-eng

#7. 编译, make提供了-j的函数帮助我们加快编译速度。但是为了避免不必要的编译失败，我们
#   就不设置了，make会自己根据系统判断选择的。
m

#8. 使用AIDEGen 导入模块到AndroidStudio
aidegen framework -s -i s
```

## 遇见的编译错误：
> Error:
> [VNDK library list has been changed. Changing the VNDK library list is not allowed in API locked ](https://stackoverflow.com/questions/74568500/failed-out-target-product-generic-arm64-obj-packaging-vndk-intermediates-check)
> 
> Solution：
> `brew install diffutils`
> 
```bash
[  7% 7816/108971] build out/target/product/generic_arm64/obj/PACKAGING/vndk_intermediates/check-list-timestamp
FAILED: out/target/product/generic_arm64/obj/PACKAGING/vndk_intermediates/check-list-timestamp
/bin/bash -c "(( diff --old-line-format=\"Removed %L\"    --new-line-format=\"Added %L\"      --unchanged-line-format=\"\"    build/make/target/product/gsi/29.txt out/target/product/generic_arm64/obj/PACKAGING/vndk_intermediates/libs.txt     || ( echo -e \" error: VNDK library list has been changed.\\n\" \"       Changing the VNDK library list is not allowed in API locked branches.\"; exit 1 )) ) && (mkdir -p out/target/product/generic_arm64/obj/PACKAGING/vndk_intermediates/ ) && (touch out/target/product/generic_arm64/obj/PACKAGING/vndk_intermediates/check-list-timestamp )"
diff: unrecognized option `--old-line-format=Removed %L'
usage: diff [-aBbdilpTtw] [-c | -e | -f | -n | -q | -u] [--ignore-case]
            [--no-ignore-case] [--normal] [--strip-trailing-cr] [--tabsize]
            [-I pattern] [-F pattern] [-L label] file1 file2
       diff [-aBbdilpTtw] [-I pattern] [-L label] [--ignore-case]
            [--no-ignore-case] [--normal] [--strip-trailing-cr] [--tabsize]
            [-F pattern] -C number file1 file2
       diff [-aBbdiltw] [-I pattern] [--ignore-case] [--no-ignore-case]
            [--normal] [--strip-trailing-cr] [--tabsize] -D string file1 file2
       diff [-aBbdilpTtw] [-I pattern] [-L label] [--ignore-case]
            [--no-ignore-case] [--normal] [--tabsize] [--strip-trailing-cr]
            [-F pattern] -U number file1 file2
       diff [-aBbdilNPprsTtw] [-c | -e | -f | -n | -q | -u] [--ignore-case]
            [--no-ignore-case] [--normal] [--tabsize] [-I pattern] [-L label]
            [-F pattern] [-S name] [-X file] [-x pattern] dir1 dir2
       diff [-aBbditwW] [--expand-tabs] [--ignore-all-blanks]
            [--ignore-blank-lines] [--ignore-case] [--minimal]
            [--no-ignore-file-name-case] [--strip-trailing-cr]
            [--suppress-common-lines] [--tabsize] [--text] [--width]
            -y | --side-by-side file1 file2
       diff [--help] [--version]
 error: VNDK library list has been changed.
        Changing the VNDK library list is not allowed in API locked branches.
12:06:16 ninja failed with: exit status 1
```

## 总结：
1. 按照文档这个流程跑下来，基本可以保证编译成功，而且遇到的问题极少。
2. 因为Google不再对Mac进行支持了，所以还是尽快投入Ubuntu的怀抱吧。

## 参考文档
> - [macOS Big Sur(11.7.5)编译Android 12.0源码过程总结](https://www.mobibrw.com/2023/37102)
> - [Google 搭建编译环境文档](https://source.android.google.cn/docs/setup?hl=zh-cn)
> - [清华 Git repo 镜像](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)
> - [清华 AOSP 镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
> - [AIDEGen README](https://android.googlesource.com/platform/tools/asuite/+/refs/heads/main/aidegen/README.md)
> - [使用 AIDEGen 将 AOSP 项目导入 Android Studio](https://juejin.cn/post/7166061140298956836)