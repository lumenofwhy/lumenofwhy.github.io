---
title: windows下的CMake使用
date: 2022-10-17 22:35:16
id: 1666017316
tags:
  - 工具使用
categories:
  - 工具
keywords: CMake,编译,链接
description: windows下的CMake使用
---

### install 函数

```bash
install(TARGETS targets... [EXPORT <export-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
          PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```

1. 可执行文件被当做`RUNTIME`;
2. 静态库一般被看做`ARCHIVE`，模块库（Module libraries）当做`LIBRARY`;
3. 非DLL平台，动态库被当做`LIBRARY`;
4. DLL平台，动态库的DLL部分被当做`RUNTIME`,导入库视为`ARCHIVE`;
5. 索引基于Windows的系统包括(Cygwin)是DLL平台。
6. 目标库视为`OBJECTS`；
7. `ARCHIVE`, `LIBRARY`, `RUNTIME`, `OBJECTS` 与 `FRAMEWORK`改变后续属性所作用的目标的类型。如果不指定，则安装属性适用所有目标类型。 如果仅给出一个，则将仅安装该类型的目标（可用于仅安装DLL或导入库）。

## 静态库、动态库、导入库

|   ×    | windows（gcc） | windows（msvc） |           linux           |
|:------:|:------------:|:-------------:|:-------------------------:|
| 静态库 |     .lib     |     .lib      |            .a             |
| 动态库（运行时使用） |     .dll     |     .dll      |            .so            |
| 导入库（编译链接时使用）|    .dll.a    |     .lib      | .so文件兼作动态库和导入库 |

> 导入库： 让加载和使用动态库的过程变得自动化的库，在编译时链接到程序中，然后使得动态库的功能可以像静态库一样有效地使用。
> 导入库包含被 DLL 导出的函数和变量的符号名， DLL 包含实际的函数和数据。在编译链接可执行文件时，只需要链接导入库，DLL中的函数代码和数据并不复制到可执行文件中，在运行的时候，再去加载DLL，访问DLL中导出的函数。
>
> MSVC 与 MingGW 编译的静态库，不通用（两者编译器生成的 .o 跟 .obj 目标文件是 ABI 兼容的，但是他们的链接器不通用）。而动态库文件 .dll 是通用的

### 静态库（归档）与静态链接

由直接编译并链接到程序中的例程组成。编译使用静态库的程序时，程序所使用的静态库的所有功能都将成为可执行文件的一部分。在Windows上，静态库通常具有.lib扩展名，而在linux上，静态库通常具有.a（archive）扩展名。
静态库的一个优点是，用户只需发布可执行文件即可。由于库成为程序的一部分，这将确保程序始终使用正确版本的库。另外，因为静态库成为程序的一部分，所以可以像为自己的程序编写的功能一样使用它们。
缺点是，由于库的副本成为使用它的每个可执行文件的一部分，这可能会造成大量的空间浪费。静态库也不容易升级——要更新库，需要替换整个可执行文件。

### 动态库（共享库）与动态链接

由运行时加载到应用程序中的子程序组成。当编译使用动态库的程序时，库不会成为可执行文件的一部分，而是作为单独的单元保留。在Windows上，动态库通常具有.dll（动态链接库）扩展名，而在Linux上，动态库通常具有.so（共享对象）扩展名。

> 动态库的一个优点是许多程序可以共享一个副本，这节省了空间。
>
> 动态库可以升级到一个新版本，而不必替换使用它的所有可执行文件。
>
> 由于动态库未链接到程序中，因此使用动态库的程序必须显式加载并与动态库交互。这种机制可能会让人困惑，并使与动态库的交互变得不易处理。为了使动态库更易于使用，可以使用导入库。

#### 装载时动态链接（Load-time Dynamic Linking）

这种用法的前提是在编译之前已经明确知道要调用DLL中的哪几个函数，编译时在目标文件中只保留必要的链接信息，而不含DLL函数的代码；当程序执行时，调用函数的时候利用链接信息加载DLL函数代码并在内存中将其链接入调用程序的执行空间中(全部函数加载进内存），其主要目的是便于代码共享。（动态加载程序，处在加载阶段，主要为了共享代码，共享代码内存）

#### 运行时动态链接（Run-time Dynamic Linking）

这种方式是指在编译之前并不知道将会调用哪些DLL函数，完全是在运行过程中根据需要决定应调用哪个函数，将其加载到内存中（只加载调用的函数进内存），并标识内存地址，其他程序也可以使用该程序，并用LoadLibrary和GetProcAddress动态获得DLL函数的入口地址。（dll在内存中只存在一份，处在运行阶段）

#### windows 下动态链接库的查找顺序

exe文件在执行的时候，检索dll文件的顺序如下:

1. 应用程序所在目录 exe
2. 当前目录
3. 系统目录 : C:\Windows\System32
4. Windows目录: C:\Windows
5. 环境变量PATH中所有目录

所以使用编译好的dll 库的三种常见做法

1. 将 dll 文件与应用程序exe，放置到同一个目录
2. 将 dll 文件拷贝/安装到 C:\Windows\System32
3. 将 dll 文件夹所在的路径添加到系统环境变量PATH里面

> CMAKE的TARGET_LINK_LIBRARIES 指令，只负责编译过程。exe执行的时候，并不会按照CMake文件里面指定的路径来查找dll路径，而是遵循Windows下动态链接库的查找顺序
>
> 如果没有设置的话，出现的情况就是 exe 编译成功，运行的时候不执行任何操作就结束

### 导入库

让加载和使用动态库的过程变得自动化的库。在Windows上，这通常是通过与动态库（.dll）同名的小型静态库（.lib）来完成的。小型静态库在编译时链接到程序中，然后动态库的功能可以像静态库一样有效地使用。在Linux上，共享对象（.so）文件兼作动态库和导入库。大多数链接器可以在创建动态库时为动态库构建导入库。
