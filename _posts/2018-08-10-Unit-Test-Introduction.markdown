---
layout: post
title: unit test introduction
date: 2018-08-10
categories: unit test
---


## what is UT actually looked like

```java
@Test
public void should_pring_success_given_list(){
	//given
	Purchase purchase = new Puchase(Arrays.asList(new Milk(),new Book()));
    Print print = new Print(puchase);
    //when
    String result = print.printPurchase();
   //then
   assertThat(result).contains("milk","book");
} 
```
It is just plain java method with a test annotation on it.

It should have three standard parts, given, when, then, the given part prepares the precondition of the test, the when part call the method to be tested, and the then part just verifies if the result is as expected. 

When used in project, the comments of given, when, then in the above example should not be added, for it is just for educating purpose.


## compare UT with other tests

### what does it mean by "unit" of a unit test

1.  unit tests are low-level , focusing on a small functional point
2.  unit tests should completed by developer who are responsible for implement functionality
3.  unit testes should run faster more than other kinds of tests , like : Service Tests , UI tests


### compare unit test with service test and UI test


![unit-test-compare-with-other-tests]({{ site.url }}/assets/images/image-unit-test.png "unit test compare with other tests")

Unit tests usually tests the functionality olny includes POJO and is the faster , A unit test's cover range is small . The numbers of unit tests is the bigger . the coverage can give is high
The other types of test  , including service test and UI test , can only provide  limited coverage rate , and their coast are higher , so their amount is relatively smaller , and they are run slowly , so they can not give a feedback quickly to the developer.

![business/technology facing]({{ site.url }}/assets/images/unit-test-business-facing.jpg "business technology facing")

There are lots of tests in this industry, in the above picture, we can see them classified in four quadrants. 

1. The upper quadrants(Q2,Q3) are mainly to verify the function is built according to the requirement, so it is called "business-facing". 

2. The lower quadrants(Q1, Q4) are for the internal quality of our delivery, which means if the code is built in a expressive, low coupling, high cohesive way.

3. The left quadrants(Q1, Q2)  are for supporting the development team to deliver better and easier.

4. The right quadrants(Q3, Q4) are for verifying the quality of the system as whole by involing test team or user to do the test


## why bother to write UT


### brain overload

Many developer may feel that programming is wearing .One of the main reason is , when programming ,we need to maintain the logic in a consistent state . If the functionality is big . our brain needs to scan again and again if logic are consistent , which drains our mind energy dramatically. 

In a modern psychology research, normal human beings can hold only 7 logic aspects in consistency, but 7 is small comparing to a programmer's daily work.

And unit test expecially TDD can effectively split the problem area into small aspect, and the programmer just needs to focus on the current small step's problem, which obviously relieve the spirit burdon of the programmer.


### shotgun change

![shotgun-change]({{ site.url }}/assets/images/image-shotgun-chnage.png "shotgun change")

We usually find ourself in a situation that some requirements changing  will impact many points of different method coding  . and the worse case is that , the modifications of these coding are not the same , No one can be sure of finally changing these code will not incur any defect , Though system covered by automatic tests do not suffered to this problem .  

### broken safenet

![broken safenet]({{ site.url }}/assets/images/image-broken-safenet.png "broken safenet")

In many teams , expecially those use waterfall development model , the test team is treated as a safenet , Since having safenet , developer ,even everyone thought of quality should be took over by test team , and thought of test team guarantees the delivery quality , and thought of no one in the group but the test team needed to take responsibility to the final quality of the delivery 

But the real is, the safenet seems has broken holes , and defects leak . Because of as the system evolved , the code base was growing up , more and more supplemental functionality , many requirement has been changed . The coast of fully regression test for each release is becoming more and more expensive and rarer , so it is difficult to avoid defects , Many projects failed or become more and more difficult cause by this delusion of having perfect safenet.   

### hard to hit the root cause of the problem

![hard to hit root cause]({{ site.url }}/assets/images/image-hit-cause.png "broken safenet")

In a high coupling system , most time , root cause of defects are hard to find out . Sometime , to setting up the environment to duplicate a error is time consuming . Using TDD, we will be more apt to write system with low coupling(because high coupling make it hard to be tested.), which will be more easy to isolate out the sub-module having defect, and setup scaffold to build the test to reproduce the error. 

### high risk to make change

![risky]({{ site.url }}/assets/images/image-risky.png "high risk")

Even though we were  aware that our system have many kinds of problem , and we know we should deal with these problem in one way or another ,for example refactoring that bad code . but no one dare . because of so high coast to do regression test for changing .
So it is time for us to do something to help ourselves to say good bye to the problem above and have a better career experience 

## the ROI of automation change

![roi]({{ site.url }}/assets/images/image-roi.png "roi")


## Goals of tests

### improve quality

### understand the SUT(system under test)

### reduce risk


## Be easy to write , maintain and run


### expressive 


### fast


### isolated


### repeatable 


### self-validating

### timely

## Be easy to evolves 


### focus on value

### tests should have low overlap

### should not depend on environment 


