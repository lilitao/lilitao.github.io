---

layout: post
title: Code Smell
date: 2018-09-20
categories: code smell
description: 识别开发过程中出的代码坏味道,重构有坏味道的代码。

--- 

## 内容介绍

Code Smell就是代码的坏味道，代码是否有坏味道是判断代码是否需要重构的重要标识之一，重构是为了让代码易于阅读和应对需求代码，易于后续的软件维护。代码坏味道主要分为以下几类：
* Bloat(膨胀)
* Change Preventers(阻挡变化)
* Dispensable(非必要的代码)
* Couplers(耦合)
* Other(其他的)

### Bloat(膨胀)
Bloat smells represent something that has grown so large that it can not be effectively handled.

膨胀代码代码已经过于肿大而难以有效地维护和阅读，膨胀代码主要有以下几种类型：

* 过长的的函数(long method)
* 长大的类(large class)
* 过长的参数列表(long parameters list)
* 基本类型偏执(primitive obsession)
* 数据泥团(data clumps)
* 过多的注释(comments)

#### 过长的函数(Long Method)

过长的函数，一般都是违反了SRP（单一职责原则），下面这个例子是截取了某个函数的一部分，从框出来的部分看，它针对不同的数据对象做了验证（validate）；像这种情况，至少应该要不同数据对象的验证放在不同的方法中实现，而不应该放在一个方法中。每个方法应该最多20行代码，再多就要考虑拆分，最好是一个方法几行代码就可以完成。

#### 过大的类(Large Class)

下面这个例子，简单地从方法返回的数据对象来看，已经有5个不一样的数据对象，这很明显违反了SRP。从这个例子来看，简单的根据数据对象拆分成不同的类，这样就可以缩小这个类。

#### 过长的参数列表(long parameter list)

方法参数越少越容易理解，一个方法的参数最多不要超过3个，如果超过就要考虑封装成一个对象，下面就是一个反面例子。

![long-parameter-list]("assets/images/code-smell-log-parameter-list.png" )

#### 基本类型偏执(Primitive Obsession)

基本类型偏执狂就是在不合适的地方使用基本类型来描述对象数据，这种情况应该把基本类型封装在对象，让代码更清晰地表达意图。

![primitive-obsession]("assets/images/code-smell-primitive-obsession.png" )

像上面这个例子，在order对象里应该直接引用Customer对象，而不是使用String对象来描述，因为这样做的话，不但阅读性不好，而且也会阻挡后面的代码变更（如果Customer需要增加属性时，这种情况会需要改动多次代码）。

#### 数据泥团(Data Clumps)

下面是一个数据泥团的例子，对于数据没有进行好的封装。应该按数据的业务含义来封装成数据对象，封装后，扩展性和灵活性会更好，而是更符合单一职责原则。

![data-clumps]("assets/images/code-smell-data-clumps.png" )

#### 过多的注释(Comments)
过多的注释就是没有必要的注释，从clean code的角度来说，除了以下注释，其他都是多余的注释。
1. 公共的JAVADOC的API，用来说明该API的使用方式；
2. 法律信息；
3. TODO信息，要做但还没有做的；
4. 对代码运行结果需要进行警告的信息；

#### 阴挡变化(Change Preventers)
阻挡变化的坏味道主要有以下几点：
* 发散式变化 (Divergent Change) ：一个类因为多个需求而要修改，其实就是违犯了单一职责原则；这时需要把职责分离，抽离出不同的类来。
* 霰弹式修改 (Shotgun Surgery) : 一个变更导致多个类要同时修改，一般是因为代码的拷贝粘贴，重复代码导致的，这时应该抽出公共类来修正这种情况。
*  平行继承体系 (Parallel Inheritance Hierarchies) ： 当你需要增加一个类的子类时，也必须为另一个类增加其子类，遇到这种情况，就要消除两个继承体系之间的引用。

#### 发散式变化(Divergent Change)
发散式变化也就是多个需求引起一个类的变化，这个是违反了单一职责原则。如下图：

![divergent-change]("assets/images/code-smell-divergent-chnage.png" )

这种情况，我们应该分离不同的职责，抽出不同的类，应该把代码重构成以下的模式：

![divergent-change]("assets/images/code-smell-divergent-change-refactoring.png" )

#### 霰弹式修改(Shotgun Surgery)
一个变更导致多个类要同时修改，一般是因为代码的拷贝粘贴，重复代码导致的。

![shotgun surgery]("assets/images/code-smell-shotgun-surgery.png"|absolute_url}})

这时应该抽出公共类来修正这种情况。

![shotgun surgery]("assets/images/code-smell-shotgun-surgery-refactoring.png"|absolute_url}})

#### 平行继承体系(Parallel Inheritance Hierarchies)

平行继承体系 (Parallel Inheritance Hierarchies) ： 当你需要增加一个类的子类时，也必须为另一个类增加其子类。

像以下例子，当你增加一个Sub A2时，也必须得增加一个Sub B2，这样就是平行继承体系，遇到这种情况，就要消除两个继承体系之间的引用。


![parallel-inheritance-hierarchies]("assets/images/code-smell-parallel-inheritance-hierarchies.png" )

#### 可有可无的代码(Dispensable)
这些都是非必要的就是过度代码，应该像清洁环境一样，删除它们。

主要的有以下几种：
* Lazy Class (冗赘类)：冗赘类，也就是说没有存在价值的类，inline需要的部分，然后删除它。
* Duplicated Code(重复代码)：就是重复代码的，应该整合后删除。
* Speculative Generality(夸夸其谈未来性)：过度设计的代码。如果没有必要，就不需要有任何多余的设计；例如：只有一个if else，那是不需要使用多态。
* Dead code(没有用的代码)：没有被调用的代码，应该删除。
* Commented-Out Code(被注释的代码)：被注释的代码，要删除掉，如果担心有以后有需要的话，版本管理工具会帮你记住他们的。

#### Couplers

主要是代码的耦合方式的坏味道，主要有以下几种：
* 依恋情结 (Feature Envy)：表明一个对象对另外一对象的数据依赖过多。
* 狎昵关系 (Inappropriate Intimacy)：两个类之间的牵联太多，你中有我，我中有你，应该尽量把有关联的方法或属性抽离出来，放在公共类中，减少关联。
* 过度耦合的消息链 (Message Chains) ： 像链子一样，一直在调用下一个对象。
* 中间人(Middle Man) ：中间人，大家也知道，一般中间人是要收费的，而代码上中间人过多，会导致信息不透明，真正的执行代码被隐藏，当发现这种情况时，需要消除中间人


##### 依恋情结 (Feature Envy)
Feature Envy ：表明一个对象对另外一对象的数据依赖过多。

像这个例子，customer使用phone对象的数据来生成号码，这种方式违反了单一职责原则。

![feature-envy]("assets/images/code-smell-feature-envy.png" )

像这种情况，应该把生成号码的职责让phone去实现，customer只需要调用phone就好了。

![feature-envy]("assets/images/code-smell-feature-envy-refactoring.png" )

##### 狎昵关系 (Inappropriate Intimacy)

两个类之间的牵联太多，你中有我，我中有你，应该尽量把有关联的方法或属性抽离出来，放在公共类中，减少关联。就是双向联系，a,b之间互相依赖.

![inappropriate-intimacy]("assets/images/code-smell-inappropriate-intimacy.png" )

应该尽量把有关联的方法或属性抽离出来，放在公共类中，减少关联。如下图所示：

![inappropriate-intimacy-refactoring]("assets/images/code-smell-inappropriate-intimacy-refactoring.png" )

##### 过度耦合的消息链 (Message Chains)
![message-chains]("assets/images/code-smell-message-chains.png" )

消息链就是上面的例子，A获取需要的数据，必要的知道B、C、D才可以，也就是知道得太多。像这种情况，应该由B作为代理直接返回A需要数据，那么A就不需要知道C、D。

##### 中间人(Middle Man) 

中间人，代码上中间人过多，会导致信息不透明，真正的执行代码被隐藏，当发现这种情况时，需要消除中间人。如下面的例子，A可以直接通过C去获取C，而不需要通过B去获取。


![middle-man]("assets/images/code-smell-middle-man.png" )

![middle-man-refactoring]("assets/images/code-smell-middle-man-refactoring.png" )

## Other

### If/Else or Switch/Case statements
在分支大于两个的情况下，应该使用多态来替换If/Else or Switch/Case结构

### Temporary Field(令人迷惑的临时字段)

临时字段，拒绝使用令人迷惑的临时字段，使用临时字段要意图清晰，给予有意义的命名。

下面的例子中，代码里的临时字段excessMinutesCharge，让人完全不清楚它的意图，而且没有必要。

![temporary-field]("assets/images/code-smell-temporary-field.png" )

### Refused Bequest(被拒绝的遗赠)
被拒绝的遗赠，当基类的特征是一些子类需求，而另一些子类是不需要的，那么需要重新抽象基类；如下面的例子，当子类不需要父类的属性时，那么该属性就不应该存在父类中。

![refused-bequest]("assets/images/code-smell-refused-bequest.png" )

### Alternative Classes with Different Interfaces(异曲同工的类)

A类的接口A，和B类的接口B，实现的是相似的功能时，这是重复代码的表现，需要消除重复的代码

![alternative-classes]("assets/images/code-smell-alternative-classes.png" )




