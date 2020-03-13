---
layout:	post
title:	Spring Batch——Retry《官网翻译》
subtitle:   
date:	2020-01-16
author:	BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Spring
    - Spring Batch
---

**郑重声明：**

- 此文档翻译自[Spring官网](https://docs.spring.io/spring-batch/docs/4.1.x/reference/html/retry.html#retry )。

- 此文档用途只以学习为目的，无任何商业用途。

- 翻译文档中如有疑问或理解不正确之处，欢迎指正。

  

# 1. Retry（重试）
为了使处理更加健壮并且不容易失败，有时会尝试自动重试失败的操作，以备在随后的尝试中成功。易受间歇性故障影响的错误通常是暂时性的。示例包括对web服务的远程调用，该调用由于网络故障或数据库更新中的deadlockleoserdataaccessexception而失败。
## 1.1. RetryTemplate（重试模板）
> 从2.2.0开始，重试功能已从Spring Batch中取出。它现在是新库Spring Retry的一部分, [Spring Retry](https://github.com/spring-projects/spring-retry)。

为了自动化重试操作，Spring Batch采用了RetryOperations策略。RetryOperations定义了以下接口：

```
public interface RetryOperations {

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback) throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback)
        throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RetryState retryState)
        throws E, ExhaustedRetryException;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback,
        RetryState retryState) throws E;

}
```
基本回调是一个简单的接口，允许插入一些要重试的业务逻辑，如以下接口定义所示：

```
public interface RetryCallback<T, E extends Throwable> {

    T doWithRetry(RetryContext context) throws E;

}
```
回调运行，如果失败（通过引发异常），则会重试，直到成功或实现中止。RetryOperations接口中有许多重载的execute方法。这些方法处理在所有重试都结束时恢复的各种用例，并处理重试状态，这允许客户端和实现在调用之间存储信息（我们将在本章后面更详细地介绍这一点）。

RetryOperations最简单的通用实现是RetryTemplate。它可以如下使用：

```
RetryTemplate template = new RetryTemplate();

TimeoutRetryPolicy policy = new TimeoutRetryPolicy();
policy.setTimeout(30000L);

template.setRetryPolicy(policy);

Foo result = template.execute(new RetryCallback<Foo>() {

    public Foo doWithRetry(RetryContext context) {
        // Do stuff that might fail, e.g. webservice operation
        return result;
    }

});
```
在前面的示例中，我们进行web服务调用并将结果返回给用户。如果调用失败，则会重试，直到达到超时。

### 1.1.1. RetryContext
RetryCallback的方法参数是RetryContext。许多回调忽略上下文，但是，如果需要，它可以用作属性包，在迭代期间存储数据。

如果同一线程中正在进行嵌套重试，则RetryContext具有父上下文。父上下文有时在存储数据上比较有用，需要在要执行的execute之间共享。

### 1.1.2. RecoveryCallback

当重试结束时，RetryOperations可以将控制权传递给另一个回调，称为RecoveryCallback。要使用此功能，客户端将回调一起传递给同一方法，如以下示例所示：

```
Foo foo = template.execute(new RetryCallback<Foo>() {
    public Foo doWithRetry(RetryContext context) {
        // business logic here
    },
  new RecoveryCallback<Foo>() {
    Foo recover(RetryContext context) throws Exception {
          // recover logic here
    }
});
```
如果在模板决定中止之前，业务逻辑没有成功，那么客户端就有机会通过恢复回调执行一些替代处理。

### 1.1.3. Stateless Retry
在最简单的情况下，重试只是一个while循环。RetryTemplate可以一直尝试，直到成功或失败。RetryContext包含一些状态来确定是重试还是中止，但是这个状态在堆栈上，不需要全局存储在任何地方，所以我们称之为无状态重试。无状态重试和有状态重试之间的区别包含在RetryPolicy的实现中（RetryTemplate可以处理这两者）。在无状态重试中，重试回调始终在其失败时所在的线程中执行。

### 1.1.4. Stateful Retry
当失败导致事务资源无效时，有一些特殊的注意事项。这不适用于简单的远程调用，因为没有事务性资源（通常），但有时确实适用于数据库更新，特别是在使用Hibernate时。在这种情况下，只有立即重新抛出调用失败的异常，这样事务才能回滚，我们才能启动新的有效事务。

在涉及事务的情况下，无状态重试不够好，因为重新抛出和回滚必然涉及离开RetryOperations.execute（）方法并可能丢失堆栈上的上下文。为了避免丢失它，我们必须引入一种存储策略，将它从堆栈中取出并（至少）放在堆存储中。为此，Spring Batch提供了一个名为RetryContextCache的存储策略，可以将其注入RetryTemplate。RetryContextCache的默认实现是在内存中，使用一个简单的Map。在集群环境中对多个进程的高级使用还可以考虑使用某种类型的集群缓存实现RetryContextCache（但是，即使在集群环境中，这也可能是过度的）。

RetryOperations的一部分职责是当失败的操作在新的执行中返回时（通常包装在新的事务中）识别它们。为了方便这一点，Spring Batch提供了RetryState抽象。这与RetryOperations接口中的特殊执行方法一起工作。

识别失败操作的方法是通过识别重试的多个调用的状态。要标识状态，用户可以提供RetryState对象，该对象负责返回标识该项的唯一密钥。标识符用作RetryContextCache接口中的key 。

> 在RetryState返回的key中实现Object.equals（）和Object.hashCode（）时要非常小心。最好的建议是使用业务密钥来标识项目。对于JMS消息，可以使用消息ID。

当重试结束时，还可以选择以不同的方式处理失败，而不是调用RetryCallback（现在假定可能会失败）。与无状态的情况一样，此选项由RecoveryCallback提供，可以通过将其传递给RetryOperations的execute方法来提供。重试或不重试的决定实际上被委托给一个常规的RetryPolicy，因此可以在那里注入有关限制和超时的通常关注点（在本章后面将进行描述）。

## 1.2. Retry Policies
在RetryTemplate中，在execute方法中重试或失败的决定由RetryPolicy决定，RetryPolicy也是RetryContext的工厂。RetryTemplate有责任使用当前策略创建RetryContext，并在每次尝试时将其传递给RetryCallback。回调失败后，RetryTemplate必须调用RetryPolicy，要求它更新其状态（存储在RetryContext中），然后询问策略是否可以再次尝试。如果无法进行另一次尝试（例如达到限制或检测到超时），则策略还负责处理耗尽状态。简单实现抛出RetryExhaustedException，这会导致回滚任何封闭事务。更复杂的实现可能会尝试采取一些恢复操作，在这种情况下，事务可以保持完整。

> 故障本质上是可重试的或不可重试的。如果总是从业务逻辑中抛出相同的异常，那么重试它是没有好处的。因此，不要对所有异常类型重试。相反，试着只关注那些你希望可以重新发布的异常。更积极地重试通常不会对业务逻辑有害，但这是浪费，因为，如果失败是确定性的，则需要花费时间重试事先知道是致命的东西。

Spring批处理提供了无状态RetryPolicy的一些简单通用实现，例如SimpleRetryPolicy和TimeoutRetryPolicy（在前面的示例中使用）。

SimpleRetryPolicy允许对任何已命名的异常类型列表进行重试，最多可重试固定次数。它还有一个永远不应重试的“致命”异常列表，此列表将重写可重试列表，以便可以使用它对重试行为进行更精细的控制，如下例所示：


```
SimpleRetryPolicy policy = new SimpleRetryPolicy();
// Set the max retry attempts
policy.setMaxAttempts(5);
// Retry on all exceptions (this is the default)
policy.setRetryableExceptions(new Class[] {Exception.class});
// ... but never retry IllegalStateException
policy.setFatalExceptions(new Class[] {IllegalStateException.class});

// Use the policy...
RetryTemplate template = new RetryTemplate();
template.setRetryPolicy(policy);
template.execute(new RetryCallback<Foo>() {
    public Foo doWithRetry(RetryContext context) {
        // business logic here
    }
});
```
还有一种更灵活的实现，名为ExpExtCyraseReqyOrdRead，它允许用户通过异常分类器抽象为任意类型的异常类型配置不同的重试行为。该策略通过调用分类器将异常转换为委托RetryPolicy来工作。例如，通过将一个异常类型映射到不同的策略，可以在失败之前比另一个异常类型重试更多次。

用户可能需要实现他们自己的重试策略以获得更自定义的决策。例如，当存在已知的、特定于解决方案的异常分类（分为可重试和不可重试）时，自定义重试策略是有意义的。



## 1.3. Backoff Policies（回退策略）

在暂时性故障后重试时，在重试之前稍等一下通常会有帮助，因为故障通常是由某些问题引起的，而这些问题只能通过等待来解决。如果RetryCallback失败，RetryTemplate可以根据BackoffPolicy暂停执行。

以下代码显示BackOffPolicy接口的接口定义：

```
public interface BackoffPolicy {

    BackOffContext start(RetryContext context);

    void backOff(BackOffContext backOffContext)
        throws BackOffInterruptedException;

}
```

回退策略可以自由地以它选择的任何方式实现回退。Spring Batch提供的开箱即用的策略都使用Object.wait()。一个常见的用例是以指数级增长的等待时间回退，以避免两次重试进入锁定步骤而两次都失败（这是从以太网中吸取的经验教训）。为此，Spring Batch处理提供了ExponentialBackoffPolicy。

## 1.4. Listeners

通常，能够接收跨多个不同重试的横切关注点的额外回调是很有用的。为此，Spring Batch理提供RetryListener接口。RetryTemplate允许用户注册retrylistener，并在迭代期间使用RetryContext和Throwable进行回调。

以下代码显示RetryListener的接口定义：


```
public interface RetryListener {

    <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback);

    <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);

    <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);
}
```
在最简单的情况下，open和close回调出现在整个重试之前和之后，onError应用于各个RetryCallback调用。close方法也可能会收到一个Throwable。如果出现错误，这是RetryCallback抛出的最后一个错误。

注意，当有多个侦听器时，它们在一个列表中，因此有一个顺序。在这种情况下，open的调用顺序与onError和close的调用顺序相反。


## 1.5. Declarative Retry
有时，有一些业务处理，你知道你想重试每次发生。典型的例子是远程服务调用。Spring Batch提供了一个AOP拦截器，它将一个方法调用包装在RetryOperations实现中。RetryOperationsInterceptor执行被截获的方法，并根据提供的RetryTemplate中的RetryPolicy在失败时重试。

以下示例显示了一个声明性重试，该重试使用java配置重试对名为remoteCall的方法的服务调用（有关如何配置AOP侦听器的详细信息，请参阅Spring用户指南）：

```
@Bean
public MyService myService() {
        ProxyFactory factory = new ProxyFactory(RepeatOperations.class.getClassLoader());
        factory.setInterfaces(MyService.class);
        factory.setTarget(new MyService());

        MyService service = (MyService) factory.getProxy();
        JdkRegexpMethodPointcut pointcut = new JdkRegexpMethodPointcut();
        pointcut.setPatterns(".*remoteCall.*");

        RetryOperationsInterceptor interceptor = new RetryOperationsInterceptor();

        ((Advised) service).addAdvisor(new DefaultPointcutAdvisor(pointcut, interceptor));

        return service;
}
```
前面的示例在拦截器中使用默认的RetryTemplate。要更改策略或侦听器，可以将RetryTemplate实例插入侦听器。