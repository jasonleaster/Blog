---
layout: post
title: "The brief introduction to the architecture of JUnit4"
date: 2016-09-11 15:44:46 +0800
comments: true
categories: Java
---

JUnit is a good framework for developer to finish testing tasks. I also use JUnit to help me with testing. From the perspective of a good programmer, the source code of JUnit is also a wonderful example to learnhow to write more good style code in Java.

In this article, I would like to try my best to share my view of JUnit.The version of JUnit what I talked is JUnit4 in this article.

<!-- more -->

### How JUnit start to run?

Most Java developers work under the IDE like eclipse, Intellij IDEA and so on. It's no doubt that the success of Java can't without the development of IDE in this years.
But there also will be a satire that some Java programmer just use this framework and have no idea about how it works.

Here is a simple demo for how to use JUnit to test your program.

``` Java

// MyTest.java
package org.junit;

public class MyTest {
    @Before
    public void before(){
        System.out.println("before");
    }
    
    @Test
    public void testFunc(){
        System.out.println("testFunc");
    }

    @After
    public void after(){
        System.out.println("after");
    }
}

```

I referenced the point of someone's view about Java programmer.

> "They press down the green buttion in their IDE, and the testing program just run. There even does't show a main() function in their testing program. They also don't think why it works and how it can run successfully? Why java program can run without a main() function? It's a magic? No, please think more deeply and don't make your self look like an innocent beginner."

With the help of IDE and framework, it's more and more easy for the developer to finish their project. 
It **DO** help the developer to accelerate the process to finish a project. It also hide some basical information about how program start to run. 

With the IDE, you still a programmer but not a magician :)


* Why `MyTest.java` could run without main() function? 
* Why the program will run with just pressing the button in your IDE? 


<img src="/images/img_for_2016_09/IntellijJUnitCallFrame.jpg" align="middle">

The reason is that IDE will integrated with some plugin. Some GUI button are corelated with that plugin.(Eg. Intellij IDEA have a junit-plugin for JUnit Framework). After you press the button, the plugin start to run and it will call the entrance of the framework for you. You can see that in the image, the first function called by java is `JUnitStarter` which is on package `com.intellij`. Finally, it will enter in `org.junit` which is the JUnit framework package.

You can also see the output of the console in your IDE. You can find that there is commands like this:

```

    D:\JAVA_ENV\jdk1.8\bin\java -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:58641,suspend=y,server=n -ea -Dfile.encoding=GBK -classpath "D:\IntelliJ IDEA\lib\idea_rt.jar;    ... A LOT OF JAR ...     E:\Java Framework Source\junit4-master\out\production\main" com.intellij.rt.execution.junit.JUnitStarter -ideVersion5 org.junit.MyTest

```

For simplicity, the commands over there can be simplied into the below procesure under the CMD or Linux-Shell.

```
Compile:
    javac -cp path/to/testclasses:path/to/junit-4.8.2.jar MyTest.java

Run:
    java  -cp path/to/testclasses:path/to/junit-4.8.2.jar org.junit.JUnitCore org.junit.MyTest

```

Now, the answer is clear that IDE just use a plugin to call the entrance of the framework instead of calling it by yourself under the OS console.

If you use `JUnitCore` to run your test, `JUnitCore` then uses reflection to find an appropriate Runner for the passed test classes. One step here is to look for a @RunWith annotation on the test class. If no other Runner is found the default runner (BlockJUnit4ClassRunner) will be used. The Runner will be instantiated and the test class will be passed to the Runner. Now it is Job of the Runner to instantiate and run the passed test class.

Let's start to look the architecture of the JUnit Framework.

### The high level view of JUnit

There are two main abstract model for JUnit framework. The one of it is the abstraction of test unit.
Each test unit has its own scope, name, initial state and the way how to release the resouces which are allocated during the testing time.

For JUnit4, it use this inheritance model to represent that concept.

Review the skeleton of `MyTest.java`, the test routine is around the test class. Here is the class `MyTest`.

``` Java

// MyTest.java
package org.junit;
...
    @Test
    public void testFunc(){
        System.out.println("testFunc");
    }
...

```

![images](/images/img_for_2016_09/JUnitInheritance2.jpg)

With the annotation, the framework use the reflection to get the meta-information of the test unit. 

A class has it's own member so JUnit use abstract class `FrameworkMember` to represent it. 
Class member can be classify into `field` and `method`.
So JUnit use class `FrameworkFiled` and `FrameworkMethod` which inherite from `FrameworkMember`.

``` java

class FrameworkField extends FrameworkMember<FrameworkField> {
        private final Field field;

        ... ...
}

/**
 * Represents a method on a test class to be invoked at the appropriate point in
 * test execution. These methods are usually marked with an annotation (such as
 * {@code @Test}, {@code @Before}, {@code @After}, {@code @BeforeClass},
 * {@code @AfterClass}, etc.)
 *
 * @since 4.5
 */
public class FrameworkMethod extends FrameworkMember<FrameworkMethod> {
        private final Method method;

        ... ...
}

```

`FrameworkField` maintain a private object of class `Field` which is used in **reflection**. So do `FrameworkMethod`.

JUnit tests are started using the `JUnitCore` class. It will can runners to finish the task.
The other important heritance show below there.

![images](/images/img_for_2016_09/JUnitInheritance1.jpg)

The `public class AllDefaultPossibilitiesBuilder extends RunnerBuilder `will be called and try to find a runner to run the test class which is written by user.

Finally, `BlockJUnit4ClassRunner` will be called.

``` java

/**
 * Implements the JUnit 4 standard test case class model, as defined by th
 * annotations in the org.junit package. 

 * It has a much simpler implementation based on {@link Statement}s,
 * allowing new operations to be inserted into the appropriate point in the
 * execution flow.
 */

public class BlockJUnit4ClassRunner extends ParentRunner<FrameworkMethod> {
     
    private final ConcurrentMap<FrameworkMethod, Description> methodDescriptions = new ConcurrentHashMap<FrameworkMethod, Description>();

    ... ...

    @Override
    protected List<FrameworkMethod> getChildren() {
        // scan test class for methonds annotated with @Test
    }
       
    @Override
    protected Description describeChild(FrameworkMethod method) {
        // create Description based on method name
    }
         
    @Override
    protected void runChild(final FrameworkMethod method, RunNotifier notifier) {
        if (/* method not annotated with @Ignore */) {                  
            // run methods annotated with @Before
            // run test method
            // run methods annotated with @After
        }
    }

    ... ...
}

```




### Summary


Reference: 
1. http://www.mscharhag.com/java/understanding-junits-runner-architecture
