---
title: windows下的Vcode+MinGW+vcpkg的使用
date: 2022-10-12 16:58:29
id: 1665565109
tags:
  - 工具使用
  - windows
categories:
  - 软件及自定义配置
keywords: windows,软件,自定义
description: windows下的Vcode+MinGW+Vcpkg的使用
---
## Vscode 安装

Microsoft Store 中安装

### Vscode 右键菜单配置

#### 空白处右键菜单里显示“Open with Code”

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\Directory\Background\shell\VSCode\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%V\""
```

#### 选中文件右键菜单里显示“Open with Code”

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\*\shell\VSCode\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%1\""
```

#### 选中文件夹右键菜单里显示“Open with Code”

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\shell\VSCode]
@="Open with Code"
"Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe"

[HKEY_CLASSES_ROOT\Directory\shell\VSCode\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%V\""
```

### Vscode 需要的插件

1. C/C++
2. CMake
3. CMake Tools

## MingGW 安装

```cmd
scoop install gcc -g
```

## make 安装

```cmd
scoop install make -g
```

## cmake 安装

```cmd
scoop install cmake -g
```

## vcpkg 安装

```cmd
scoop install vcpkg -g
```

## Vscode中配置使用 MinGW + cmake + vcpkg

### 配置系统环境变量

|          环境变量          |        值         |
|:--------------------------:|:-----------------:|
|   VCPKG_DEFAULT_TRIPLET    | x64-mingw-dynamic |
| VCPKG_DEFAULT_HOST_TRIPLET | x64-mingw-dynamic |

### 初始化 vcpkg 并安装包

[中文官网](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md)

[vcpkg and Mingw-w64](https://github.com/microsoft/vcpkg/blob/master/docs/users/mingw.md)

### 配置 vscode settings.json

```json
{
  "cmake.cmakePath": "[cmake root]/bin/cmake.exe",
  "cmake.mingwSearchDirs": [
        "[MinGW root]/bin",
  ],

  "cmake.configureSettings": {
    "CMAKE_TOOLCHAIN_FILE": "[vcpkg root]/scripts/buildsystems/vcpkg.cmake"
  }
}
```

在项目的 CMakeLists.txt 添加
(有时候配置的系统环境变量会失效。。。)

```cmd
set(VCPKG_TARGET_TRIPLET "x64-mingw-dynamic" CACHE STRING "VCPKG Target Triplet to use")
set(VCPKG_DEFAULT_HOST_TRIPLET "x64-mingw-dynamic")
```

> 如果要使用静态链接库，将 dynamic 替换为 static 并提前安装好对应的静态库即可

参考链接：

[vcpkg+CLion+cmake+MinGW使用](https://blog.csdn.net/weixin_40448140/article/details/109111042)

[Windows C++配置 vscode 与 vcpkg](https://zhuanlan.zhihu.com/p/563999913)

[vscode + cmake + vcpkg搭建c++开发环境](https://zhuanlan.zhihu.com/p/430835667)
