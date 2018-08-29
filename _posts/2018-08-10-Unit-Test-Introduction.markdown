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

Generally , we will have a singular point changing the advantage between auto-testing and manual testing. 

If the project is small and the maintenance lifetime is short , there is no need to use auto test.

Just as the volume ,complexity ,number of iteration of the project becomes big,the benefit of auto testing will finally overtake.

## Goals of tests


### improve quality

It is an executable specification to help us improve external quality.

If we use UT code as an executable specification, it can help us to understand the requirement better, because we need to provide an example for each scenario, and if we change our product code, the test will break, which help us to synchronize the test as the project proceeds.  

AS the below example, we can see the test describes a test senario clearly. Given there are 10 black jacket and 5 shirt in the stock, when there is a customer comes and ask to change his black jacket for a shirt, then there should be 11 black jacket and 4 shirt left in the stock. 

```java

public class StockTest{

  @Test

  public void should_support_the_customer_return_for_exchange(){

    Stock stock = new Stock();

    stock.put(ClothFactory.create(BLACKJACKET,10));

    stock.put(ClothFactory.create(SHIRT,5));

     

    stock.exchangeCloth(BLACKJACKET,SHIRT);

     

    assertEquals(11, stock.getClothCount(BLACKJACKET));

    assertEquals(4, stock.getClothCount(SHIRT));

  }

}

```

UT do not find bug,but repel bug

![ut]({{ "assets/images/image-ut.png" | absolute_url }})

UT written with TDD can not help us to find bug(it doesn't existed at the first place), but can help keep the bug from crawling in.

Defact localization

![defect-localization]({{ "assets/images/image-defect-localization.png" | absolute_url }})

As code base grows, to find out if there is any problem in the system is getting harder and harder.


Auto UT can help to locate the problem in seconds. 

And the above two points need our UT to have high coverage rate, run fast and be run frequently. 


### understand the SUT(system under test)

We usually will find ourselves forget about the behavior of the class we designed months ago. And sometime, our team mate would come and ask how to use the API we designed. 

At this moment document can help, but document can be out of synchronization, and it is not as useful as automation test case which can help to do auto regression test. 

Below is an example of using test as document, we can see that, in the given part, it illustrate what is the precondition needed to call the API. 

The when part indicates how to call the API, what kinds of parameters it needs. 

The then part tells what has happened after the tested method is called.


```java

@Test

public void should_be_able_to_search_by_tags_when_it_is_nested(){

    //We can see how to use the API

    Dom dom = buildDom("<root><tag1><tag2 id=\"i1\"></tag2></tag1><tag2 id=\"i2\"></tag2></root>");

 

    SearchResult result = DomSearcher.search(dom, "tag1 tag2");

 

    //The result should look like

    assertTrue(result.isNotEmpty());

    assertEquals("i1", result.get(1).getAttr("id"));

}

```

### reduce risk

UT is a safenet

![]({{ "assets/images/image-safenet.png" | absolute_url }})

As we know, to rely on the test team to do all regression tests and make sure the system has no problem is not feasible. 

But we can rely on the UT that have coverage high enough to be our safenet. 

With UT as the safenet and version control tool, we feel safe to do any refactoring to improve our system.

UT should not do harm

![harm]({{ "assets/images/image-harm.png" | absolute_url }})

As the picture shows, UT should be just used as scaffold, which helps us to build the house better and easier but should not jeopardize the house itself. 

Below is a bad example, it is a piece of production code, in which there is a logic decides if it is the test environment. 

This logic is added just for testing and impact the readability of the code seriously. 

```java

//This is a bad example.

public static Price calculatePrice(Purchase purchase) {

  BaseProductInfoSrc infoSrc;

  if (Environment.isTest()) {

    infoSrc = new StubBaseProductInfoSrc();

  }else {

    infoSrc = new BaseProductInfoSrc();

  }

  return calculate(purchase, infoSrc);

}

```

## Be easy to write , maintain and run
Test should be easy to write, maintain and run, or we will just think it as a big burdon. 

### expressive 

A good test should be easy to understand, anytime the system needs to change for requirement changed, the test and production code need to be changed at the same time.  If the test code is hard to read and maintain, it would let us be harder to make the change.

Please just compare the below two example, they are doing the same thing, but one is very easy to understand and the other is not. Clean code principle should be applied to test code as well.

```java

@Test

//This is a good example

public void should_restock_when_customer_return_for_refund_given_stock_available(){

  Stock stock = new Stock(15);

  stock.put(ClothFactory.create(BLACKJACKET,10));

 

  stock.returnCloth(ClothFactory.create(BLACKJACKET,1));

 

  assertEquals(11, stock.getClothCount());

}

 

@Test

//This is a bad example

public void testreturn(){

  Stock s = new Stock(15);

  Cloth[] c = new Cloth[10];

  for(int i=0;i<10;i++){

    Cloth ci = new Cloth("BJCode");

    c[i]=ci;

  }

  s.put(c);

  s.returnCloth(new Cloth("BJCode"));

  assertEquals(11, s.getClothCount());

}

```

### fast

Test needs to be run as frequently as possible, or it can not help us to make sure at any time, our system is in good shape.

The amount of unit test is big, for instance, if we have 3000 unit tests in one module, and it needs 200 micro second to run one on average, how long do we need to run all of them? 

200 ms * 3000 = 600 sec = 10 mins

It is too long that no programmer will like to run it. It is preferable that one test just takes several micro seconds.


### isolated

We can imaging, if we need to run a script before running a unit test, it is hard to use. 

If we need to start a external server to run a unit test, it is hard to use.

If we need to make sure another test is run before this test, it is hard to use. 

So a good test should be able to run without any dependency.

The below example is another bad practise for it verifies two conditions in one test. In this case, if the test fails, it is not able to get the reason at one glance. 

```java

@Test

public void matches() {

  Profile profile = new Profile("Bull Hockey, Inc.");

  Question question = new BooleanQuestion(1, "Got milk?");

 

  //answer false when must-match criteria not met

  profile.add(new Answer(question, Bool.FALSE));

   

  Criteria criteria = new Criteria();

  criteria.add(new Criterion(new Answer(question, Bool.TRUE), Weight.MustMatch));

  assertFalse(profle.matches(criteria));

   

  //answers true for any don't care criteria

  profile.add(new Answer(question, Bool.FALSE));

 

  criteria = new Criteria();

  criteria.add(new Criterion(new Answer(question, Bool.TRUE), Weight.DontCare));

  assertTrue(profile.matches(criteria));

}

```

### repeatable 

We can image, if one test can only be run on the author's PC, is it a good test? No.

If the test can not be run twice without any data patch, is it a good test? No.

If the test can only be run on Windows, is it a good test? No.

If the test can only be run on working day, but fails on Sunday, is it a good test? No.

So good tests should be able to run on any machine, any environment, any time and have the same result. 


### self-validating

In the below example, the first test method needs to be verified manually. 

It would make it impossible to run and do the verification manually if we have more than tens of them, no mention if we have thousands of them.

```java

@Test

//Bad Example

public void should_return_6_results_when_given_3_charactors() {

    Anagrams anagrams = new Anagrams();

 

    List<String> result = anagrams.from("ABC");

 

    //The test needs manual verification

    System.out.println(result);

}

 

@Test

//Good Example

public void should_return_6_results_when_given_3_charactors() {

    Anagrams anagrams = new Anagrams();

 

    List<String> result = anagrams.from("ABC");

 

    //The test can verify its expected result

    assertThat(result).containsExactlyInAnyOrder("","ABC","ACB","BAC","BCA","CAB","CBA");

}

```

### timely


It would be easier for us to write test at the time the production code is written, because at that time, we know about the purpose of the production code clearly, and we may forget many details as we start another function.

The earlier it is written, the earlier it can help us to keep our system in good state.

It would be best to write it before the production code, because it can help the programmer to clear the mind, what should be done at the next step, and get focused.


## Be easy to evolves 

Bad tests are just a burdon preventing our system to evolve to be better. So we should make sure to build our test in a good way to make them to be a catalyst to change.

### focus on value

Test should be built from outer to inner. We should always care about a module's behavior, its value, not its inner structure, its implementation. Each test should have a link to the requirement, to the value the tested module can provide, which proves its usefulness. Useless code or test should not exist in the system, for they are a burdon to maintain and impede us to change. 

### tests should have low overlap

The verification areas between different tests should be minimal. Below is a bad example. We can see both tests verify the result type. 

For any requirement to change the result type, we need to change multiple tests, which make us harder to change.

```java

@Test

public void should_be_able_to_search_by_tags_when_it_is_nested(){

    Dom dom = buildDom("<root><tag1><tag2 id=\"i1\"></tag2></tag1><tag2 id=\"i2\"></tag2></root>");

 

    SearchResult result = DomSearcher.search(dom, "tag1 tag2");

 

    assertTrue(result.isNotEmpty());

    //This is checked

    assertEquals("text/xml", result.getResultType());

    assertEquals("i1",result.get(1).getAttr("id"));

}

 

@Test

public void should_be_able_to_search_by_class (){

    Dom dom = buildDom("<root><tag1><tag2 id=\"i1\" class=\"c1\"></tag2></tag1></root>");

 

    SearchResult result = DomSearcher.search(dom, ".c1");

 

    assertTrue(result.isNotEmpty());

    //This is checked again

    assertEquals("text/xml", result.getResultType());

    assertEquals("i1",result.get(1).getAttr("id"));

}

```

### should not depend on environment 

Below is a bad example, we can see in this test, it depends on the sql server connection and threadload. 

If one day, the team wants change Sql Server to DB2, we will find that it is hard, for we need to change all tests depend on the sql server connection. 

The same problem happens when we want to use property file to replace threadload to store the connection string. 

So the test should not depend on the environment, or it would prevent us from making any change to the environment. 


```java

@Test

public void should_return_customers(){

  //Depend on the DB

  SqlServerConnection conn = getSqlServerConnection();

  //Depend on the threadlocal

  String srcString = ThreadLocal.get("srcString");

 

  CustomerManager manager = new CustomerManager();

  manager.setConn(conn);

  manager.setCommand(srcString);

  List<Customer> customers = manager.queryAllCustomer();

 

  assertNotNull(customers);

  assertEquals(10, customers.size());

}

```

### Only apply DRY(Don't Repeat Yourself)to the supportive part

DRY can be apply to our test code, but not each part of them. 

We can apply DRY to the given and then part of the test by extracting the duplicate logic into a test util, which can help to make our test more readable.

But we must keep the three parts, given, when, then, can be clearly expressed in our test, and the when part should not be extracted and proxied, or it is hard to know what the test is testing. 

