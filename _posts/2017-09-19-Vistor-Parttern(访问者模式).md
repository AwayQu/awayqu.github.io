---
layout: post
title:  "Visitor Pattern(访问者模式)"
date:   2017-09-19 00:48:22 +0800
categories: designparttern
---

# Visitor

## 接口
```
class interface Visitor {
  public visitor(Node node);
}

class interface Node {
  public accpet(Visitor visitor);
}
```

## 作用
解耦了 数据结构 和 行为


# 示例

## 商店和客户
```
public class Merchant implements Node {


    @Override
    public void accept(Visitor v) {
        v.visit(this);
    }

    public void orderGoodsA() {
        System.out.println("order Goods A");
    }

    public void orderGoodsB() {
        System.out.println("order Goods B");
    }

}

public class CustomerA implements Visitor {

    @Override
    public void visit(Node n) {
        if (n instanceof Merchant) {
            ((Merchant)n).orderGoodsA();
        }
    }
}


public class CustomerB implements Visitor {

    @Override
    public void visit(Node n) {
        if (n instanceof Merchant) {
            ((Merchant)n).orderGoodsB();
        }
    }
}

```

## 调用和输出
```
@Test
public void visitorTest() throws Exception {
    Merchant m = new Merchant();
    CustomerA customerA = new CustomerA();
    CustomerB customerB = new CustomerB();

    m.accept(customerA);
    m.accept(customerB);
}
```
```shell
> order Goods A
> order Goods B
```

# 访问文件目录树
## 
## 调用和输出
```
@Test
public void visitorDirectoryTreeTest() throws Exception {

    FileVisitor visitor = new FileVisitor() {
        private int depth = 0;

        @Override
        public void visitorChildren(FileNode n) {
            depth++;
            File file = n.getFile();
            StringBuilder sb = new StringBuilder();
            for (int i = 1; i < depth; i++) {
                sb.append("---");
            }
            sb.append(file.getName());
            System.out.println(sb.toString());


            if (file.isDirectory()) {

                for (File f : file.listFiles()) {
                    FileNode node = new FileNode();
                    node.setFile(f);
                    node.accept(this);
                }
            }

            depth--;
        }

        @Override
        public void visit(Node n) {
            if (n instanceof FileNode) {
                this.visitorChildren((FileNode) n);
            }
        }
    };

    FileNode root = new FileNode();
    root.setFile(new File("./test/res/parent"));
    root.accept(visitor);

}
```
```shell
> parent
> |--child1
> |-----file1
> |--child2
> |-----file2
> |--file
```

## 实际的使用

主要还是行为和数据结构的分离, 针对一个数据结构, 通过统一的接口实现不同的操作或者行为.

如上面的示例, 对目录树,进行深度优先遍历, 同样可以使用同样的接口,进行广度优先遍历,实现对应的行为.

另外这种模式的使用,主要在编译器的语法分析器的实现中比较常见, 对于同一语法树, 实现不同的功能.

如[clang的输出dot的CallGraph实现](https://github.com/llvm-mirror/clang/blob/master/lib/Analysis/CallGraph.cpp)以及[ANTLR4](https://github.com/antlr/antlr4).

--------
<br/>
[示例代码链接](https://github.com/AwayQuEM/blogSampleCode/tree/master/antlr)