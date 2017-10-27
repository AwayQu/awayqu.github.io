---
layout: post
title:  "xcodebuild编译结果目录结构"
date:   2017-10-28 01:20:25 +0800
categories: iOS
tags: [iOS, clang, xcodebuild]
---

* TOC
{:toc}

# xcodebuild编译结果目录结构

-------

> * 目前讨论的目录结构的工程是`没有workspace的`。
> * Demo工程使用的infer的[example](https://github.com/facebook/infer/tree/master/examples/ios_hello)

## 编译命令 
```bash
xcodebuild -target HelloWorldApp -configuration Debug -sdk iphonesimulator
```

## 编译结果目录树
```bash
./build
├── Debug-iphonesimulator
│   ├── HelloWorldApp.app
│   │   ├── Base.lproj
│   │   │   ├── LaunchScreen.nib
│   │   │   └── Main.storyboardc
│   │   │       ├── Info.plist
│   │   │       ├── UIViewController-vXZ-lx-hvc.nib
│   │   │       └── vXZ-lx-hvc-view-kh9-bI-dsS.nib
│   │   ├── HelloWorldApp
│   │   ├── Info.plist
│   │   ├── PkgInfo
│   │   └── _CodeSignature
│   │       └── CodeResources
│   └── HelloWorldApp.app.dSYM
│       └── Contents
│           ├── Info.plist
│           └── Resources
│               └── DWARF
│                   └── HelloWorldApp
└── HelloWorldApp.build
    └── Debug-iphonesimulator
        └── HelloWorldApp.build
            ├── Base.lproj
            │   └── Main.storyboardc
            │       ├── Info.plist
            │       ├── UIViewController-vXZ-lx-hvc.nib
            │       └── vXZ-lx-hvc-view-kh9-bI-dsS.nib
            ├── HelloWorldApp-all-non-framework-target-headers.hmap
            ├── HelloWorldApp-all-target-headers.hmap
            ├── HelloWorldApp-generated-files.hmap
            ├── HelloWorldApp-own-target-headers.hmap
            ├── HelloWorldApp-project-headers.hmap
            ├── HelloWorldApp.app.xcent
            ├── HelloWorldApp.hmap
            ├── LaunchScreen-PartialInfo.plist
            ├── Main-SBPartialInfo.plist
            ├── Objects-normal
            │   └── i386
            │       ├── AppDelegate.d
            │       ├── AppDelegate.dia
            │       ├── AppDelegate.o
            │       ├── Hello.d
            │       ├── Hello.dia
            │       ├── Hello.o
            │       ├── HelloWorldApp.LinkFileList
            │       ├── HelloWorldApp_dependency_info.dat
            │       ├── ViewController.d
            │       ├── ViewController.dia
            │       ├── ViewController.o
            │       ├── main.d
            │       ├── main.dia
            │       └── main.o
            ├── assetcatalog_dependencies
            ├── assetcatalog_generated_info.plist
            ├── dgph
            └── dgph~

```

### 最上层目录

```bash
./build
├── Debug-iphonesimulator
└── HelloWorldApp.build
```

| 文件夹      |     内容 |     文件名称格式 | 
| :-------- | --------| --------:| 
|  Debug-iphonesimulator  |   编译结果包括`.app`以及 `dSYM`文件 | `$configuration-$sdk` |
|  HelloWorldApp.build  |   编译中间文件`Intermediates` | `$project_name-.build` |


### Debug-iphonesimulator(编译结果)

```bash
.
├── HelloWorldApp.app
│   ├── Base.lproj
│   │   ├── LaunchScreen.nib
│   │   └── Main.storyboardc
│   │       ├── Info.plist
│   │       ├── UIViewController-vXZ-lx-hvc.nib
│   │       └── vXZ-lx-hvc-view-kh9-bI-dsS.nib
│   ├── HelloWorldApp
│   ├── Info.plist
│   ├── PkgInfo
│   └── _CodeSignature
│       └── CodeResources
└── HelloWorldApp.app.dSYM
    └── Contents
        ├── Info.plist
        └── Resources
            └── DWARF
                └── HelloWorldApp

```


| 文件夹      |     内容 |     文件名称格式 | 
| :-------- | --------| --------:| 
|  HelloWorldApp.app/Base.lproj  |   编译的`资源文件` |  |
|  HelloWorldApp.app/HelloWorldApp  |   可执行二进制文件 |  |

`dSYM` 是`符号化文件`，细节这里不提。

### HelloWorldApp.build编译中间文件


```bash

./HelloWorldApp.build
└── Debug-iphonesimulator
    └── HelloWorldApp.build
        ├── Base.lproj
        │   └── Main.storyboardc
        │       ├── Info.plist
        │       ├── UIViewController-vXZ-lx-hvc.nib
        │       └── vXZ-lx-hvc-view-kh9-bI-dsS.nib
        ├── HelloWorldApp-all-non-framework-target-headers.hmap
        ├── HelloWorldApp-all-target-headers.hmap
        ├── HelloWorldApp-generated-files.hmap
        ├── HelloWorldApp-own-target-headers.hmap
        ├── HelloWorldApp-project-headers.hmap
        ├── HelloWorldApp.app.xcent
        ├── HelloWorldApp.hmap
        ├── LaunchScreen-PartialInfo.plist
        ├── Main-SBPartialInfo.plist
        ├── Objects-normal
        │   └── i386
        │       ├── AppDelegate.d
        │       ├── AppDelegate.dia
        │       ├── AppDelegate.o
        │       ├── Hello.d
        │       ├── Hello.dia
        │       ├── Hello.o
        │       ├── HelloWorldApp.LinkFileList
        │       ├── HelloWorldApp_dependency_info.dat
        │       ├── ViewController.d
        │       ├── ViewController.dia
        │       ├── ViewController.o
        │       ├── main.d
        │       ├── main.dia
        │       └── main.o
        ├── assetcatalog_dependencies
        ├── assetcatalog_generated_info.plist
        ├── dgph
        └── dgph~


```


| 文件夹      |     内容 |     文件名称格式 | 
| :-------- | --------| --------:| 
|  HelloWorldApp.build/Debug-iphonesimulator/HelloWorldApp.build  |  编译中间文件目录  | `$project_name-.build`/`$configuration-$sdk`/`$target_name.build` |
|  HelloWorldApp.hmap  |   `header map` 二进制的头文件映射集合 | `$production_name.build` |
|  AppDelegate.d  |   文件依赖头文件信息 | `$file_name.d` |
|  AppDelegate.dia  |   `clang诊断（diagnostics）` 序列化信息(二进制) | `$file_name.dia` |
|  AppDelegate.o  |   `编译后的AST信息`（二进制） | `$file_name.o` |

> Note: 
>  * 每一层的`prefix`都是`HelloWorldApp`， 但是实际上变量来源不一样。
>  * 需要多层目录的原因是让同一个`build目录`，支持编译一个`workspace`中的不同`project`的不同`target`的不同`架构（arch）`，这样编译文件不会重叠。其实之前的`Debug-Simulator`也是一样，为了区分不同`configuration`和不同的`sdk`
>  * `hmap`具体生成方式未知，在xcodebuild编译的最开始阶段生成，因为所有文件编译都会需要`hmap`，用于查找依赖头文件，**查到了对应声明，才能正确生成AST**。

### hmap相关

`clang`的源代码中包含一些信息，关于hmap以及不同的头文件引用种类。具体没有细看, 大概是头文件有各种查找方式，同时`-iquote`,  `-include`, `-I`这几种头文件依赖查找声明，会有些许不同。

> **clang源代码**
> **file:** ***HeaderSearchOptions.h***

```cpp
namespace frontend {
  /// IncludeDirGroup - Identifies the group an include Entry belongs to,
  /// representing its relative positive in the search list.
  /// \#include directives whose paths are enclosed by string quotes ("")
  /// start searching at the Quoted group (specified by '-iquote'),
  /// then search the Angled group, then the System group, etc.
  enum IncludeDirGroup {
    Quoted = 0,     ///< '\#include ""' paths, added by 'gcc -iquote'.
    Angled,         ///< Paths for '\#include <>' added by '-I'.
    IndexHeaderMap, ///< Like Angled, but marks header maps used when
                       ///  building frameworks.
    System,         ///< Like Angled, but marks system directories.
    ExternCSystem,  ///< Like System, but headers are implicitly wrapped in
                    ///  extern "C".
    CSystem,        ///< Like System, but only used for C.
    CXXSystem,      ///< Like System, but only used for C++.
    ObjCSystem,     ///< Like System, but only used for ObjC.
    ObjCXXSystem,   ///< Like System, but only used for ObjC++.
    After           ///< Like System, but searched after the system directories.
  };
}
```


# 相关$变量说明

-------

| 变量      |     内容 |    
| :-------- | -------: |
|  `$project_name`  |  `xcodeproj`文件的名称  | 
|  `$configuration`  |  `xcodebuild`命令输入时指定的编译配置， 在`pbxproj`中有相应声明，在`XCConfigurationList` | 
|  `$sdk`  |  `xcodebuild`输入时指定的`sdk` | 
|  `$target_name`  |  一个`target`的配置的名称， `pbxproj`中有相应声明, 在`PBXNativeTarget` | 
|  `$product_name`  |  一个`product`的名称， `pbxproj`中有相应声明，在`XCBuildConfiguration` |



# 引用

-------

[clang](https://github.com/llvm-mirror/clang)

[infer-example](https://github.com/facebook/infer/tree/master/examples/ios_hello)