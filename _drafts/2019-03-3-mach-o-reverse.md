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

## bugly 热修复实现

## UIKit CoreAnimation [CATransaction commit] 时机


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
