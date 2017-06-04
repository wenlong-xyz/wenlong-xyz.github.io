---
title: 设计模式学习笔记-1 工厂模式
category: Technology
toc: true
date: 2016-11-13 15:54:28
tags: [设计模式]
---
&emsp;&emsp;当谈及创建对象时，我们往往会想到new操作符。但当我们的业务代码里充斥着大量的new的同时，代码之间的耦合度往往会变得越来越高，彼此之间的依赖错综复杂。当业务发生变更时，原有代码的拓展会很困难。工厂模式则是一种帮助我们实现“解耦”的方法。

>以下内容主要参考自《Head First设计模式》，一下简称HF书

# 问题重述
## Version 0
```java
public Pizza orderPizza(String type) {
        Pizza pizza;

        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        }
        
        // use pizza to do somtheing else:
        // pizza.prepare();
        // pizza.bake()l
        // ...
    }
```
&emsp;&emsp;这是HF书提出的一个例子，Pizza店。对象的创建需要在运行时才能确定，当多个地方需要这样动态创建对象时，我们需要将这段代码嵌入到多个地方。当我们新增pizza或者删除pizza是，不可避免的，我们需要在多个地方修改这段业务代码，当然这个过程会很痛苦。
## Version 1 简单工厂
&emsp;&emsp;如果我们稍微有点编程经验，我们就会想到，将创建对象的那部分代码拿出来，构成一个公共方法。这样大家就都可以使用了。为了更加规范一点，我们将其放到一个类中，这个类中有一个可以创建Pizza对象的方法。而这个类就被成为工厂类。
```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        return pizza;
    }
}
```
注：上述方法可以改成静态方法，使其成为静态工厂。
# 扩展1 - 工厂方法
&emsp;&emsp;如果将一家Pizza店拓展为多家。而每家可以制作的Pizza不同，那又该洞天的创建Pizza对象。最直观的方法可能是创建多个Factory，每个Facotry对应一家店。然后根据店的不同来使用不同的工厂，进而创建不同出Pizza。难道我们的代码要这个样子？
```java

```

