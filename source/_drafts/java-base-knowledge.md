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
        // 自定义方法
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

2. **hashCode方法**
&emsp;&emsp;返回该对象的哈希码值，一般默认返回对象的内存地址，例如在Map中，会利用该整数值、选择合适的哈希函数进行最终哈希地址的计算。所以从逻辑上将，如果两个对象相等，其hashcode也应该相等，反之则不一定。故而，hashCode 与 equals方法通常同时重写，而不是仅仅重写一个，否则可能导致逻辑上的错误。常见的重写方法：每个成员的hashCode与素数相乘相加。

    ```java
    public static int hashCode(long a[]) {
        if (a == null)
            return 0;

        int result = 1;
        for (long element : a) {
            int elementHash = (int)(element ^ (element >>> 32));
            result = 31 * result + elementHash;
        }

        return result;
    }
    ```

3. **toString方法**: 返回该对象的字符串表示，主要用于更好的输出对象内容。
4. **wait, notify, notifyAll**: [等待与唤醒](https://github.com/ZongWenlong/JavaLab/tree/master/src/main/java/multithread/tools#自定义同步工具)
5. **clone方法**：返回对象的副本
    * 问题：默认为“选择性”拷贝，不是真正的深拷贝
        * ``obj.clone() != obj  //  √ ``
        * 基本类型：如果变量是基本很类型，则拷贝其值，比如int、float等。
        * 对象：如果变量是一个实例对象，则拷贝其地址引用，也就是说此时新对象与原来对象是公用该实例变量。
        * 包装类：类似于String 是传值还是传引用的情况，拷贝后的值如果修改，则会生成新字符串，变更的是拷贝对象的属性引用值，而不会变更原对象
    * 解决方案：
        1. 递归式一层一层clone
        2. 先序列化再反序列化，根本也是一层一层clone
    * Reference:
        - [java提高篇（五）-----使用序列化实现对象的拷贝](http://blog.csdn.net/chenssy/article/details/12952063)
6. **finalize**：当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。子类重写 finalize 方法，以配置系统资源或执行其他清除，但由于垃圾回收时间不可控等原因，此方法调用时间甚至是否被调用不可控，而且可能会带来性能问题，此方法应避免使用，通常用finally代码块代替
7. **getClass方法**：返回一个对象的运行时类

## 作用域范围
| 修饰词         |公开|子类 | 同一包内 | 私有 |
| ------------- |:-----:|:-----:|:-----:|:-----:|
| public        | √ | √ | √ | √ |
| protected     | × | √ | √ | √ |
| 无修饰词（默认）| × | × | √ | √ |
| private       | × | × | × | √ |

## 继承与覆盖注意点
1. **构造函数**
    * 父类无构造函数或者一个无参数构造函数，子类若无构造函数或者有无参数构造函数，子类构造函数中不需要显式调用父类的构造函数，系统会自动在调用子类构造函数前调用父类的构造函数
    * 父类只有有参数构造函数，子类在构造方法中必须要显示调用父类的构造函数，否则编译出错
    * 父类既有无参数构造函数，也有有参构造函数，子类可以不在构造方法中调用父类的构造函数，这时使用的是父类的无参数构造函数
2. **普通方法**
    * 子类覆盖父类的方法，必须有同样的参数返回类型，否则编译不能通过
    * 子类覆盖父类的方法，在jdk1.5后，参数返回类可以是父类方法返回类的子类
    * 子类覆盖父类方法，可以修改方法作用域修饰符，但只能把方法的作用域放大，而不能把public修改为private
    * 子类的静态方法不能隐藏同名的父类实例方法
3. **成员属性**
    * 当子类覆盖父类的成员变量时，父类方法使用的是父类的成员变量，子类方法使用的是子类的成员变量
4. Reference: [java继承覆盖总结](http://blog.csdn.net/stonecao/article/details/6317353)

## API使用
1. **Map遍历**: 以 Map<Integer,Integer> 为例，四种方式
    ```java
    for (int key : map.keySet()) {}

    Iterator<Map.Entry<Integer, Integer>> it = map.entrySet().iterator();
    while(it.hasNext()) {
        Map.Entry<Integer, Integer> cur = it.next();
    }

    for (Map.Entry<Integer, Integer> e : map.entrySet()) {
        int key = e.getKey();
        int value = e.getValue();
    }

    for (int v : map.values()) {}
    ```

