---
layout: post
title: 尝试扩展List<T>类中的LastIndexOf方法
categories: C#
description: 尝试扩展List<T>类中的LastIndexOf方法
keywords: C#
---

在学习Linq的基础知识的时候遇到这么一个问题:

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
