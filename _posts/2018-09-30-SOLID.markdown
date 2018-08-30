---

layout: post
title: SOLID
date: 2018-08-30
categories: SOLID
description: 面象设计的五大原则S.O.I.D

---
	

### 五原则
* Single responsibility principle
* open/closed principle
* Liskov substitution principle
* Interface segregation principle
* Dependency inversion principle

 S.O.L.I.D 五大原则首写字母，每个字母代表一个原则，是面象对象开发的重要原则。遵循这五个原则，使我们的代码更加高内聚低偶合，可读可维护性强。使我们的代码对需求变化更加友好。

#### Single responsibility principle 

![single responsibility principle]({{ "assets/images/solid-single-responsibility.png" | absolute_url }})

单个类应该只承担一种职责，做好一件事，把负责不同功能职责的代码分离到不同的类中。

当需求变化时，反映在功能职责上的变化，一种职责的变化代表一个维度上的变化。如果多种职责的代码偶合到一个类里，使得代码在单个维度上的变化会有多个维度的影响，使变化带来的修改变得复杂，不可维护。现实中，偶合度高的变化是我们良好的预先设计崩溃的一个重要原因，高偶合使得我们的设计变得十分脆弱。

举个例子:
类：Rectangle有两个方法:draw()画图方法；area()计算面积方法，应用程序即可以调用Rectangle类完成界面画图功能，也可以调用Rctangle类计算面积。如下图所示:

![rectangle]({{ "assets/images/solid-single-responsibility-rectangle.png" | absolute_url }})

两个不同的需求职责都由Rectangle类完成，这就违反了Single responsibility principle，画图和计算面积两种不同的需求职责的变化频率可以预计是不一样的，最终会引入高偶合的代码。好的设计应该把两种职责分离开来.

![rectangle-good]({{ "assets/images/solid-single-responsibility-rectangle-good.png" | absolute_url }})

计算面积的职责由Rectangle完成，画图的职职由GeometricRectangle类完成。

怎样么定义职责?

![rectangle-what]({{ "assets/images/solid-single-responsibility-what.png" | absolute_url }})

在单一职责原则里，职责的定义是“变化的原因”。如果你有多于一个动机去修改一个类，那么这个类就是多于一个职责。