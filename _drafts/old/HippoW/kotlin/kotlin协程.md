# Coroutines(协程) in kotlin & java

## 0. 目录

[TOC]

## 1. 什么是协程

协程把异步编程放入库中来简化这类操作。程序逻辑在协程中**顺序表述**，而底层的库会将其**转换为异步操作**。库会将相关的用户代码打包成回调，订阅相关事件，调度其执行到不同的线程（甚至不同的机器），而代码依然想顺序执行那么简单。

一般情况下我们写一个异步回调是这样的（层层嵌套回调）：

```
service.login(phone, password, new LoginCallback() {
	@Override
   public void onCompleted(UserInfo userInfo) {
   		saveUserInfo(userInfo);
   		uiHanlder.post(new Runnable() {
   			@Override
   			public void run() {
   				// Do UI Operation
   				showUser(userInfo);
   			}
   		});
   }

   @Override
   public void onError(Throwable e) {
   }
});
```

如果使用RxJava，会是这样的（链式调用避免回调嵌套，但是还是依赖回调机制）：

```
service.login(phone, password)
				.subscribeOn(Schedulers.newThread())
				.observeOn(Schedulers.io())
				.doOnNext(new Action1<UserInfo>() {
					@Override
					public void call(UserInfo userInfo) {
						saveUserInfo(userInfo);//保存用户信息到本地
					}
				})
				.observeOn(AndroidSchedulers.mainThread())
				.subscribe(new Subscriber<UserInfo>() {
					@Override
					public void onCompleted(UserInfo userInfo) {
						// Do UI Operation
						showUser(userInfo);
					}
					@Override
					public void onError(Throwable e) {
					}
				});
```

使用协程，是这样的（**回归顺序调用！**）：

```
async(UI) {
    val userInfo: Deferred<UserInfo> = bg {
        // Runs in background
        UserInfo userInfo = login(phone, password)
        saveUserInfo(userInfo);
    }

    // This code is executed on the UI thread
    showUser(userInfo.await())
}
```

`bg {} `所包裹的`login()`与`saveUserInfo()`函数是跑在background的，而接下来在 UI thread 上执行的代码直接引用了`saveUserInfo()`返回的对象。这在异步线程中按理说无法实现。

其实关键在于`bg {} `包裹的代码块最终返回的是一个`Deferred`对象，而这个`Deferred`对象的`await`函数在这里起到了关键作用 —— 阻塞当前的协程，等待结果。

最外层的`async(UI) {} `，则是指在UI线程上开辟一条**异步的协程任务**。因为是异步的，哪怕被阻塞了也不会导致整个UI线程阻塞；因为还是在UI线程上的，所以我们可以放心的做UI操作。相应的，`bg {} `可以理解为`async(BACKGROUND) {} `，所以可以在后台线程中执行网络请求。

## 2. 协程 in Kotlin

### 2.1. 线程阻塞与协程挂起

协程是通过编译技术实现(不需要虚拟机VM/操作系统OS的支持)，通过插入相关代码来生效。与之相反，线程/进程是需要虚拟机VM/操作系统OS的支持，通过调度CPU执行生效。

线程阻塞的代价昂贵，尤其在高负载时的可用线程很少，阻塞线程会导致一些重要任务缺少可用线程而被延迟。

协程挂起几乎无代价，无需上下文切换或涉及OS，最重要的是，协程挂起可由用户控制:可决定挂起时发生什么，并根据需求优化/记录日志/拦截。

另一个不同之处是，协程不能在随机指令中挂起，只能在挂起点挂起(调用标记函数)!

### 2.2. suspend 关键字

当调用`suspend`修饰的函数时会发生协程挂起:

```
suspend fun doSomething(foo: Foo): Bar {           
}
```        

该函数称为挂起函数，调用它们可能挂起协程(如果调用结果已经可用，协程库可决定不挂起)
挂起函数能像普通函数获取参数和返回值，但只能在协程/挂起函数中被调用。

### 2.3. 协程上下文

**`CoroutineContext`**

字面上理解是协程上下文，表明代码块执行在哪个场景。

**`CoroutineDispatcher`**

调度器，由它来调度和处理任务。这是一个抽象基类。它有如下几个实现类：

```
// 通过线程池调度
CommonPool
// 通过Handler调度
HandlerContext
// 通过当前线程调度
Unconfined

// 安卓开发特定 
val UI = HandlerContext(Handler(Looper.getMainLooper()), "UI")
```

**`Job` & `Deferred`**

由调度器执行的代码块称为一个Job。它是个接口，有如下几种生命周期：

| state | isActive | isCompleted | 
| ----- |:--------:|:-----------:|
| new   | F        | F           |
| active | T       | F           |
| completed | F    | T           |

`Deferred`是`Job`的一个扩展提供取消、异常等周期：

| state | isActive | isCompleted | isCompletedExceptionally | isCancelled |
| ----- |:--------:|:-----------:|:-----------:|:-----------:|
| new   | F        | F           | F           | F           |
| active | T       | F           | F           | F           |
| resolved | F     | T           | F           | F           |
| failed | F       | T           | T           | F           |
| cancelled | F    | T           | T           | T           |

Defferred有两个实现类：

```
// 任务创建后会立即启动
DeferredCoroutine
// 任务创建后new的状态，要等用户调用 start() or join() or await()去启动他
LazyDeferredCoroutine
```

### 2.4. 协程的创建与使用

协程的创建与使用主要API只有两个：`async`，`launch`。

```
public fun <T> async(context: CoroutineContext, start: Boolean = true, block: suspend CoroutineScope.() -> T) : Deferred<T> {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start)
        DeferredCoroutine<T>(newContext, active = true) else
        LazyDeferredCoroutine(newContext, block)
    coroutine.initParentJob(context[Job])
    if (start) block.startCoroutine(coroutine, coroutine)
    return coroutine
}


fun launch(context: CoroutineContext, start: Boolean = true, block: suspend CoroutineScope.() -> Unit): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start)
        StandaloneCoroutine(newContext, active = true) else
        LazyStandaloneCoroutine(newContext, block)
    coroutine.initParentJob(context[Job])
    if (start) block.startCoroutine(coroutine, coroutine)
    return coroutine
}
```

两个方法差不多，都是启动一个协程，不同的是 返回值，正如它们方法名一样，async是异步，可以取消。launch就是启动一个协程。

安卓下常规写法如下：

```
val 异步任务 = async(CommonPool,false) {
    //需要线程中执行的代码块 
}
//上面的异步任务 参数false 说明需要调用 await()方法区启动这个任务
launch(UI) {  val 返回值 = 异步任务.await() }

//launch 默认参数 start= true 所以会立即执行
```

## 2.5. 更多介绍

https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md

## 3. 应用

### 3.1. 一个更复杂的demo

接下来看一个更复杂的应用：

我们假设有两个函数：`f1()`和`f2()`，二者都会在一段延迟之后返回一个数。调用这两个函数，将其返回值求和并呈现给用户。然而如果`500ms`之内没有返回的话，就抛弃返回值，此外我们会在有限次数内进行重试，超过重试次数则最终放弃请求。

首先，写两个函数来实现延迟操作：

```
suspend fun f1(i: Int) {
    Thread.sleep(if (i != 2) 2000L else 200L)
    return 1;
}

suspend fun f2(i: Int) {
    Thread.sleep(if (i != 2) 2000L else 200L)
    return 2;
}
```

协程实现：

```
suspend fun coroutineWay() {
    var i = 0;
    while (true) {                                       
        var v1 = async(CommonPool) { f1(i) }             
        var v2 = async(CommonPool) { f2(i) }
        var v3 = launch(CommonPool) {                    
            Thread.sleep(500)
            val te = TimeoutException();
            v1.cancel(te);                               
            v2.cancel(te);
        }

        try {
            val r1 = v1.await();                         
            val r2 = v2.await();
            v3.cancel();                                 
            break;                                       
        } catch (ex: TimeoutException) {                 
            if (++i > 2) {                               
                throw ex;
            }
        }
    }
}
```

可以想想如果使用普通的嵌套回调方式，或者RxJava会如何实现？

下面给出一个RxJava的例子：

```
fun reactiveWay() {
    RxJavaPlugins.setErrorHandler({ })                         

    val sched = BlockingScheduler()                            
    sched.execute {
        val t0 = System.currentTimeMillis()
        val count = Array<Int>(1, { 0 })                       

        Single.defer({                                        
            val c = count[0]++;
            Single.zip(                                        
                    Single.fromCallable({ f3(c) })
                        .subscribeOn(Schedulers.io()),
                    Single.fromCallable({ f4(c) })
                        .subscribeOn(Schedulers.io()),
                    BiFunction<Int, Int> { a, b -> a + b }     
            )
        })
        .doOnDispose({                                         
            // cancel
        })
        .timeout(500, TimeUnit.MILLISECONDS)                   
        .retry({ x, e ->
            x < 3 && e is TimeoutException                     
        })
        .doAfterTerminate { sched.shutdown() }                
        .subscribe({
            // end             
        },
        { it.printStackTrace() })
    }
}
```

### 3.2. 协程的业界应用

**C/C++协程库libco：微信怎样漂亮地完成异步化改造**
http://www.infoq.com/cn/articles/CplusStyleCorourtine-At-Wechat

## 4. 工作机制

协程完全通过编译技术实现(不需要来自 VM 或 OS 端的支持)，挂起通过代码来生效。基本上，每个挂起函数(优化可能适用，但我们不在这里讨论)都转换为状态机，其中的状态对应于挂起调用。刚好在挂起前，下一状态与相关局部变量等一起存储在编译器生成的类的字段中。在恢复该协程时，恢复局部变量并且状态机从刚好挂起之后的状态进行。

挂起的协程可以作为保持其挂起状态与局部变量的对象来存储和传递。这种对象的类型是 Continuation，而这里描述的整个代码转换对应于经典的延续性传递风格(Continuation-passing style)。因此，挂起函数有一个 Continuation 类型的额外参数作为高级选项。

关于协程工作原理的更多细节可以在这个设计文档中找到。在其他语言(如 C# 或者 ECMAScript 2016)中的 async/await 的类似描述与此相关，虽然它们实现的语言功能可能不像 Kotlin 协程这样通用。