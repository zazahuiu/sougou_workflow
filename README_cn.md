[3月24日开始 Workflow 中文介绍](https://github.com/sogou/workflow/blob/master/README_cn.md)

[Windows Build Guide in English](/README.md)

[![license MIT](https://img.shields.io/badge/License-Apache-yellow.svg)](https://git.sogou-inc.com/wujiaxu/Filter/blob/master/LICENSE)
[![C++](https://img.shields.io/badge/language-c++-red.svg)](https://en.cppreference.com/)
[![platform](https://img.shields.io/badge/platform-linux%20%7C%20macos%20%7C%20windows-lightgrey.svg)](#%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%B8%80%E4%BA%9B%E8%AE%BE%E8%AE%A1%E7%89%B9%E7%82%B9)
[![Build Status](https://travis-ci.com/sogou/workflow.svg?branch=windows)](https://travis-ci.com/sogou/workflow)

# Windows下编译

Workflow采用CMake作为构建系统，依赖OpenSSL，编译workflow前需要先安装**CMake和OpenSSL**

## 安装CMake
通过[CMake官网](https://cmake.org/download/)下载cmake对应的安装包安装即可

## 安装OpenSSL

### Windows下安装OpenSSL库包括以下几种方式

* 下载源码自行编译
* 下载二进制文件或安装包
* 通过Chocolatey安装
* 通过vcpkg安装

### 下载源码自行编译

可以去官网或者GitHub仓库进行代码下载，编译过程参见OpenSSL源码中的编译文档。

* [openssl官网](https://www.openssl.org/)
* [github仓库](https://github.com/openssl/openssl)

### 下载安装包安装

请自行通过搜索引擎进行搜索，将文件解压到特定目录，或者下载并运行安装包。
安装包下载时，请**不要安装light版本**，因为light版本中不包含include和lib目录，无法正常编译
* [安装包下载](https://slproweb.com/products/Win32OpenSSL.html)


### 通过Chocolatey安装

Chocolatey是一个 Windows下的包管理器，类似于 Linux 下的 apt-get或 yum，可以方便的进行软件包的安装，Chocolatey的安装参考[Chocolatey官网](https://www.chocolatey.org/)
安装完Chocolatey后，执行下面的命令，即可安装OpenSSL
```powershell
choco install openssl
```

### 通过vcpkg安装

vcpkg是一个Windows下的C++包管理工具，它极大地简化了第三方库的配置与安装，vcpkg的安装请参考微软官方文档[VCPKG官方文档](https://docs.microsoft.com/zh-cn/cpp/build/vcpkg?view=vs-2019)
安装完vcpkg后，执行下面的命令，即可安装OpenSSL
```powershell
vcpkg install openssl
```

## 编译workflow库
### 通过源码编译OpenSSL，生成VS工程
通过源码编译或者下载二进制进行openssl安装的，在编译过程中可以通过指定OPENSSL_ROOT_DIR的方式进行编译，在workflow根目录下执行下面的命令

```powershell
cmake -B [build directory] -S . -DOPENSSL_ROOT_DIR=[openssl directory]
```

### 通过安装包或Chocolatey安装OpenSSL，生成VS工程
通过安装包安装或者通过Chocolatey安装openssl的，无需指定openssl的路径，在workflow根目录下执行下面的命令

```powershell
cmake -B [build directory] -S .
```

### 通过vcpkg安装OpenSSL，生成VS工程
通过vcpkg安装openssl的，需要指定CMAKE_TOOLCHAIN_FILE的路径

```powershell
 cmake -B [build directory] -S . -DCMAKE_TOOLCHAIN_FILE=[vcpkg.cmake directory]
 # 如果上面执行失败，可以尝试指定DVCPKG_TARGET_TRIPLET。 x86-windows或x64-windows
cmake -B [build directory] -S . -DVCPKG_TARGET_TRIPLET=x86-windows -DCMAKE_TOOLCHAIN_FILE=[vcpkg.cmake directory]
```

### 其他
**[openssl directory]**: openssl目录，请自行替换

**[build directory]** : 构建目录，一般使用build目录

**[vcpkg.cmake directory]** : 一般为vcpkg根目录\scripts\buildsystems\vcpkg.cmake

执行完成后，在[build directory]下会生成workflow.sln，可以用VS打开进行编译，或执行cmake命令进行编译

```powershell
# 编译Debug版本
cmake --build [build directory] --config Debug
# 编译Release版本
cmake --build [build directory] --config Release
```

### 编译tutorial
```powershell

# 生成tutorial工程
cmake -B build_tutorial tutorial

# 编译Debug版本
cmake --build build_tutorial --config Debug
# 编译Release版本
cmake --build build_tutorial --config Release

# tutorial\Debug或者tutorial\Release目录下即为tutorial编译结果
```

# VCPKG安装workflow

Workflow已经支持VCPKG安装啦！**如果只是使用workflow，而不关注源码，可以使用VCPKG安装**。

VCPKG如何安装及使用详见[官方文档](https://docs.microsoft.com/zh-cn/cpp/build/vcpkg?view=msvc-160)

安装workflow当前版本：`vcpkg install workflow`

安装workflow最新HEAD提交版本：`vcpkg install workflow --head`

下面是各平台的安装示例：

## On Windows

```powershell
md D:\tmp
cd D:\tmp
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
.\bootstrap-vcpkg.bat
# 默认安装x86-windows，与.\vcpkg.exe install workflow:x86-windows相同
.\vcpkg.exe install workflow
# 也可以指定安装x64
.\vcpkg.exe install workflow:x64-windows

cd D:\tmp
git clone https://github.com/dengjunplusplus/workflow-vcpkg-tutorial

cd D:\tmp\workflow-vcpkg-tutorial\workflow
cmake  -DCMAKE_TOOLCHAIN_FILE=D:/tmp/vcpkg/scripts/buildsystems/vcpkg.cmake -B build
# 如果上面执行失败，可以尝试指定VCPKG_TARGET_TRIPLET。 x86-windows或x64-windows
cmake  -DCMAKE_TOOLCHAIN_FILE=D:/tmp/vcpkg/scripts/buildsystems/vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x86-windows -B build
cmake --build build --config Debug
cmake --build build --config Release

```

## On Linux & Mac

```bash
cd /tmp
rm -rf vcpkg
rm -rf workflow-vcpkg-tutorial
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg install workflow

cd ..
git clone https://github.com/dengjunplusplus/workflow-vcpkg-tutorial.git
cd workflow-vcpkg-tutorial/workflow
cmake -DCMAKE_TOOLCHAIN_FILE=/tmp/vcpkg/scripts/buildsystems/vcpkg.cmake -B build
# 如果上面执行失败，可以尝试指定DVCPKG_TARGET_TRIPLET。 x64-linux 或x86-linux或x64-osx
cmake -DCMAKE_TOOLCHAIN_FILE=/tmp/vcpkg/scripts/buildsystems/vcpkg.cmake  -DVCPKG_TARGET_TRIPLET=x64-linux -B build
cmake --build build --config Debug
cmake --build build --config Release

```

# 联系方式
如果编译和使用过程有什么问题，欢迎提Issue，或者通过邮件和QQ群(618773193)联系我们

## Workflow使用相关问题

* **Xie Han** - *[xiehan@sogou-inc.com](mailto:xiehan@sogou-inc.com)*
* **Li Yingxin** - *[liyingxin@sogou-inc.com](mailto:liyingxin@sogou-inc.com)*

## Windows分支相关问题或VCPKG使用问题

* **Deng Jun** - *[dengjun@sogou-inc.com](mailto:dengjun@sogou-inc.com)*
