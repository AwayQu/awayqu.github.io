---
layout: post
title:  "mach-o & 逆向"
date:   2019-03-03 21:53:01 +0800
categories: iOS
tags: [iOS, clang, dyld]
---


* TOC
{:toc}

# mach-o 文件结构

## 胖mach-o
```s
.
├── Fat Header
├── Mach-O-armv7
└── Mach-O-x86_64
```
## mach-o
```s
.
└── Mach-O-x86_64
    ├── Header
    ├── Load Commands
    │   ├── Segment Commnad 1
    │   └── Segment Commnad 2
    └── Data
        ├── Segment 1
        │   ├── Section 1 Data
        │   ├── Section 2 Data
        │   └── Section 3 Data
        └── Segment 2
            ├── Section 1 Data
            ├── Section 2 Data
            ├── Section 3 Data
            └── Section 4 Data
```



# **Header**

> mach-o 主要梗概


```s
> otool -h -v MachODemoexe                                                               
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64  X86_64        ALL  0x00     EXECUTE    20       2696   NOUNDEFS DYLDLINK TWOLEVEL PIE
```

## Header Define

**32位**
{% capture code-capture %}
```c
/*
 * The 32-bit mach header appears at the very beginning of the object file for
 * 32-bit architectures.
 */
struct mach_header {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
};
```
{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats" button-text="Toggle Code" toggle-text=code-capture %}

**64位**
{% capture code-capture %}
```c
/*
 * The 64-bit mach header appears at the very beginning of object files for
 * 64-bit architectures.
 */
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};

/* Constant for the magic field of the mach_header_64 (64-bit architectures) */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
```
{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats1" button-text="Toggle Code" toggle-text=code-capture %}

-----------------

# Load Commands

> 根据Load Commands的信息，加载动态库

**Load Commands 精简**
{% capture code-capture %}
```python
> jtool -l MachODemoexe           
LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x100000000	__PAGEZERO
LC 01: LC_SEGMENT_64          Mem: 0x100000000-0x100003000	__TEXT
LC 02: LC_SEGMENT_64          Mem: 0x100003000-0x100004000	__DATA
LC 03: LC_SEGMENT_64          Mem: 0x100004000-0x10000a000	__LINKEDIT
LC 04: LC_DYLD_INFO
LC 05: LC_SYMTAB
LC 06: LC_DYSYMTAB
LC 07: LC_LOAD_DYLINKER      	/usr/lib/dyld
LC 08: LC_UUID               	UUID: FB4B13D0-EB62-37EE-98CF-DD8B1CCB6B26
LC 09: LC_BUILD_VERSION      	Build Version:           Platform: Unknown? 12.1.0
LC 10: LC_SOURCE_VERSION     	Source Version:          0.0.0.0.0
LC 11: LC_MAIN               	Entry Point:             0x1770 (Mem: 0x100001770)
LC 12: LC_LOAD_DYLIB         	/System/Library/Frameworks/Foundation.framework/Foundation
LC 13: LC_LOAD_DYLIB         	/usr/lib/libobjc.A.dylib
LC 14: LC_LOAD_DYLIB         	/usr/lib/libSystem.B.dylib
LC 15: LC_LOAD_DYLIB         	/System/Library/Frameworks/UIKit.framework/UIKit
LC 16: LC_RPATH              	@executable_path/Frameworks
LC 17: LC_FUNCTION_STARTS    	Offset: 17256, Size: 16 (0x4368-0x4378)
LC 18: LC_DATA_IN_CODE       	Offset: 17272, Size: 0 (0x4378-0x4378)
LC 19: LC_CODE_SIGNATURE     	Offset: 21120, Size: 18784 (0x5280-0x9be0)
```
{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats2" button-text="Toggle Code" toggle-text=code-capture %}


### 步骤

> 这部分是我大致整的

>1.加载mach-o
```python
LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x100000000	__PAGEZERO
LC 01: LC_SEGMENT_64          Mem: 0x100000000-0x100003000	__TEXT
LC 02: LC_SEGMENT_64          Mem: 0x100003000-0x100004000	__DATA
LC 03: LC_SEGMENT_64          Mem: 0x100004000-0x10000a000	__LINKEDIT
```

> 2.找到`dynamic linker`声明
```python
LC_LOAD_DYLINKER      	/usr/lib/dyld
```

> 3.获取`rpath`
```python
LC 16: LC_RPATH              	@executable_path/Frameworks
```

> 4.`dyld` 加载动态库 `rebase` `binding` 「需要用到`__LINKEDIT`中的信息」
```python
LC 12: LC_LOAD_DYLIB         	/System/Library/Frameworks/Foundation.framework/Foundation
LC 13: LC_LOAD_DYLIB         	/usr/lib/libobjc.A.dylib
LC 14: LC_LOAD_DYLIB         	/usr/lib/libSystem.B.dylib
LC 15: LC_LOAD_DYLIB         	/System/Library/Frameworks/UIKit.framework/UIKit
```

> 5.跳回到main函数
```python
LC 11: LC_MAIN               	Entry Point:             0x1770 (Mem: 0x100001770)
```



**Load Commands 完整**
{% capture code-capture %}
```python
> jtool -l MachODemoexe                                                                 
LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x100000000	__PAGEZERO
LC 01: LC_SEGMENT_64          Mem: 0x100000000-0x100003000	__TEXT
	Mem: 0x100001730-0x100001a63		__TEXT.__text	(Normal)
	Mem: 0x100001a64-0x100001a8e		__TEXT.__stubs	(Symbol Stubs)
	Mem: 0x100001a90-0x100001ae6		__TEXT.__stub_helper	(Normal)
	Mem: 0x100001ae6-0x10000250d		__TEXT.__objc_methname	(C-String Literals)
	Mem: 0x10000250d-0x100002549		__TEXT.__objc_classname	(C-String Literals)
	Mem: 0x100002549-0x100002db6		__TEXT.__objc_methtype	(C-String Literals)
	Mem: 0x100002db6-0x100002e30		__TEXT.__cstring	(C-String Literals)
	Mem: 0x100002e30-0x100002faa		__TEXT.__entitlements
	Mem: 0x100002fac-0x100002ff4		__TEXT.__unwind_info
LC 02: LC_SEGMENT_64          Mem: 0x100003000-0x100004000	__DATA
	Mem: 0x100003000-0x100003010		__DATA.__nl_symbol_ptr	(Non-Lazy Symbol Ptrs)
	Mem: 0x100003010-0x100003020		__DATA.__got	(Non-Lazy Symbol Ptrs)
	Mem: 0x100003020-0x100003058		__DATA.__la_symbol_ptr	(Lazy Symbol Ptrs)
	Mem: 0x100003058-0x100003068		__DATA.__objc_classlist	(Normal)
	Mem: 0x100003068-0x100003078		__DATA.__objc_protolist
	Mem: 0x100003078-0x100003080		__DATA.__objc_imageinfo
	Mem: 0x100003080-0x100003c68		__DATA.__objc_const
	Mem: 0x100003c68-0x100003c78		__DATA.__objc_selrefs	(Literal Pointers)
	Mem: 0x100003c78-0x100003c80		__DATA.__objc_classrefs	(Normal)
	Mem: 0x100003c80-0x100003c88		__DATA.__objc_superrefs	(Normal)
	Mem: 0x100003c88-0x100003c90		__DATA.__objc_ivar
	Mem: 0x100003c90-0x100003d30		__DATA.__objc_data
	Mem: 0x100003d30-0x100003df0		__DATA.__data
LC 03: LC_SEGMENT_64          Mem: 0x100004000-0x10000a000	__LINKEDIT
LC 04: LC_DYLD_INFO
LC 05: LC_SYMTAB
	Symbol table is at offset 0x4378 (17272), 95 entries
	String table is at offset 0x49b0 (18864), 2256 bytes
LC 06: LC_DYSYMTAB
	   73 local symbols at index     0
	    6 external symbols at index  73
	   16 undefined symbols at index 79
	   No TOC
	   No modtab
	   18 Indirect symbols at offset 0x4968

LC 07: LC_LOAD_DYLINKER      	/usr/lib/dyld
LC 08: LC_UUID               	UUID: FB4B13D0-EB62-37EE-98CF-DD8B1CCB6B26
LC 09: LC_BUILD_VERSION      	Build Version:           Platform: Unknown? 12.1.0
LC 10: LC_SOURCE_VERSION     	Source Version:          0.0.0.0.0
LC 11: LC_MAIN               	Entry Point:             0x1770 (Mem: 0x100001770)
LC 12: LC_LOAD_DYLIB         	/System/Library/Frameworks/Foundation.framework/Foundation
LC 13: LC_LOAD_DYLIB         	/usr/lib/libobjc.A.dylib
LC 14: LC_LOAD_DYLIB         	/usr/lib/libSystem.B.dylib
LC 15: LC_LOAD_DYLIB         	/System/Library/Frameworks/UIKit.framework/UIKit
LC 16: LC_RPATH              	@executable_path/Frameworks
LC 17: LC_FUNCTION_STARTS    	Offset: 17256, Size: 16 (0x4368-0x4378)
LC 18: LC_DATA_IN_CODE       	Offset: 17272, Size: 0 (0x4378-0x4378)
LC 19: LC_CODE_SIGNATURE     	Offset: 21120, Size: 18784 (0x5280-0x9be0)
```
{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats3" button-text="Toggle Code" toggle-text=code-capture %}

# Segments

## Segments 类型

* `__PAGEZERO`  *虚拟地址空间，低位占位Segment*
* `__TEXT` *可执行代码 和 只读数据*
* `__DATA` *可读 可写数据* （copy on write）
* `__OBJC` *Objc 运行时相关内容* （已不再使用归并到了__DATA段和，__TEXT段）
* `__LINKEDIT` *动态链接 需要的相关信息*

## Segments 结构

> x86_64 可执行二进制mach-o 内容示例

{% capture code-capture %}

```python
> jtool --pages MachODemoexe
0x0-0x3000	__TEXT
	0x1730-0x1a63	__TEXT.__text
	0x1a64-0x1a8e	__TEXT.__stubs
	0x1a90-0x1ae6	__TEXT.__stub_helper
	0x1ae6-0x250d	__TEXT.__objc_methname
	0x250d-0x2549	__TEXT.__objc_classname
	0x2549-0x2db6	__TEXT.__objc_methtype
	0x2db6-0x2e30	__TEXT.__cstring
	0x2e30-0x2faa	__TEXT.__entitlements
	0x2fac-0x2ff4	__TEXT.__unwind_info
0x3000-0x4000	__DATA
	0x3000-0x3010	__DATA.__nl_symbol_ptr
	0x3010-0x3020	__DATA.__got
	0x3020-0x3058	__DATA.__la_symbol_ptr
	0x3058-0x3068	__DATA.__objc_classlist
	0x3068-0x3078	__DATA.__objc_protolist
	0x3078-0x3080	__DATA.__objc_imageinfo
	0x3080-0x3c68	__DATA.__objc_const
	0x3c68-0x3c78	__DATA.__objc_selrefs
	0x3c78-0x3c80	__DATA.__objc_classrefs
	0x3c80-0x3c88	__DATA.__objc_superrefs
	0x3c88-0x3c90	__DATA.__objc_ivar
	0x3c90-0x3d30	__DATA.__objc_data
	0x3d30-0x3df0	__DATA.__data
0x4000-0x9be0	__LINKEDIT
	0x4000-0x40c8	Rebase Info     (opcodes)
	0x40c8-0x41f8	Binding Info    (opcodes)
	0x41f8-0x42c8	Lazy Bind Info  (opcodes)
	0x42c8-0x4368	Exports
	0x4368-0x4378	Function Starts
	0x4378-0x4968	Symbol Table
	0x4378-0x4378	Data In Code
	0x4968-0x49b0	Indirect Symbol Table
	0x49b0-0x5280	String Table
	0x5280-0x9be0	Code Signature
```
{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats4" button-text="Toggle Code" toggle-text=code-capture %}

## Section 
```python
__TEXT.__text        主程序代码
__TEXT.__stubs       用于动态链接的桩
__TEXT.__stub_helper 用于动态链接的桩
__TEXT.__cstring     程序中c语言字符串
__TEXT.__const       常量
```

# Objc 运行时相关 section

```python

__TEXT.__objc_methname    OC方法名称
__TEXT.__objc_methtype    OC方法类型
__TEXT.__objc_classname   OC类名

__DATA.__objc_classlist   OC类列表
__DATA.__objc_protollist  OC原型列表
__DATA.__objc_imageinfo   OC镜像信息
__DATA.__objc_const       OC常量
__DATA.__objc_selfrefs    OC类自引用(self)
__DATA.__objc_superrefs   OC类超类引用(super)
__DATA.__objc_protolrefs  OC原型引用
__DATA.__bss              没有初始化和初始化为0 的全局变量

```

# 逆向实践

> 主要涉及到 `代码段` `sel name 字面量` `sel 定义`

```s
__TEXT.__text	(Normal)
__TEXT.__objc_methname	(C-String Literals)
__DATA.__objc_selrefs	(Literal Pointers)

```

## UIKit CoreAnimation CATransaction 提交时机以及实现

> 原先UIKit库直接是一个`UIKit`mach-o文件
  目前iOS12 使用了`UIKit.tbd` stub libraires (说是用来减小空间 还有 增加兼容性)
  改了以后结构

**拆分成了多个**

```s
.
└── UIKit
    ├── UIFoundation
    └── UIKitCore
		|...
		|...
		└── UIKitServices
```



`/System/iOSSupport/System/Library/Frameworks/UIKit.framework`

`/System/Library/PrivateFrameworks/UIFoundation.framework/UIFoundation`

{% capture code-capture %}
```s
> vim /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/System/Library/Frameworks/UIKit.framework/UIKit.tbd
 archs:           [ armv7, armv7s, arm64, arm64e ]
uuids:           [ 'armv7: BEBC4565-E9B4-393F-9F26-8F54D55F29D1', 'armv7s: D3F9ABEB-27BE-3945-8B7F-EF960AC09684',
                   'arm64: BFF07886-59D6-3F76-936A-36B9C6D6D438', 'arm64e: 15EB540C-B6B8-3EEA-BC21-593E5EEBBA89' ]
platform:        ios
install-name:    /System/Library/Frameworks/UIKit.framework/UIKit
current-version: 61000
objc-constraint: none
exports:
  - archs:           [ armv7, armv7s ]
    re-exports:      [ /System/Library/PrivateFrameworks/UIFoundation.framework/UIFoundation ]

```
{% endcapture %}
{% include toggle-field.html toggle-name="toggle-thats8" button-text="Toggle Code" toggle-text=code-capture %}

`/System/iOSSupport/System/Library/PrivateFrameworks/UIKitCore.framework`

{% capture code-capture %}

```s
> jtool -l /System/iOSSupport/System/Library/Frameworks/UIKit.framework/Versions/Current
/UIKit

LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x1000	__TEXT
	Mem: 0x000000fd0-0x000000fd0		__TEXT.__text	(Normal)
	Mem: 0x000000fd0-0x000001000		__TEXT.__const	
LC 01: LC_SEGMENT_64          Mem: 0x000001000-0x4000	__LINKEDIT
LC 02: LC_ID_DYLIB           	/System/iOSSupport/System/Library/Frameworks/UIKit.framework/Versions/A/UIKit
LC 03: LC_DYLD_INFO          
LC 04: LC_SYMTAB             
	Symbol table is at offset 0x1008 (4104), 3 entries
	String table is at offset 0x1038 (4152), 64 bytes
LC 05: LC_DYSYMTAB           
	    2 local symbols at index     0
	   No external symbols
	    1 undefined symbols at index 2
	   No TOC
	   No modtab
	   No Indirect symbols

LC 06: LC_UUID               	UUID: 5410F2AF-351A-3BF0-ACEB-01631FE110CC
LC 07: LC_BUILD_VERSION      	Build Version:           Platform: Unknown? 12.0.0
LC 08: LC_SOURCE_VERSION     	Source Version:          3698.94.2.0.0
LC 09: LC_REEXPORT_DYLIB     	/System/iOSSupport/System/Library/PrivateFrameworks/UIKitCore.framework/Versions/A/UIKitCore
LC 10: LC_LOAD_DYLIB         	/usr/lib/libSystem.B.dylib
LC 11: LC_FUNCTION_STARTS    	Offset: 4096, Size: 8 (0x1000-0x1008) 
LC 12: LC_DATA_IN_CODE       	Offset: 4104, Size: 0 (0x1008-0x1008) 
LC 13: LC_CODE_SIGNATURE     	Offset: 4224, Size: 9376 (0x1080-0x3520)
```
{% endcapture %}
{% include toggle-field.html toggle-name="toggle-thats8" button-text="Toggle Code" toggle-text=code-capture %}

{% capture code-capture %}

```s
> jtool -l /System/iOSSupport/System/Library/PrivateFrameworks/UIKitCore.framework/Versions/A/UIKitCore
LC 00: LC_SEGMENT_64          Mem: 0x000000000-0x1000000	__TEXT
	Mem: 0x000002920-0x000d480c3		__TEXT.__text	(Normal)
	Mem: 0x000d480c4-0x000d49a44		__TEXT.__stubs	(Symbol Stubs)
	Mem: 0x000d49a44-0x000d4c4c0		__TEXT.__stub_helper	(Normal)
	Mem: 0x000d4c4c0-0x000d56288		__TEXT.__const	
	Mem: 0x000d56288-0x000e8e57d		__TEXT.__objc_methname	(C-String Literals)
	Mem: 0x000e8e57d-0x000f5d924		__TEXT.__cstring	(C-String Literals)
	Mem: 0x000f5d924-0x000f749a3		__TEXT.__objc_classname	(C-String Literals)
	Mem: 0x000f749a3-0x000faa289		__TEXT.__objc_methtype	(C-String Literals)
	Mem: 0x000faa290-0x000fb8f93		__TEXT.__oslogstring	(C-String Literals)
	Mem: 0x000fb8f94-0x000fd4f0c		__TEXT.__gcc_except_tab	
	Mem: 0x000fd4f0c-0x000fd592c		__TEXT.__ustring	
	Mem: 0x000fd592c-0x000ffffc8		__TEXT.__unwind_info	
	Mem: 0x000ffffc8-0x001000000		__TEXT.__eh_frame	
LC 01: LC_SEGMENT_64          Mem: 0x001000000-0x13d8000	__DATA
	Mem: 0x001000000-0x001000010		__DATA.__nl_symbol_ptr	(Non-Lazy Symbol Ptrs)
	Mem: 0x001000010-0x001001298		__DATA.__got	(Non-Lazy Symbol Ptrs)
	Mem: 0x001001298-0x001003498		__DATA.__la_symbol_ptr	(Lazy Symbol Ptrs)
	Mem: 0x0010034a0-0x0010598a0		__DATA.__const	
	Mem: 0x0010598a0-0x0010aca80		__DATA.__cfstring	
	Mem: 0x0010aca80-0x0010b1500		__DATA.__objc_classlist	(Normal)
	Mem: 0x0010b1500-0x0010b18c8		__DATA.__objc_catlist	(Normal)
	Mem: 0x0010b18c8-0x0010b2b78		__DATA.__objc_protolist	
	Mem: 0x0010b2b78-0x0010b2b80		__DATA.__objc_imageinfo	
	Mem: 0x0010b2b80-0x001331478		__DATA.__objc_const	
	Mem: 0x001331478-0x0013719f8		__DATA.__objc_selrefs	(Literal Pointers)
	Mem: 0x0013719f8-0x001371d40		__DATA.__objc_protorefs	
	Mem: 0x001371d40-0x0013764b0		__DATA.__objc_classrefs	(Normal)
	Mem: 0x0013764b0-0x001379de8		__DATA.__objc_superrefs	(Normal)
	Mem: 0x001379de8-0x001393a98		__DATA.__objc_ivar	
	Mem: 0x001393a98-0x0013c2398		__DATA.__objc_data	
	Mem: 0x0013c23a0-0x0013d0b98		__DATA.__data	
	Mem: 0x0013d0ba0-0x0013d7878		__DATA.__bss	(Zero Fill)
	Mem: 0x0013d7878-0x0013d7930		__DATA.__common	(Zero Fill)
LC 02: LC_SEGMENT_64          Mem: 0x0013d8000-0x1b3e000	__LINKEDIT
LC 03: LC_ID_DYLIB           	/System/iOSSupport/System/Library/PrivateFrameworks/UIKitCore.framework/Versions/A/UIKitCore
LC 04: LC_DYLD_INFO          
LC 05: LC_SYMTAB             
	Symbol table is at offset 0x1430710 (21169936), 110030 entries
	String table is at offset 0x15e0f3c (22941500), 5363992 bytes
LC 06: LC_DYSYMTAB           
	104063 local symbols at index     0
	 3947 external symbols at index  104063
	 2020 undefined symbols at index 108010
	   No TOC
	   No modtab
	 2771 Indirect symbols at offset 0x15de3f0

LC 07: LC_UUID               	UUID: D4DB7215-65A7-37F7-9DA6-521E50A65B
LC 08: LC_BUILD_VERSION      	Build Version:           Platform: Unknown? 12.0.0
LC 09: LC_SOURCE_VERSION     	Source Version:          3698.94.2.0.0
LC 10: LC_REEXPORT_DYLIB     	/System/Library/PrivateFrameworks/UIFoundation.framework/Versions/A/UIFoundation
LC 11: LC_LOAD_UPWARD_DYLIB  	/System/Library/PrivateFrameworks/XCTTargetBootstrap.framework/Versions/A/XCTTargetBootstrap
LC 12: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/UIKitHostAppServices.framework/Versions/A/UIKitHostAppServices
LC 13: LC_LOAD_DYLIB         	/System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices
LC 14: LC_LOAD_DYLIB         	/System/iOSSupport/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/JavaScriptCore
LC 15: LC_LOAD_DYLIB         	/System/Library/Frameworks/IOSurface.framework/Versions/A/IOSurface
LC 16: LC_LOAD_DYLIB         	/usr/lib/libobjc.A.dylib
LC 17: LC_LOAD_DYLIB         	/System/Library/Frameworks/Accelerate.framework/Versions/A/Accelerate
LC 18: LC_LOAD_DYLIB         	/System/Library/Frameworks/CFNetwork.framework/Versions/A/CFNetwork
LC 19: LC_LOAD_DYLIB         	/System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation
LC 20: LC_LOAD_DYLIB         	/System/Library/Frameworks/CoreGraphics.framework/Versions/A/CoreGraphics
LC 21: LC_LOAD_DYLIB         	/System/Library/Frameworks/CoreImage.framework/Versions/A/CoreImage
LC 22: LC_LOAD_DYLIB         	/System/Library/Frameworks/CoreText.framework/Versions/A/CoreText
LC 23: LC_LOAD_DYLIB         	/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation
LC 24: LC_LOAD_DYLIB         	/System/Library/Frameworks/IOKit.framework/Versions/A/IOKit
LC 25: LC_LOAD_DYLIB         	/System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO
LC 26: LC_LOAD_DYLIB         	/System/iOSSupport/System/Library/Frameworks/MobileCoreServices.framework/Versions/A/MobileCoreServices
LC 27: LC_LOAD_DYLIB         	/System/Library/Frameworks/QuartzCore.framework/Versions/A/QuartzCore
LC 28: LC_LOAD_DYLIB         	/System/Library/Frameworks/Security.framework/Versions/A/Security
LC 29: LC_LOAD_DYLIB         	/System/Library/Frameworks/UserNotifications.framework/Versions/A/UserNotifications
LC 30: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/AggregateDictionary.framework/Versions/A/AggregateDictionary
LC 31: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/AppSupport.framework/Versions/A/AppSupport
LC 32: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/AssertionServices.framework/Versions/A/AssertionServices
LC 33: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/BackBoardServices.framework/Versions/A/BackBoardServices
LC 34: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/BaseBoard.framework/Versions/A/BaseBoard
LC 35: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/CoreUI.framework/Versions/A/CoreUI
LC 36: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/FrontBoardServices.framework/Versions/A/FrontBoardServices
LC 37: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/GraphicsServices.framework/Versions/A/GraphicsServices
LC 38: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/MobileAsset.framework/Versions/A/MobileAsset
LC 39: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/PhysicsKit.framework/Versions/A/PhysicsKit
LC 40: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/ProofReader.framework/Versions/A/ProofReader
LC 41: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/PrototypeTools.framework/Versions/A/PrototypeTools
LC 42: LC_LOAD_DYLIB         	/System/Library/PrivateFrameworks/TextInput.framework/Versions/A/TextInput
LC 43: LC_LOAD_DYLIB         	/System/iOSSupport/System/Library/PrivateFrameworks/UIKitServices.framework/Versions/A/UIKitServices
LC 44: LC_LOAD_DYLIB         	/usr/lib/libAccessibility.dylib
LC 45: LC_LOAD_DYLIB         	/usr/lib/libMobileGestalt.dylib
LC 46: LC_LOAD_DYLIB         	/usr/lib/libicucore.A.dylib
LC 47: LC_LOAD_DYLIB         	/usr/lib/libc++.1.dylib
LC 48: LC_LOAD_DYLIB         	/usr/lib/libSystem.B.dylib
LC 49: LC_FUNCTION_STARTS    	Offset: 21067184, Size: 100160 (0x14175b0-0x142fcf0) 
LC 50: LC_DATA_IN_CODE       	Offset: 21167344, Size: 2592 (0x142fcf0-0x1430710) 
LC 51: LC_CODE_SIGNATURE     	Offset: 28305504, Size: 230464 (0x1afe860-0x1b36ca0) 
```

{% endcapture %}
{% include toggle-field.html toggle-name="toggle-thats7" button-text="Toggle Code" toggle-text=code-capture %}


> `使用前提：`有逼数大致实现是怎样的

### 查找到对应函数方法


> 1.先找相关函数函数指针
```s
  CFRunLoopAddSource

; ================ B E G I N N I N G   O F   P R O C E D U R E ================
        		imp___stubs__CFRunLoopAddObserver:
0000000000d48448         jmp        qword [_CFRunLoopAddObserver_ptr]           
                        ; endp
						
```
![image.png]({{ site.img_url }}imgs/iOS/mach-o-stub.png)

> 2. 右键查找Reference To
![image.png]({{ site.img_url }}imgs/iOS/mach-o-find-ref.png)

> 3. 点击对应Ref
![image.png]({{ site.img_url }}imgs/iOS/mach-o-ref.png)

### CATransaction相关实现

{% capture code-capture %}

```c
int __installAfterCACommitHandler(int arg0) {
    r14 = CFRunLoopGetCurrent();
    rax = 0x0;
    var_40 = rax;
    if (*___beforeCARunLoopObserver == rax) {
            rax = CFRunLoopObserverCreate(0x0, 0xa0, 0x1, 0x1e8098, __beforeCACommitHandler, &var_40);
            *___beforeCARunLoopObserver = rax;
            CFRunLoopAddObserver(r14, rax, *_kCFRunLoopCommonModes);
            rax = CFRunLoopObserverCreate(0x0, 0xa0, 0x1, 0x1e8868, __afterCACommitHandler, &var_40);
            *___afterCARunLoopObserver = rax;
            rax = CFRunLoopAddObserver(r14, rax, *_kCFRunLoopCommonModes);
    }
    return rax;
}
```

```c
void __beforeCACommitHandler() {
    __prepareForCAFlush();
    rbx = [BKSFenceLogApp() retain];
    if (os_log_type_enabled(rbx, 0x2) != 0x0) {
            *(int16_t *)(rsp + 0xfffffffffffffff0) = 0x0;
            _os_log_impl(0xfffffffffffe362f, rbx, 0x2, "_beforeCACommitHandler", rsp + 0xfffffffffffffff0, 0x2);
    }
    [rbx release];
    return;
}
```

```c
void __afterCACommitHandler() {
    r14 = rdx;
    rbx = [BKSFenceLogApp() retain];
    if (os_log_type_enabled(rbx, 0x2) != 0x0) {
            *(int16_t *)(rsp + 0xfffffffffffffff0) = 0x0;
            _os_log_impl(0xfffffffffffe3324, rbx, 0x2, "_afterCACommitHandler", rsp + 0xfffffffffffffff0, 0x2);
            rsp = rsp;
    }
    [rbx release];
    r14 = [r14 retain];
    if (__cleanUpAfterCAFlushAndRunDeferredBlocks() != 0x0) {
            rbx = [BKSFenceLogApp() retain];
            if (os_log_type_enabled(rbx, 0x2) != 0x0) {
                    *(int16_t *)(rsp + 0xfffffffffffffff0) = 0x0;
                    _os_log_impl(0xfffffffffffe32a2, rbx, 0x2, "_beforeCACommitHandler2", rsp + 0xfffffffffffffff0, 0x2);
                    rsp = rsp;
            }
            [rbx release];
            __prepareForCAFlush();
            [CATransaction flush];
            __cleanUpAfterCAFlushAndRunDeferredBlocks();
            rbx = [BKSFenceLogApp() retain];
            if (os_log_type_enabled(rbx, 0x2) != 0x0) {
                    *(int16_t *)(rsp + 0xfffffffffffffff0) = 0x0;
                    _os_log_impl(0xfffffffffffe321b, rbx, 0x2, "_afterCACommitHandler2", rsp + 0xfffffffffffffff0, 0x2);
            }
            [rbx release];
    }
    [r14 release];
    return;
}
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats5" button-text="Toggle Code" toggle-text=code-capture %}

**-[UApplication _run]**

{% capture code-capture %}

``` 
void -[UIApplication _run](void * self, void * _cmd) {
    rbx = self;
    r14 = objc_autoreleasePoolPush();
    kdebug_trace(0x2b8700d8, 0x3, 0x0, 0x0, 0x0);
    __UIAccessibilityInitialize();
    kdebug_trace(0x2b8700dc, 0x3, 0x0, 0x0, 0x0);
    if (os_variant_has_internal_content("com.apple.UIKit") != 0x0) {
            XCTTargetBootstrap();
    }
    [rbx _registerForUserDefaultsChanges];
    [rbx _registerForSignificantTimeChangeNotification];
    [rbx _registerForLanguageChangedNotification];
    [rbx _registerForLocaleWillChangeNotification];
    [rbx _registerForLocaleChangedNotification];
    [rbx _registerForAlertItemStateChangeNotification];
    [rbx _registerForKeyBagLockStatusNotification];
    [rbx _registerForNameLayerTreeNotification];
    [rbx _registerForBackgroundRefreshStatusChangedNotification];
    [rbx _registerForHangTracerEnabledStateChangedNotification];
    __installAfterCACommitHandler(rbx);  /// 注册CATransaction处理
    [rbx _installAutoreleasePoolsIfNecessaryForMode:*_kCFRunLoopDefaultMode]; /// 注册AutoreleasePool处理
    [rbx->_eventDispatcher _installEventRunLoopSources:CFRunLoopGetMain()]; // 注册事件处理
    if (([[rbx class] registerAsSystemApp] == 0x0) && ([rbx isFrontBoard] == 0x0)) {
            if ([[rbx class] _isSystemUIService] != 0x0) {
                    var_30 = *___workspace;
                    r13 = [[[rbx class] _systemUIServiceIdentifier] retain];
                    rbx = [[[rbx class] _systemUIServiceClientSettings] retain];
                    [var_30 requestSceneCreationWithIdentifier:r13 initialClientSettings:rbx completion:^ {/* block implemented at ___21-[UIApplication _run]_block_invoke */ } }];
                    [rbx release];
                    [r13 release];
            }
            else {
                    r15 = [rbx class];
                    if (*__UIApplicationIsExtension.once != 0xffffffffffffffff) {
                            dispatch_once(__UIApplicationIsExtension.once, ^ {/* block implemented at ____UIApplicationIsExtension_block_invoke */ } });
                    }
                    if ((*(int8_t *)__UIApplicationIsExtension.result != 0x0) && ([r15 _wantsApplicationBehaviorAsExtension] == 0x0)) {
                            *(int8_t *)___calledRunWithMainScene = 0x1;
                            [rbx __completeAndRunAsPlugin];
                    }
            }
    }
    else {
            rbx->_eventDispatcher->_mainEnvironment->_isSystemApplication = 0x1;
            *(int8_t *)___calledRunWithMainScene = 0x1;
    }
    objc_autoreleasePoolPop(r14);
    kdebug_trace(0x2b870168, 0x33, 0x0, 0x0, 0x0);
    GSEventRun();
    return;
}
```

{% endcapture %}

{% include toggle-field.html toggle-name="toggle-thats6" button-text="Toggle Code" toggle-text=code-capture %}

> `CFRunLoopAddSource` 在`CoreFoundation`动态库中，所以是`__TEXT.__stubs` Section

> 除了`CATransaction`之外，还可以看到`AutoReleasePool`以及`UIEventDispatcher`之类相关的处理都是类似的监听。

## bugly 热修复实现




# 引用

[mach-o 掘金](https://juejin.im/post/5ab47ca1518825611a406a39) **偏向于oc相关section**

[parse-mach-o files](https://lowlevelbits.org/parsing-mach-o-files/) **偏向于加载过程**

[objc.io mach-o](https://www.objc.io/issues/6-build-tools/mach-o-executables/)

[mach-o file format](https://github.com/aidansteele/osx-abi-macho-file-format-reference)

[认识macho](https://www.jianshu.com/p/c07e5ee89b3e)

[dyld](http://newosxbook.com/articles/DYLD.html)

[wwdc dyld 2016](https://developer.apple.com/videos/play/wwdc2016/406/)

[wwdc dyld 2017](https://developer.apple.com/videos/play/wwdc2017/413/)

[wwdc dyld 2018](https://developer.apple.com/videos/play/wwdc2018/702/)

[mach o load](https://www.blackhat.com/presentations/bh-dc-09/Iozzo/BlackHat-DC-09-Iozzo-Macho-on-the-fly.pdf)

[macho js](https://www.jianshu.com/p/4ab0e06c5ec9)
<!--
 。。。
-->
