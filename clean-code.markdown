---

layout: post
title: Clean Code
date: 2018-09-18
categories: cleanCode
description: 关于我们晶常开发过程中，怎么写出干净可用的代码：clean code

--- 

## What is clean code?

很容易即可以写出机器可以理解的代码，但是在软件工程范围内写出易于人理解的代码才是一个程序员应该做的事。 clean code即整洁的代码，  代码意图明确，易于理解的代码。
* 代码逻辑应当直接了当
* 尽量减少少依赖关系
* 只做好一件事
* 简单直接
* 不隐藏设计者意图
* 没有冗余，没有重复

## Why we need clean code

我们先看下面一组市场调查数据:

![clean-market-data]("assets/images/clean-code-market-data.png" )

* 上面的图表显示，生产的代码行数随着软件生命的延长，生产的代码行数不断增长，但是增长率会变小。
* 软件生命周期越长，需要的开发人员越来越多。
* 随着开发人员增长，但是新代码的增加率却变慢了，生产每行代码的费用变越来越大了。
* 生产效率越来越低了。

如下代码示例，随着时间的推移，即使作者也会搞不清楚代码需要表达的业务意义。

![clean-market-data]("assets/images/clean-cood-code-demo.png" )

当你在写这代码时，只有老天和你知道这段代码需要表达什么业务含意，当过了相当长一段时间后，也许是半年，或者一年多以后，有可能更长的时间后，只有老天知道这段代码是要完成什么业务功能。

![god-knows]("assets/images/clean-code-knows.png" )

### We are the authors

* 研究表示，开发人员日常工作中阅读代码和写编写代码的时间是10:1
* 我们是作者，我们在创作，作者有读者。作者的一个重要任务和职责就是如何清楚地表达自己的思想，内容，把自已的意图清淅地传达给读者。通常情况下，我们会是我们写的代码的第一个读者。
* 很容易写出机器可以阅读的代码，但是写出人可以很容易阅读的代码者是一个合格的开发人员。

### clean code 的好处

* 花时间保持你的代码整洁，不仅是节约成本，更是职业生存的关键；
* 让你在期限内交付，让你工作效率更高的唯一方法是保持你的代码尽可能地整洁；
* 保持代码整洁是让你工作效率更高的唯一方法；
* 坏代码会导致混乱增长；
* Clean code是为了让我们自己在写代码时更轻松点；

### how to clean code

The Philosophy of Clean Code

![how-to-clean-code]("assets/images/cleancode-how-to.png" )


* Don't Repeat Yourself ：不要重复你自己，不要有重复的代码；
* KISS(keep it simple stupid)：不要写复杂的代码，让你的代码在实现功能的同时越简单越好；
* SOLID：代码要尽量符合设计原则，SOLID内容有对应课程；
* You aren't gonna need it：不要过程设计，代码就是刚好满足需求就好。

#### simple design

THE 4 RULES OF SIMPLE DESIGN:

![simple-design]("assets/images/clean-code-simple-design.png" )

1. 编写测试会引导我们做出更好的设计，为了让我们的系统可测试会推动我们把类设计得小，并且只做一件事；当我们写的测试越多，会推动我们编写更容易测试的代码。一个有成熟可测试的系统才是一个稳定灵活的系统。测试是基础。因为有了测试，我们才不怕在clean code时破坏我们的系统。

2. 重复是一个好设计系统的主要敌人，重复代表着额外的工作量，额外的风险，额外的不必要的复杂度。重复代码有很多表现形式，可以表现有一样的代码行或者不同的代码但实现的功能是一样的。消除重复，不只是消除大的重复，即使是两三行重复的代码也应该要消除。我们在一个很细小的级别上抽取相同代码变成一个公共函数来消除重复，当这个级别很细时，我们可能会发现这些抽出来的一个个小小的函数，放在一个类里会违法SRP（单一职责原则），那我们会把这个函数抽离到对应的类中，那么这些类，就可能可以让其他开发人员在其他领域中重用。所以在这种“reuse in the small”的方式可以让我们的代码复杂度有显著的减少。

3. 在代码上清楚表达你的业务意图，为你的代码选择适合的名字，让它们易于阅读，没有歧义，直接明了。一个系统的大部成本都是维护上，而前面也说了，阅读代码和编写代码的时间花费是10：1，让代码易于阅读对开发效率是很好的提升。而代码写得短小，功能越单一，就越容易表达。所以关键还是要写短小的代码。

4. 前面的努力，可能会导致有很多小的方法和小的类，这时我们就应该尽量减少这些方法和类，我们的目的就是让我们的系统尽可能小的同时保持尽可能少的方法和类


#### Meaningful Names

有意义的命名，就是名副其实。如下图，The eater plant， 吃人的植物。那么这植物必须会吃人才是名副其实。


![meaningful-design]("assets/images/clean-code-maeaningful.png" )

所以名称应该告诉你，它是什么、为什么会存在、它做了什么事、应该怎么用。如果需要注释来补充，那就不算名副其实了。

tips
* Classes and objects should have noun or noun phrase names.类名和对象名应该是名词或名词短语。
* Methods should have verb or verb phrase names.方法名应该是动词或动词短语。
* Use solution domain names. For example,  AccountFactory or JobQueue.使用解决方案领域名称。例如，访问者模式可以用AccountVisitor，也可以使用JobQueue等。
* Use problem domain names.使用源自所涉问题领域的名称。就像我们是保险行业的，那我们可以使用policy, product, benefit这些来命名我们的问题域。
* Use long names for long scopes.用长名字来描述长范围，在适当情况下可以使用长名字，只要可以清楚表达真实意图就可以。如果太长，那么需要考虑方法是否太大，要考虑拆分方法。
* Names should describe side-effects.虽然不建议方法会有副作用，但如果方法是有副作用的，那必须要在方法名里有说明。这里的副作用是指修改了转入的参数。
* Pick one word per concept.为一个概念选择一个词来表达，并且组织内，这个概念只用这个词来表达。

#### Comments

“Don’t comment bad code—rewrite it.”  —Brian W. Kernighan and P. J. Plaugher

![comments]("assets/images/clean-code-comments.png" )

别给糟糕的代码加注释——重新写吧。

注释总是一种失败。如果发现自己需要写注释，那就尽量先重构自己的代码。

##### GOOD COMMENTS

一般来说，需要注释的合理场景如下：
* Javadocs in Public APIs：公共的JAVADOC的API，用来说明该API的使用方式。
* Legal Comments：法律信息，如作者、公司、代码的版权信息等。
* TODO Comments：要做但还没有做的内容。
* Warning of Consequences：对代码运行结果需要进行警告的信息，特别是运行结果会有很大影响的情况下；如运行时长有可能很长，如果清除数据库内容等。

##### Formatting

代码格式就是代码缩进，换行、括号的位置等规范，一个团队只有一套规则，并且这套规则应该在开发工具上定义好模板，大家都导入该模板，并且设置保存代码时自动格式化代码就可以了。
One Team One Rule.

##### Functions 

* Small：函数的第一规则是要短小。
* Smaller Than That：第二条规则是还要更短小。20行应该封顶。能在几行内完成的方法就更好了。
* Do One Thing：函数应该做一件事，做好这件事，只做一件事；
* One Level of Abstraction per Function：每个函数一个抽象层级。函数中的语句都要在同一抽象层级上，避免让人迷惑。举个product  productAge prdoutPlan的例子；
* Top To Bottom：自顶向下读代码：向下规则。代码应该自上而下写，而且相同作用的代码要放在一起，不要定义了一个变量，然后隔了几行代码才使用。
* Have No Side Effects：方法不应该有副作用，也就是不要修改传入的参数对象，如果必须要有副作用，那么在方法名字上要暴露这点；
* Function Arguments as less as it can：方法的参数越少越好，如果超过3入参，那么就得考虑封装了。

##### Error Handing

代码中的错误处理应该参考以下几点：
* Use Exceptions Rather Than Return Codes：使用抛出异常，而不用返回码；如果我们每个方法都使用返回码，那么，我们在调用方法后，都需要对返回码进行处理，无论是正确还是错误，都得判断一下，如果使用异常，那么异常只需要在框架上统一处理就可以，我们的代码不需要关心异常的情况。
* Write Your  Try-Catch-Finally Statement First：如果你需要关心你写的代码运行的异常情况，那么你就先写你的异常处理代码。
* Use Unchecked Exceptions：使用Unchecked Exceptions，考虑一下，如果底层代码抛出了一个checked exception，而上层代码完全不想处理这个异常，但不得不catch这个异常，或者在方法上增加抛出声明。这个就破坏了开闭原则。
* Define Exception Classes in Terms of a Caller’s Needs：为调用方定义自己的模块的异常类，把调用方不关心的异常装成一个公共的异常。

### summary 

Clean Code是代码工匠的自我修养，工匠指在某一方面造诣高深的人，而我们不能只满足于完成功能开发，坚持clean code，可以让我们在写代码方面造诣高深。

![summary]("assets/images/clean-code-summary.png" )
