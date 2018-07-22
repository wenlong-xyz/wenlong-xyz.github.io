---
title: C# Ramp Up
category: Technology
toc: true
date: 2018-05-01 17:19:29
tags: [C#, 编程语言]
---

&emsp;&emsp;因为之前有Java基础，在学习C#的时候，就容易了许多。本文会记录一些学习和使用C#过程中所遇到的一些问题。持续更新~
* [容器类 Collections](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/collections)
* **Struct vs Class**
    * 主要的区别在于`Reference type` 和 `Value type` 的区别, **Struct is value type and Class is Reference type.**
    * 细节可以参考 [Choosing Between Class and Struct](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct)
* **yield**
    * `yield return` 和 `yield break`, 利用状态机保存迭代状态, `yield return`返回一次迭代结果, `yield break`退出迭代
    * [yield Reference](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield)
* **Nullable Types**
    * int?
    * [Nullable Types](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/nullable-types/)
* **异步编程（Asynchronous Programming）**
    * 基本使用
        * CPU-bound: Prefer multithread
        * IO-bound: Prefer `async` and `await`
        * [Reference](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
    * 深入理解
        * 简单说来，`async` 和 `await`两个关键字并不会创建新线程，而是以状态机的形式保存code执行状态，并释放控制权（返回Task对象，用于后续管理），以同步的代码风格实现异步的执行逻辑。
        * 推荐阅读：[Dissecting the async methods in C#](https://blogs.msdn.microsoft.com/seteplia/2017/11/30/dissecting-the-async-methods-in-c/)