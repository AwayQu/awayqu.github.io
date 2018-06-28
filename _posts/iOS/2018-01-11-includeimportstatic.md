---
layout: post
title:  "include & import & static"
date:   2018-1-11 02:02:00 +0800
categories: iOS
tags: [iOS, c]
---

* TOC
{:toc}

# include

> 按顺序扩展整个头文件，所有的头文件， `重复头文件会多次展开`

## 示例1

**`TestStatic.h`**

```c

#ifndef TestStatic_h
#define TestStatic_h

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";

#endif /* TestStatic_h */

```

**`TestHeader2.h`**

```c
#ifndef TestHeader2_h
#define TestHeader2_h

// 包含头文件
#include "TestStatic.h"

#endif /* TestHeader2_h */
```

**`main.m`**

```c
#import <Foundation/Foundation.h>
#include "TestStatic.h"
#include "TestHeader2.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}
```

## 预编译结果

**`main.m` preprocess**

```c

// Preprocessed output for main.m
// Generated at 01:32:54 on Thursday, January 11, 2018
// Using Debug configuration, x86_64 architecture for StaticExternDemo target of StaticExternDemo project

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 344 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2

@import Foundation; /* clang -E: implicit import for "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" */
# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1
# 12 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h"
NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 11 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/TestHeader2.h" 1
# 12 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}


```



## 示例2



**`TestStatic.h`**

```c

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";

#ifndef TestStatic_h
#define TestStatic_h


#endif /* TestStatic_h */

```

**`TestHeader2.h`**

```c
#ifndef TestHeader2_h
#define TestHeader2_h

// 包含了头文件
#include "TestStatic.h"

#endif /* TestHeader2_h */
```

## 预编译结果

**`main.m` preprocess** 

> 会报 `Redefinition`

```c
// Preprocessed output for main.m
// Generated at 01:39:13 on Thursday, January 11, 2018
// Using Debug configuration, x86_64 architecture for StaticExternDemo target of StaticExternDemo project

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 344 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2

@import Foundation; /* clang -E: implicit import for "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" */
# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 11 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/TestHeader2.h" 1

# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 10 "/Users/away/IOSProject/StaticExternDemo/TestHeader2.h" 2
# 12 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}

```


# import

> 先处理头文件依赖关系，`重复头文件只展开一次`


## 示例3

**`TestStatic.h`**

```c

#ifndef TestStatic_h
#define TestStatic_h

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";

#endif /* TestStatic_h */

```

**`TestHeader1.h`**

```c
#ifndef TestHeader1_h
#define TestHeader1_h

// 导入头文件
#import "TestStatic.h"

#endif /* TestHeader1_h */
```

**`main.m`**

```c
#import <Foundation/Foundation.h>
// 导入
#import "TestStatic.h"
#import "TestHeader1.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}
```

## 预编译结果

**`main.m` preprocess**

```c
// Preprocessed output for main.m
// Generated at 01:47:19 on Thursday, January 11, 2018
// Using Debug configuration, x86_64 architecture for StaticExternDemo target of StaticExternDemo project

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 344 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2

@import Foundation; /* clang -E: implicit import for "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" */
# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 11 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/TestHeader1.h" 1
# 12 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}

```


# 多个源文件中（.c、.mm）include & import

## 示例3


**`main.m`**

```c
#import <Foundation/Foundation.h>
// 导入
#import "TestStatic.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}
```
**`ViewController.m`**
```objc
#import "ViewController.h"
#import "TestStatic.h"
@interface ViewController ()
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do view setup here.
}

@end
```

## 预编译结果

**`main.m` preprocess**

```c
// Preprocessed output for main.m
// Generated at 01:47:19 on Thursday, January 11, 2018
// Using Debug configuration, x86_64 architecture for StaticExternDemo target of StaticExternDemo project

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 344 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2

@import Foundation; /* clang -E: implicit import for "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" */
# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 11 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/TestHeader1.h" 1
# 12 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/main.m" 2
int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSLog(@"Hello, World!");
        NSLog(noPrefix);
        NSLog(staticPrefix);
    }
    return 0;
}

```

**`ViewController.m` preprocess**

```c
// Preprocessed output for ViewController.m
// Generated at 02:03:57 on Thursday, January 11, 2018
// Using Debug configuration, x86_64 architecture for StaticExternDemo target of StaticExternDemo project

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/ViewController.m"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 344 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/ViewController.m" 2

# 1 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/ViewController.h" 1

@import Cocoa; /* clang -E: implicit import for "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Cocoa.framework/Headers/Cocoa.h" */

@interface ViewController : NSViewController

@end
# 10 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/ViewController.m" 2
# 1 "/Users/away/IOSProject/StaticExternDemo/TestStatic.h" 1

NSString * const noPrefix = @"noPrefix";

static NSString * const staticPrefix = @"staticPrefix";
# 11 "/Users/away/IOSProject/StaticExternDemo/StaticExternDemo/ViewController.m" 2
@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

}
@end

```

**`报错`**

```shell
duplicate symbol _noPrefix in:
    /Users/away/Library/Developer/Xcode/DerivedData/StaticExternDemo-cagxovqlkhtubxandrffxkswjrup/Build/Intermediates.noindex/StaticExternDemo.build/Debug/StaticExternDemo.build/Objects-normal/x86_64/ViewController.o
    /Users/away/Library/Developer/Xcode/DerivedData/StaticExternDemo-cagxovqlkhtubxandrffxkswjrup/Build/Intermediates.noindex/StaticExternDemo.build/Debug/StaticExternDemo.build/Objects-normal/x86_64/main.o
ld: 1 duplicate symbol for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

# Assembly Section cfstring

> 不会有多余的内存占用

```s
; Section __cfstring
; Range: [0x100001050; 0x1000010b0[ (96 bytes)
; File offset : [4176; 4272[ (96 bytes)
;   S_REGULAR

cfstring_staticPrefix:
0000000100001050         dq         ___CFConstantStringClassReference, 0x7c8, 0x100000f64, 0xc ; "staticPrefix", DATA XREF=-[ViewController viewDidLoad]+71, _main+65
cfstring_noPrefix:
0000000100001070         dq         ___CFConstantStringClassReference, 0x7c8, 0x100000f71, 0x8 ; "noPrefix", DATA XREF=-[ViewController viewDidLoad]+54, _main+48
cfstring_Hello__World_:
0000000100001090         dq         ___CFConstantStringClassReference, 0x7c8, 0x100000f7a, 0xd ; "Hello, World!", DATA XREF=_main+27
```

# 总结
* `#ifndef #define #endif` 套路对于使用 `#import` 而言没有意义
* 反正加上`static`就对了