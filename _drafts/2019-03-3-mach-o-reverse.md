---
layout: post
title:  "mach-o & 逆向"
date:   2019-03-03 21:53:01 +0800
categories: iOS
tags: [iOS, clang]
---


* TOC
{:toc}

# mach-o 文件结构

** 胖mach-o**
```
.
├── Fat Header
├── Mach-O-armv7
└── Mach-O-x86_64
```

```
.
└── Mach-O-x86_64
    ├── Header
    ├── Load\ Commands
    │   ├── Segment\ Commnad\ 1
    │   └── Segment\ Commnad\ 2
    └── Data
        ├── Segment\ 1
        │   ├── Section\ 1\ Data
        │   ├── Section\ 2\ Data
        │   └── Section\ 3\ Data
        └── Segment\ 2
            ├── Section\ 1\ Data
            ├── Section\ 2\ Data
            ├── Section\ 3\ Data
            └── Section\ 4\ Data
```

## Header 定义

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

[mach-o 掘金](https://juejin.im/post/5ab47ca1518825611a406a39) **偏向于oc相关section**
[parse-mach-o files](https://lowlevelbits.org/parsing-mach-o-files/) **偏向于加载过程**
[objc.io mach-o](https://www.objc.io/issues/6-build-tools/mach-o-executables/)
[mach-o file format](https://github.com/aidansteele/osx-abi-macho-file-format-reference)