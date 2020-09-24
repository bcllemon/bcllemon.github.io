layout: post
title: "异常处理"
date: "2020-09-24 17:10"
categories: 技术
---
# 错误返回码
处理错误最直接的方式是通过错误码，这也是传统的方式，在过程式语言中通常都是用这样的方式处理错误的.基本上来说，其通过函数的返回值标识是否有错，然后通过全局的errno变量并配合一个 errstr 的数组来告诉你为什么出错。
# 捕获异常
try - catch - finally 异常处理
* 函数接口在 input(参数)和 output(返回值)以及错误处理的语义是比较清楚的。 
* 正常逻辑的代码可以与错误处理和资源清理的代码分开，提高了代码的可读性。 
* 异常不能被忽略(如果要忽略也需要 catch 住，这是显式忽略)。 
* 在面向对象的语言中(如 Java)，异常是个对象，所以，可以实现多态式的 catch。

# 错误返回码 vs 异常捕捉
* 对于我们并不期望会发生的事，我们可以使用异常捕捉;
* 对于我们觉得可能会发生的事，使用返回码。

# 异步的错误处理
在异步编程的世界里，因为被调用的函数是被放到了另外一个线程里运行，这将导致: 
* 无法使用返回码。因为函数在被异步运行中，所谓的返回只是把处理权交给下一条指令，而不是把函数运行完的结果返回。所以，函数返回的语义完全变了，返回码也没有用了。
* 无法使用抛异常的方式。因为除了上述的函数立马返回的原因之外，抛出的异常也在另外一 个线程中，不同线程中的栈是完全不一样的，所以主线程的 catch 完全看不到另外一个线程 中的异常。

## Callback 方式
通过注册错误处理的回调函数，让异步执行的函数在出错的时候，调用被注册进来的错误处理函数，这样的方式比较好地解决了程序的错误处理。而出错的语义从返回码、异常捕捉到了直接耦合错误出处函数的样子。

## Promise 模式
```
doSomething()
.then(result => doSomethingElse(result)) 
.then(newResult => doThirdThing(newResult))
.then(finalResult => {
	console.log(`Got the final result: ${finalResult}`);
})
.catch(failureCallback);
```
上面代码中的 then() 和 catch() 方法就是 Promise对象的方法，then()方法可以把各个异步的函数给串联起来，而catch() 方法则是出错的处理。
### Java 异步编程的 Promise 模式
链式处理:
```
CompletableFuture.supplyAsync(this::findReceiver)
					.thenApply(this::sendMsg)
					.thenAccept(this::notify);
```
异常处理:
```
CompletableFuture.supplyAsync(Integer::parseInt)
				 .thenApply(r -> r * 2 * Math.PI)
				.thenApply(s -> "apply>> " + s)
				.exceptionally(ex -> "Error: " + ex.getMessage());
```
混合处理:
```
CompletableFuture.supplyAsync(Integer::parseInt) 
				.thenApply(r -> r * 2 * Math.PI)
				.thenApply(s -> "apply>> " + s) .handle((result, ex) -> {
						if (result != null) {
                         return result;
                     } else {                     	
						return "Error handling: " + ex.getMessage();
					}
				});
```

#最佳实践
* 统一分类的错误字典。无论你是使用错误码还是异常捕捉，都需要认真并统一地做好错误的分类。最好是在一个地方定义相关的错误。
* 定义错误的严重程度。
* 错误日志的输出最好使用错误码，而不是错误信息。打印错误日志的时候，除了要用统一的格式，最好不要用错误信息，而使用相应的错误码，错误码不一定是数字，也可以是一个能从错误字典里找到的一个唯一的可以让人读懂的关键字。这样，会非常有利于日志分析软件进行自动化监控，而不是要从错误信息中做语义分析。
* 忽略错误最好有日志
* 对于同一个地方不停的报错，最好不要都打到日志里，打出一个错误以及出现的次数。
* 尽可能在错误发生的地方处理错误。
* 处理错误时，总是要清理已分配的资源。
* 为你的错误定义提供清楚的文档以及每种错误的代码示例
