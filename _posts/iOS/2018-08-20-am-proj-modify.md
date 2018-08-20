---
layout: post
title:  "pod & 工程结构化（实践）"
date:   2018-8-20 21:53:01 +0800
categories: iOS
tags: [iOS, dev, pod]
---

* TOC
{:toc}


# 原工程结构

## 原工程目录树

```shell
.
|____AMHexin.xcodeproj
|____AMHexin.xcworkspace
|____AM_code                   # 行情代码
|____WT_code                   # 交易代码
|____config_files              # 配置文件
|____OC_public_Interface       # 公用接口
|____ModuleIntergrate          # 其他部门SDK集成

```

## 原工程 workspace 结构

![image.png]({{ site.img_url }}imgs/iOS/原工程结构.png)

## 原工程 Podfile define

{% capture code-capture %}

```ruby
abstract_target 'IHexin' do
    pod 'Aspect'
    pod 'AFNetworking'
    pod 'Masonry'
    pod 'MBProgressHUB'
    pod 'TMCache'
    pod 'SDWebImage'
    pod 'SSZipArchive'
    pod 'MJRefresh'
    pod 'WTModuleSDK'
    pod 'WTModuleResource'
    pod 'RNSVG', :path => 'ReactNativeModule/react-native-svg'
    pod 'React', :path => 'ReactNativeModule/', :subspecs => [
        # ....subspecs
    ]
    pod 'RNSVG', :path => 'ReactNativeModule/ReactCommon/yoga'

    target 'EQHexin' do
    end

    target 'EQHexinB' do
    end

    target 'EQHexinPro' do
    end
end
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats" button-text="Toggle Code" toggle-text=code-capture %}

# 原工程结构

##  现工程目录树

```shell
.
|____AMHexin.xcodeproj
|____AMHexin.xcworkspace
|____AM_code                   # 行情代码
|____WT_code                   # 交易代码
|____config_files              # 配置文件
|____OC_public_Interface       # 公用接口
|____ModuleIntergrate          # 其他部门SDK集成
|____AMIphoneBase              # am_iphone 基础代码 静态库
|  |____AMIphoneBase.xcodeproj 
|____AMIphoneDebug             # am_iphone debug代码 静态库          
|  |____AMIphoneDebug.xcodeproj
|____ReactNativeModule         # rn 动态库
   |____reactNativeSDK.xcodeproj
   |____reactNativeSDK.xcworkspace
```

## 现工程 workspace 结构


![image.png]({{ site.img_url }}imgs/iOS/现在工程结构截图.png)


## 现工程 Podfile define

{% capture code-capture %}

```ruby
def eq
    pod 'EQBase'
end

def am
    pod 'AMIphoneBase', :path => 'AMIphoneBase/'
end

def am_debug
    pod 'AMIphoneDebug', :path => 'AMIphoneDebug', :configurations => ['Adhoc', 'Beta', 'Debug']
end

def rn
    pod 'reactNativeSDK', :path => 'ReactNativeModule/reactNativeSDK/'
end

def base
    pod 'Aspects'
    pod 'AFNetworking'
    pod 'SDWebImage'
end

def masonry
    pod 'Masonry'
end

workspace 'AMHexion.xcworkspace'

target 'AMIphoneBase' do
    project 'AMIphoneBase/AMIphoneBase.xcodeproj'
    workspace 'AMHexin'
    eq
    base
end

target 'AMIphoneDebug' do
    project 'AMIphoneDebug/AMIphoneDebug.xcodeproj'
    worksapce 'AMHexin'
    eq
    am
    base
    masonry
end

abstract_target 'IHexin' do
    project 'AMHexin.xcodeproj'
    workspace 'AMHexin'
    eq
    am
    rn
    am_debug
    base
    masonry
    pod 'MBProgressHUB'
    pod 'TMCache'
    pod 'SSZipArchive'
    pod 'MJRefresh'
    pod 'WTModuleSDK'
    pod 'WTModuleResource'

    target 'EQHexin' do
    end

    target 'EQHexinB' do
    end

    target 'EQHexinPro' do
    end
end
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats1" button-text="Toggle Code" toggle-text=code-capture %}

# 改动

> * 增加了`AMIphoneDebug` （**Debug下使用的 `HXPerformaceCenter` 等**）
> * 增加了`EQBase` （**C++实现**）
> * 增加了`AMIphoneBase` （**类目扩展 和 部分代码**）
> * 增加了`reactNativeSDK`

# AMIphoneDebug

## 实现功能点

> * 增加了`AMIphoneDebug` **Debug下使用的 `HXPerformaceCenter` 等**

## 功能目标

* `AMIphoneDebug`部分代码只在`Debug`才编译入最终的`products`
* `AMIphoneDebug` 部分代码不重复编译，占用编译时间

-------

* `AMIphoneDebug`部分代码与主工程`am_iphone`的依赖关系清楚可见

-------

## 使用到的功能特性

* `pod` 本地 podspec
* `pod` build configurations 配置

{% capture code-capture %}

```ruby
def am_debug
    pod 'AMIphoneDebug', :path => 'AMIphoneDebug', :configurations => ['Adhoc', 'Beta', 'Debug']
=begin
:path => 'AMIphoneDebug' 会去查找 ./AMIphoneDebug目录下的pod名称为AMIphoneDebug的 podspec
:configurations => ['Adhoc', 'Beta', 'Debug'] 生成的Release的xcconfig文件中将不会声明加入AMIphoneDebug进行链接
=end
end
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats2" button-text="Toggle Code" toggle-text=code-capture %}

### 本地podspec

#### 远程pod流程

> 1. 同步远程pod仓库到本地`~/.cocoapods/repos/28-communism`，我们使用的是私有仓库`communsim`
> 2. 查找对应的podspec
> 3. `git clone`对应的源代码到本地`~/Library/Caches/CocoaPods/Pods/Relase/`
> 4. 根据`podspec`, 生成对应的Pods.xcodeproj文件，拷贝`源代码文件`，执行 `pod hook` `pod plugin`，根据`podspec`和`Podfile` 生成 `xconfig` 以及 `framework-install.sh`脚本
> 5. 主工程`Build Phases`中插入, `Check Pods Manifest.lock`，`Embed Pods Frameworks`， `Copy Pods Resource`
> 5. 计算依赖项hash值以及记录版本号，生成 `Podfile.lock` 以及 `Manifest.lock`

#### 本地pod流程

> 1. 查找对应的podspec
> 2. 根据`podspec`, 生成对应的Pods.xcodeproj文件，拷贝`源代码文件`，执行 `pod hook` `pod plugin`，根据`podspec`和`Podfile` 生成 `xconfig` 以及 `framework-install.sh`脚本
> 3. 主工程`Build Phases`中插入, `Check Pods Manifest.lock`，`Embed Pods Frameworks`， `Copy Pods Resource`
> 4. 计算依赖项hash值以及记录版本号，生成 `Podfile.lock` 以及 `Manifest.lock`

#### xcconfig

> `xcodeproj` 有许多层级的编译指令描述配置， 主要有单文件的 `Complier Flages`, `xcodeporj` 文件内的 `Build Settings`， 以及 `xcconfig` 文件级别的配置， 还有 xcodebuild 以及 xcrun的命令 option。 （动态性依次递增）

> 上述配置之后 `Relase` 的 `xcconfig` 中 的 `HEADER_SEARCH_PATH` （头文件查找路径）, `OTHER_CfLAGS`（额外编译选项），`OTHER_LDFLAGS` （额外链接选项）中将不包含`AMIphoneDebug`

-------------------

* `pod` vendored_framework

> 直接使用已编译完的`framework`

```ruby
s.verndored_frameworks = "products/AMIphoneDebug.framework"
```

* `podfile` workspace target 声明

> 多个不同的`xcodeproj`可以共用一个`Podfile`声明依赖，并在一个`workspace`中进行开发

> 由于 `AMIphoneDebug` 以及 `AMIphoneBase` 依赖 `am_iphone` 的部分实现， 这样通过 `git status` 更容易看出改动影响面，所以没选择完全使用`cocoapods`管理

{% capture code-capture %}

```ruby
workspace 'AMHexion.xcworkspace'
target 'AMIphoneDebug' do
    project 'AMIphoneDebug/AMIphoneDebug.xcodeproj'
    worksapce 'AMHexin'
    eq
    am
    base
    masonry
end
target 'AMIphoneBase' do
    project 'AMIphoneBase/AMIphoneBase.xcodeproj'
    workspace 'AMHexin'
    eq
    base
end
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats3" button-text="Toggle Code" toggle-text=code-capture %}

* 静态库和动态库

> 静态库不需要完全链接， 动态库需要完全链接

> 由于 `AMIphoneDebug` 以及 `AMIphoneBase` 依赖 `am_iphone` 的部分实现，不能完全链接，所以只能使用静态库编译

# EQBase

## 实现功能点

> * 增加了`EQBase` （**C++实现**）

## 功能目标

> `EQBase` 部分代码不重复编译，占用编译时间
> `EQBase` （**C++实现**）对外隐藏

## 使用到的功能特性

* git 子仓库
* gitlab 权限组


# AMIphoneBase

## 实现功能点 

> * 增加了`AMIphoneBase` （**类目扩展 和 部分代码**）

> 实现和`AMIphoneDebug`类似

# reactNativeSDK

## 实现功能点

> * 增加了`reactNativeSDK`

## 功能目标

* `reactNativeSDK` 部分代码不重复编译，占用编译时间

## 使用到的功能特性

* `pod` 空工程编译3个依赖项

> 编译空工程`reactNativeSDK` 获取`RNSVG`，`React`，`RNSVG` 三个framework， 通过 `vendored_framework` 统一引入。

{% capture code-capture %}

```ruby
# reactNativeSDK Podfile
target 'reactNativeSDK' do
    pod 'RNSVG', :path => 'ReactNativeModule/react-native-svg'
    pod 'React', :path => 'ReactNativeModule/', :subspecs => [
        # ....subspecs
    ]
    pod 'RNSVG', :path => 'ReactNativeModule/ReactCommon/yoga'
end
```

```ruby
# reactNativeSDK podspec
def rn
    pod 'reactNativeSDK', :path => 'ReactNativeModule/reactNativeSDK/'
end
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats4" button-text="Toggle Code" toggle-text=code-capture %}

* `pod` vendored_framework