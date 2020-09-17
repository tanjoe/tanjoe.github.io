---
title: 在Windows上编译VTK 9.0.1及其示例
date: 2020-09-17 11:42:52
img: /images/VTKLogo.jpg
tags:
- VTK
- CMake
typora-root-url: 在Windows上编译VTK-9-0-1及其示例
---

## 1 准备工作

在Windows下编译VTK需要以下内容：

1. CMake 3.10及以上
2. Visual Studio及MSVC编译环境
3. VTK源码

### 1.1 下载VTK源码

可以在https://vtk.org/download/ 下载VTK的源码、测试数据、文档等内容

<img src="image-20200917114948740.png" style="zoom:80%;" />

其中，`VTKData-9.0.1.tar.gz`是VTK用于测试的数据，`VTKLargeData-9.0.1.tar.gz`是VTK部分示例程序所用的数据，这两个文件都可以不用下载，只下载`VTK-9.0.1.tar.gz`即可。



## 2 生成VTK

### 2.1 创建目录

1. 为VTK创建一个文件夹
2. 在文件夹内，创建`src`和`build`目录，分别存放源码和生成的文件
3. 将`VTK-9.0.1.tar.gz`解压到`src`内

目录结构类似于：

```
c:\data\cpp\vtk\build    <--空
c:\data\cpp\vtk\src
c:\data\cpp\vtk\src\Accelerators
c:\data\cpp\vtk\src\Charts
c:\data\cpp\vtk\src\....
```

### 2.2 运行CMake

1. 打开CMake-GUI

2. 选择VTK的源码路径和生成路径

   <img src="cmake1.png" style="zoom: 80%;" />

3. 点击“Configure”

4. 选择需要的“generator”，这里选择Visual Studio 15 2017 Win64作为生成器

5. 根据需要调整CMake选项。以下为部分常见的选项：

   - `CMAKE_INSTALL_PRFIX`：生成的VTK库的安装路径。根据需要选择即可

   - `VTK_BUILD_EXAMPLES`：默认为`OFF`，勾选后生成测试时将一并生成示例代码。由于源码中的测试工程管理不便，且测试资源需要单独下载和配置，这里选择不生成源码中的示例，后续单独克隆VTK的测试代码仓库并生成。
   - `VTK_USE_CUDA`：默认为`OFF`，勾选后开启对CUDA的支持
   - `VTK_GROUP_QT`：勾选后将开启对Qt的支持，编译VTK在Qt中的控件类等

6. 再次点击“Configure”进行配置

7. 点击“Generator”生成VS 2017对应的工程

8. 点击"Open Project"，这时会调用VS 2017打开CMake生成的工程

### 2.3 构建

在VS 2017中将配置切换到"Release"，右键`ALL_BUILD`项目，选择“生成”即可。构建完成后，在`build/bin/Release`下便能看到编译好的动态库文件。

### 2.4 安装

右键`INSTALL`项目，选择“生成”即可将VTK相关的头文件、库文件、动态库文件安装到`CMAKE_INSTALL_PREFIX`指定的位置。安装完成后，目录结构应类似于：

```
c:\program file\VTK\bin
c:\program file\VTK\include
c:\program file\VTK\lib
c:\program file\VTK\share
```

其中，`\bin`目录包含了所有的VTK动态库文件，可以该路径加入环境变量。

> 如果需要更改安装路径，再次打开CMake-GUI更改`CMAKE_INSTALL_PREFIX`后，重新加载工程生成`INSTALL`即可



## 3 生成示例

### 3.1 示例的来源

如前所言，VTK源码包内已经附带了一些示例程序，这些示例程序旨在以简单一致的格式来说明VTK的一些概念，然而这些例子只涵盖了VTK功能的一小部分。

另一方面，源码包还附带了数以百计的测试，这些测试位于源码包中的`/Testing`目录下。但这些测试代码旨在验证VTK本身工作的正确性，并不是合适的教育资源。

VTK官方现在使用[github pages](https://lorensen.github.io/VTKExamples/site/)来提供测试代码的说明，并且这些代码及资源文件统一集中在单独的仓库中，见https://gitlab.kitware.com/vtk/vtk-examples/-/tree/master/

### 3.2 生成示例

首先克隆vtk-examples仓库：

```shell
git clone git@gitlab.kitware.com:vtk/vtk-examples.git
```

vtk-examples同样可以使用CMake进行构建：

1. 打开CMake-GUI，选择源码路径和构建路径
2. 点击“Configure”
3. “generator”选择Visual Studio 15 2017 Win64
4. 再次点击“Configure”
5. 配置CMake选项，将`VTK_DIR`改为前文VTK的构建目录，这里选择为`C:/data/cpp/vtk/build`
6. 点击“Generate”生成工程文件
7. 点击"Open Project"打开工程
8. 使用VS 2017进行生成即可

> 注意，由于VTK和CMake发展都很快，vtk-examples中有些示例附带的`CMakeLists.txt`已然无法正确查找到VTK。如果确实需要运行该示例，需要手动修改`CMakeLists.txt`

![](image-20200917170158681.png)