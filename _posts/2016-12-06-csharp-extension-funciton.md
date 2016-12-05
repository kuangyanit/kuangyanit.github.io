---
layout: post
title: 一道在知乎很火的 Java 题——如何输出 ab
categories: Java
description: 一道在知乎讨论得很火热的 Java 题，网友们的脑洞能给出一些什么样的答案呢？
keywords: Java
---

这是一个源自知乎的话题，原贴链接：[一道百度的面试题，有大神会嘛？](https://www.zhihu.com/question/50801791)

虽然我不是大神，但我也点进去看了一下，思考了一会之后有了一些思路，然后去看其它人的答案的时候果然全都已经被各路大神们先想到并贴出来了，所以我就不去凑热闹写重复答案了，整理一下网友们的智慧在这里自娱自乐好了。

## 题目

![java-output-ab](/images/posts/java/output-ab.jpg)

## 思路一

作为一个多年前也见过不少笔试题的少年，看到这个题目的第一想法是脑筋急转弯——注入一段逻辑直接改变原 if 结构。

### 解法一

填入内容 `true){System.out.print("a");}if(false`。

```java
public void print() {
    if (true) {
        System.out.print("a");
    }

    if (false) {
        System.out.print("a");
    } else {
        System.out.print("b");
    }
}
```

类似地也可以填入 `true){System.out.print("ab");return;}if(false` 等。

当初大学时单纯的少年可是很难想出这样的套路的，时间改变了我们啊。

## 思路二

如果正经遵从题目的原代码结构，那就得想办法构造一段代码，既能输出 `a`，又能返回 `false`。

### 解法二

我也想到能否使用 `System.out.print` 的返回值来做文章，但奈何并不记得它返回什么，首先让我们复习一下 `PrintStream` 的 `print`、`println` 和 `printf` 方法的区别：

| 方法    | 功能               | 返回值      |
|---------|--------------------|-------------|
| print   | 打印一个值或者对象 | void        |
| println | 打印并换行         | void        |
| printf  | 格式化打印         | PrintStream |

所以适用的是 `printf`，它的返回值是 `PrintStream` 类型的 `System.out`，判它是否为空即可。

填入内容 `System.out.printf("a") == null`。

```java
public void print() {
    if (System.out.printf("a") == null) {
        System.out.print("a");
    } else {
        System.out.print("b");
    }
}
```

经测试填入 `System.out.append("a") == null` 也是可以达到效果的。

### 解法三

仍然是思路二，但从匿名内部类来作文章。

实现代码：

```java
public void print() {
    if (new Object() {
        boolean print() {
            System.out.print("a");
            return false;
        }
    }.print()) {
        System.out.print("a");
    } else {
        System.out.print("b");
    }
}
```

这里利用的知识点是匿名内部类可以声明基类没有的新方法并且马上调用。

### 解法四

使用 Java 8 里的 lambda 来实现思路二。

```java
public void print() {
    if (((BooleanSupplier)(() -> {System.out.print("a");return false;})).getAsBoolean()) {
        System.out.print("a");
    } else {
        System.out.print("b");
    }
}
```

严格来讲这个不一定能算作正确答案，因为要增加 `import java.util.function.BooleanSupplier;`。

## 脑洞大开

讲完严肃的解法，来看看网友 [穷小子](https://www.zhihu.com/people/qiong-xiao-zi-158) 开脑洞的思路：

```java
public void print() {
//    if ( ) {
        System.out.print("a");
//    } else {
        System.out.print("b");
//    }
}
```
今天在学习Linq的基础知识的时候遇到这么一个问题:

```cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace DemoLinq
{
    class Program
    {
        public static LinqData dataForTest = new LinqData { Name = "lsb", Score = "85", Age = 15 };
        static void Main(string[] args)
        {
            List<LinqData> linqDatas = new List<LinqData>();
            linqDatas.Add(new LinqData() { Name = "zsgg", Score = "98", Age = 12 }); //这里使用了对象初始化器
            linqDatas.Add(new LinqData() { Name = "lsb", Score = "85", Age = 15 });
            linqDatas.Add(new LinqData() { Name = "ww", Score = "87", Age = 15 });
            linqDatas.Add(new LinqData() { Name = "zd", Score = "85", Age = 18 });
            int lastIndex=linqDatas.LastIndexOf(dataForTest);
            Console.WriteLine(lastIndex);
        }

    }

    public class LinqData
    {
        public string Name { get; set; }
        public string Score { get; set; }
        public int Age { get; set; }

    }


}

```
这里用List<T>类内置的LastIndexOf方法来求取对象的下标时发现其下标一直为-1（即未在linqDatas集合中找到dataForTest对象），可是该集合中明明包含dataForTest对象的，这是为什么呢？

经过群里好友的提醒我明白了，原来是LastIndexOf方法内置的比较逻辑无法判别出两个LinqData对象是相等的，故而无法找到dataForTest对象，所以返回-1。那么该怎么办呢？我的想法是对LastIndexOf方法进行扩展。故而有了以下的代码：


```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace DemoLinq
{
    class Program
    {
        public static LinqData dataForTest = new LinqData { Name = "lsb", Score = "85", Age = 15 };
        static void Main(string[] args)
        {

            List<LinqData> linqDatas = new List<LinqData>();
            linqDatas.Add(new LinqData() { Name = "zsgg", Score = "98", Age = 12 }); //这里使用了对象初始化器
            linqDatas.Add(new LinqData() { Name = "lsb", Score = "85", Age = 15 });
            linqDatas.Add(new LinqData() { Name = "ww", Score = "87", Age = 15 });
            linqDatas.Add(new LinqData() { Name = "zd", Score = "85", Age = 18 });
	        int lastIndex = linqDatas.LastIndexOf(dataForTest); 
            Console.WriteLine(lastIndex);
        }


    }

    public class LinqData:IEquatable<LinqData>  //自定义类需实现IEquatable<T>接口
    {
        public string Name { get; set; }
        public string Score { get; set; }
        public int Age { get; set; }

        public bool Equals(LinqData other)
        { 
           return Name.Equals(other.Name) && Score.Equals(other.Score) && Age.Equals(other.Age);
        }
    }



    public static class Extend
    {
       
        public static int MyLastIndexOf<TSource>(this IEnumerable<TSource> source, TSource item)  where TSource:IEquatable<TSource>
        {
            List<TSource> data = source.ToList();
            int index = -1;
            foreach (TSource t in data)
            {
                if (t.Equals(item))
                {
                    index = data.Count - 1 - data.IndexOf(t);
                }
            }
            return index;
        }

      
    }
}

```
主要思路如下：

* 自定义类实现IEquatable<T>接口（即实现Equals方法来定义判等规则）
* 编写扩展方法MyLastIndexOf（注意扩展方法的参数，在这里为this IEnumerable<TSource> source与TSource item）
* 对扩展方法添加泛型约束（where TSource:IEquatable<TSource>）
