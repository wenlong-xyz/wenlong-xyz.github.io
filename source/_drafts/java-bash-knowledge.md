---
title: Java 基础知识点
date: 2017-02-13 20:31:37
category: Technology
toc: true
tags: Java
---
&emsp;&emsp;记录一些Java容易理解错误的基础知识点
## Java 部分Package介绍
1. **java.lang**: 包含Java基础的设计类：字符串、数学以及基础的线程和进程运行支持 
2. **java.util**: 包含：容器框架、格式化输入输出、数组操纵工具类、时间模型、日期和时间工具类、国际化、以及其他工具类

Reference：
> [Java document](http://docs.oracle.com/javase/8/docs/technotes/guides/lang/index.html)

## Object类方法介绍
1. **equals方法**
&emsp;&emsp;测试对象是否与另一个相等，所谓“相等”,是人为定义的逻辑“相等”。其实现既可以是默认的引用是否相等，也可以是对象中某个属性是否相等。总之，可以根据实际需求定义“相等”。
```java
  // 默认方法
  public boolean equals(Object obj) {
    return (this == obj);
  }
```

```java
  public boolean equals(Object otherObj){
        //快速测试是否是同一个对象
        if(this == otherObj) return true;
        //如果显式参数为null，必须返回false
        if(otherObj == null) reutrn false;
        //如果类不匹配，就不可能相等
        if(getClass() != otherObj.getClass()) return false;
        
        //现在已经知道otherObj是个非空的Employee对象
        Employee other = (Employee)otherObj;
        //测试所有的字段是否相等
        return name.equals(other.name)
            && salary == other.salary
            && hireDay.equals(other.hireDay);
    }
```

