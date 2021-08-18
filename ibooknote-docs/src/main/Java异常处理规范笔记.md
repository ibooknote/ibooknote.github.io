# Java异常处理规范笔记

## 1. Java异常介绍

* 所有异常都是Throwable的子类
    * Error子类的异常是程序无法预知、无法处理的异常发生时产生，如内存耗尽
    * 程序员可报告的异常都是 Exception 的子类
        * 未检查（Unchecked）异常都是 RuntimeException 的子类
        * 所有其他异常都是 checked 异常

checked 异常必须在程序中捕获或者在方法头部声明，编译器在编译时会检查这些异常是否做了相应的处理。

unchecked 异常可以抛出到最适合处理它的地方来捕获、处理

异常的黄金法则：“早抛出，晚捕获”

如果捕获到异常又不知道如何处理，只是想记录日志，可以捕捉之后再重新抛出。

## 2. 《Effective Java 3rd Edition》中提到的关于异常的最佳实践

Effective Java中文版（原书第3版）（10. Exceptions）

这些实践经验可以作为项目的异常处理规范来使用。

### item 69: 只在异常条件下使用异常

不要使用异常代替边界判断，去控制数组的遍历。正常的循环终止测试,性能要比异常的创建、捕捉好很多，并且很多JVM会对其进行优化。

__异常正如他们的名字所描述的，仅仅用于异常条件下；永远不要用于正常的流程控制。__

这条原则也适用于API设计：一个设计良好的API一定不能强制它的客户端使用异常来处理正常流程控制（例如Iterator必须提供hasNext()用于终止迭代判断）

除了hasNext()这种单独的状态测试方法以外，另一种做法是：状态相关的方法被调用时，如果该对象处于不适当的状态中，则返回一个可被识别的值，比如null。但这种方式对于Iterator的应用场景并不合适，因为null是next方法的合法返回值。

“状态测试方法”和“可被识别的返回值”两种方式在使用上的选择原则：

* 如果一个对象在缺少外部同步控制的情况下被并发访问，或者可被外部改变状态，那么使用一个可被识别的返回值可能更有必要，因为在调用“状态测试”方法和调用对应的状态相关方法的时间间隔之间，对象状态可能会发生变化。
* 如果一个单独的“状态测试”方法必须要重复“状态相关”方法的工作，那么从性能角度考虑，应该使用可被识别的返回值。
* 如果不存在上述两点问题，那么“状态测试”方法略优于可被识别的返回值，因为它提供了更好的可读性。

### item 70: checked 异常用于可恢复条件，runtime 异常用于程序错误

Java语言提供的三类异常：

* checked exceptions
* runtime exceptions
* errors

决定是使用已检查异常还是未检查异常的一条重要原则：

* 当调用方能够合理的恢复时，使用 checked 异常 -- 通过抛出 checked 异常，可以强迫调用方捕获并处理异常，或者将其传递给上层调用方。方法声明抛出的每一个 checked 异常可以有效的指示 API 使用者，相关状况是执行方法的一种可能输出。

举例：

* HttpTimeoutException，Http超时的异常，可以让调用方自己来决定是否需要重试
* 当处理过程中如果抛出异常，需要保存中间状态用于后续重试，那么就要捕捉异常并处理

有两种未检查的throwables：runtime 异常和错误。它们在行为上是一致的：都是可抛出的，不需要也不应该被捕获。如果一个程序抛出一个未检查异常或者错误，通常表明这种状况是不可恢复的，并且继续执行下去有害无益。如果一个程序不捕获这种异常，它将导致当前线程挂起，并带有合适的错误信息。

使用 runtime 异常表示程序错误 -- _precondition violations_。

precondition violations -- 是指API客户端没有遵守API规范发布的约定。例如数组越界异常、参数无效等。

如果相信一个条件下允许恢复，使用已检查异常；否则，使用运行时异常。如果不太明确使用哪种，那就尽量选择未检查异常。（原因参见 item 71）

普遍接受的约定： 最好不要实现 Error 的子类；所有未检查 throwables 都应该直接或间接的实现为 RuntimeException 的子类。不但不应该定义 Error的子类,AssertionError 的异常也不应该被抛出。

永远不要自己定义Exception、RuntimeException 或者 Error 以外的其他异常。Java语言规范会把它们当作普通的 checked 异常。

因为 checked 异常通常指示了可恢复条件，对于这样的异常，提供一些辅助方法非常重要，可以帮助调用者在异常条件下获取用于恢复的额外信息。例如，当购买礼品卡时，因余额不足而抛出的 checked 异常。这种异常应该提供一个方法可以查询缺少的金额。这样，调用方可以将金额信息返回给购买者。

总结：

* 可恢复条件下，抛出 checked 异常；
* 程序错误抛出 unchecked 异常；
* 当有所怀疑时抛出 unchecked 异常；
* 不要定义除了 checked 异常和 runtime 异常以外的其他异常；
* 为 checked 异常提供 recovery 时用于获取信息的方法。

### Item 71: 避免不必要的情况下使用已检查异常

如果一个方法抛出一个或多个已检查异常，那么调用方必须在一个或多个catch中处理这些异常，或者声明抛出这些异常，向上传递，这会加重程序员的负担。

在Java 8中，当方法抛出 checked 异常时，它不能直接用于 streams 中。

_如果正确使用API无法阻止出现异常情况，并且程序员使用API时如果遭遇异常可以采取一些有用的措施，那么这种负担是合理的。除非这两个条件都成立，否则，未检查异常更合适。_

如果使用API的程序员不能更好的处理，那么未检查异常更合适。

总结：

* 谨慎的使用 checked 异常，可以增加程序的可靠性；
* 当过度使用 checked 异常时，会让API的使用变的很痛苦；
* 如果调用方不能从失败中恢复，那么抛出 unchecked 异常；
* 如果恢复可能可行，并且希望强制调用方处理异常情况，可以优先考虑返回一个 optional
* 仅当在失败情况下提供的信息不足时，才应该抛出已检查异常（？？？）

### Item 72: 尽量使用标准异常

重用标准异常的几点好处：

* API 更容易学习和使用（标准异常大家都熟悉）
* 使用这种API的程序可读性更好
* 更少的异常类，意味着更小的内存空间需求和类加载时间

常用的异常：

* IllegalArgumentException 传入参数不合法
* IllegalStateException 要调用的对象状态不合法，如未初始化
* NullPointerException
* IndexOutOfBoundsException
* ConcurrentModificationException
* UnsupportedOperationException

不要直接使用 Exception, RuntimeException, Throwable, Error。把这些类当作抽象类对待。因为它们是其他异常类的超类，所以当方法抛出时，不能更可靠的测试这些异常（可能不是你预期的地方抛出的，而是其他位置抛出的异常子类，而被你给当成超类对象捕捉到了）

### Item 73: 抛出适合于抽象的异常

exception translation -- _上层应该捕获底层的异常，在它们的位置再抛出可以用上层抽象解释的异常。_

exception chaining -- 当底层异常有助于调试问题时，底层异常当作 cause 传递给上层异常，上层异常提供getCause方法可以重新获取底层异常。

尽管异常转换比无意识的从底层传递异常更好，但也不要过度使用。


## 3. 《Core Java Volume I -- Fundamentals》 关于异常的描述

Chapter 7 Exception

7.3 异常使用提示

1. 不能使用异常处理来代替正常的单个测试 -- if (!s.empty()) s.pop();
2. 不要将每条语句可能产生的异常单独管理（每条语句用一个try 、catch）
3. 合理利用合适的异常级别 -- 不要直接抛出 RuntimeException，而是选择一个合适的子类或创建自己的
4. 不要压制（隐藏）异常
5. 不要可以用无意义值代替异常（当一个栈是空的时候，pop是返回null还是抛出异常？）应该立即抛出异常，这符合今早抛出原则。
6. 不要强迫自己捕获所有异常，有些异常在更高层处理可能更合适

## 4. Spring Web框架异常处理规范

### 4.3. Spring官方文档中对异常的描述


https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers

```
1.1.7 Exceptions
If an exception occurs during request mapping or is thrown from a request handler (such as a @Controller), the DispatcherServlet delegates to a chain of HandlerExceptionResolver beans to resolve the exception and provide alternative handling, which is typically an error response.

The following table lists the available HandlerExceptionResolver implementations:

HandlerExceptionResolver	Description
SimpleMappingExceptionResolver
A mapping between exception class names and error view names. Useful for rendering error pages in a browser application.
DefaultHandlerExceptionResolver
Resolves exceptions raised by Spring MVC and maps them to HTTP status codes. See also alternative ResponseEntityExceptionHandler and REST API exceptions.
ResponseStatusExceptionResolver
Resolves exceptions with the @ResponseStatus annotation and maps them to HTTP status codes based on the value in the annotation.
ExceptionHandlerExceptionResolver
Resolves exceptions by invoking an @ExceptionHandler method in a @Controller or a @ControllerAdvice class. See @ExceptionHandler methods.
```

1.3.6. Exceptions

https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler

带有 @Controller 注解的控制器和 带有 @ControllerAdvice 注解的类可以使用 @ExceptionHandler 注解方法来处理控制器方法中抛出的异常。

https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-rest-exceptions

REST API exceptions

1.3.7. Controller Advice


### 4.2. Spring官方博客中对异常处理的描述

https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc

What to Use When? Spring关于异常处理的建议：

* 自定义异常，考虑添加 @ResponseStatus，这样在控制器返回给客户端时，直接返回相应的HttpStatus
* 对于所有其他已定义的异常，在 @ControllerAdvice 类上实现一个 @ExceptionHandler 方法，或者使用 SimpleMappingExceptionResolver 的一个实例。
* 对于某个控制器中如果有特殊的异常需要处理，则在控制器中的方法上添加 @ExceptionHandler 
* 警告： 在同一个应用中混用上述多种方式要小心。如果同一个异常能通过多种方式处理，可能不会达到预期的效果。控制器中方法的注解 @ExceptionHandler 通常先于 @ControllerAdvice 实例。controller-advices 二者之间无先后顺序定义。

## 5. 其他参考资料

### 5.1. 《Spring 5 design patterns》

```
RuntimeException, that is, it is an unchecked exception. In an enterprise application, unchecked exceptions can be thrown up the call hierarchy to the best place to handle it. The good thing is that the methods in between don't know about it in the application.
```

```
Roll back if the business method throws a RuntimeException--it is the default behavior of a Spring transaction, but you can override it for checked and custom exceptions also
```

### 5.2. 《Thinking in Java Fourth Edition》

Standard Java exceptions

```
There’s a whole group of exception types that are in this category. They’re always thrown automatically by Java and you don’t need to include them in your exception specifications. Conveniently enough, they’re all grouped together by putting them under a single base class called RuntimeException, which is a perfect example of inheritance: It establishes a family of types that have some characteristics and behaviors in common. Also, you never need to write an exception specification saying that a method might throw a RuntimeException (or any type inherited from RuntimeException), because they are unchecked exceptions. Because they indicate bugs, you don’t usually catch a RuntimeException—it’s dealt with automatically. If you were forced to check for RuntimeExceptions, your code could get too messy. Even though you don’t typically catch RuntimeExceptions, in your own packages you might choose to throw some of the RuntimeExceptions.
```

```
Keep in mind that only exceptions of type RuntimeException (and subclasses) can be ignored in your coding, since the compiler carefully enforces the handling of all checked exceptions. The reasoning is that a RuntimeException represents a programming error, which is:

1. An error you cannot anticipate. For example,a null reference that is out side of your control.
2. An error that you, as a programmer, should have checked for in your code (such as ArraylndexOutOfBoundsException where you should have paid attention to the size of the array). An exception that happens from point #1 often becomes an issue for point #2.
```

### 5.3. 《Core Java SE 9》

5 Exceptions, Assertions and Logging

5.1.3


*Throw early, catch late*
----
TIP: Some programmers think it is shameful to admit that a method might throw an exception. Wouldn’t it be better to handle it instead? Actually, the opposite is true. You should allow each exception to find its way to a competent handler. The golden rule of exceptions is, “Throw early, catch late.”
----

参考资料：

* 《Effective Java 3rd Edition》（+++）
* https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc （+++）
* 16-RESTful Applications with Spring Boot.pdf
* 《Core Java Volume I -- Fundamentals》（++）
* 《Spring 5 design patterns》（+）
* 《Thinking in Java Fourth Edition》（+）
* 《Core Java SE 9》（+）
