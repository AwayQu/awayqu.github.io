
# SICP 读书笔记

> 为啥读这书呢, 因为 `黑客与画家` 的 作者 一直在吹`Lisp`才是最纯粹的`lambda演算` 实现, 以前学过 `Haskell`, 所以想看看`Lisp`感受对比一把.


# 1.1 Building Abstractions with Functions

> 第一章主要介绍了 `函数`
  顺便扯了点名词`Statements & Expressions`,`Functions`,`Objects`,`Interpreters`

**对我而言比较新奇的是区分了声明和表达**


`Expressions`: Compute some value

> 举了例子是`py`的 `递推式`

```python
{w for w in words if len(w) == 6 and w[::-1] in words}
```


`Statements`: Carry out some action

> 举了例子是两个 `assignment statement`

```python
shakespeare = urlopen('http://composingprograms.com/shakespeare.txt')
words = set(shakespeare.read().decode().split())
```

> 额,道理是有道理,毛估估也感受到了一些区别吧,暂时没体会到有啥子用.


> 扯了一句 还是很有意思的 **`computer = powerful + stupid`Programming is about a person using their real insight to build something useful, constructed out of these teeny, simple little operations that the computer can do.**


[sicp](https://cs61a.org/)