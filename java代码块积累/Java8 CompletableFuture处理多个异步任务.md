
### CompletableFuture
Java5引入了Future和 FutureTask，用于异步处理。Future可以通过get()方法获取异步的返回值。
在Java8引入了CompletableFuture,CompletableFuture不仅实现了Future接口, 还实现了CompletionStage接口。
CompletableFuture实现了CompletionStage接口，重写thenApply()、thenCombine()等方法。

CompletableFuture类能够处理多个异步任务，还能处理异步回调。还能完成以下操作：
* 将两个异步计算合并为一个——这两个异步计算之间相互独立，同时第二个又依赖于第一个的结果。
* 等待 Future 集合中的所有任务都完成。
* 仅等待 Future集合中最快结束的任务完成（有可能因为它们试图通过不同的方式计算同一个值），并返回它的结果。
* 通过编程方式完成一个Future任务的执行（即以手工设定异步操作结果的方式）。
* 应对 Future 的完成事件（即当 Future 的完成事件发生时会收到通知，并能使用 Future 计算的结果进行下一步的操作，不只是简单地阻塞等待操作的结果）


### 构建CompletableFuture
构建CompletableFuture主要有两种方式，runAsync和supplyAsync。

* runAsync

runAsync异步处理任务，使用Runnable，构建CompletableFuture。

runAsync方法的参数有两种形式，一种是使用默认的ForkJoinPool，另一种是使用自定义的线程池。
第一种源码如下：
```
    private static final Executor asyncPool = useCommonPool ?
        ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();

    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }
```
注意，默认的ForkJoinPool的线程是守护线程，当主线程结束时，ForkJoinPool的线程也会随之结束，会影响异步任务的执行。
因此，建议使用自定义的线程池。

第二种源码如下：
```
    public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor) {
        return asyncRunStage(screenExecutor(executor), runnable);
    }
```
此处的Executor executor就是指线程池。

示例如下：
```
    public static void runAsyncTest2()  {
        //该线程池仅用于示例，实际建议使用自定义的线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            System.out.println("completableFuture runAsync2.");
        }, executorService);
        //        executorService.shutdown();

    }
```

* supplyAsync

supplyAsync同样可以构建CompletableFuture。
supplyAsync跟runAsync的区别，主要是supplyAsync有返回值。

```
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```
此处的参数supplier其实是一个匿名对象，实现了Supplier<T>接口，重写get()方法并返回值。
```
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
Supplier是函数式接口，关于函数式接口的理解，详情见参考资料: [java8 函数式接口](https://www.cnblogs.com/expiator/p/14841355.html)

示例如下：
```
    /**
     *   supplyAsync使用线程池，构建CompletableFuture
     *
     */
    public static void supplyAsyncTest2() {
        ExecutorService executorService = Executors.newCachedThreadPool();
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("supplyAsyncTest2");
            return "completableFuture结果";
        }, executorService);
//        executorService.shutdown();
    }
```

### 获取CompletableFuture任务的结果
```
T get() throws InterruptedException, ExecutionException： 获取返回值，会阻塞，还需要处理受检的异常

V get(long timeout,Timeout unit)：可以设置阻塞时间，unit为时间的单位(秒/分/时之类)

T getNow(T defaultValue)：表示当有了返回结果时会返回结果,如果异步线程抛了异常会返回设置的默认值.

T join()：获取返回值，会阻塞
```

在实际编程中，不建议使用CompletableFuture的 get() 方法，
最好用get(long timeout,Timeout unit) 设置超时时间，这样不会一直阻塞。
```
    public static void supplyAsyncGet()  {
        //该线程池仅用于示例，实际建议使用自定义的线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(()
                -> runTask(), executorService);

        String result = null;
        try {
            //获取返回值，2秒超时
            result = completableFuture.get(2, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            logger.error("InterruptedException error.", e);
        } catch (ExecutionException e) {
            logger.error("ExecutionException error.", e);
        } catch (TimeoutException e) {
            logger.error("TimeoutException error.", e);
        }
        logger.info("result:"+result);
    }

    private static String runTask() {
        try {
            //任务耗时。可以分别设置1000和3000，看未超时和超时的不同结果。
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            logger.error("supplyAsyncGet error.");
        }
        return "supplyAsyncGet";
    }

```

### CompletableFuture任务执行中

thenApply()： 接收一个任务的前一阶段的输出作为本阶段的输入，该方法有一个参数，也有返回值。
通过thenApply()，可以在supplyAsync()异步完成后，马上就使用supplyAsync()的返回值，不会阻塞。
```
    public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
```
thenAccept()： 接收一个任务的前一阶段的输出作为本阶段的输入，该方法返回值类型为Void，相当于没有返回值。
```
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
```
thenRun()： 根本不关心一个任务的前一阶段的输出，它只负责运行新的Runnable任务，该方法返回值类型为Void，相当于没有返回值。
```
    public CompletableFuture<Void> thenRun(Runnable action) 
```


示例如下：
```
    public static void thenApplyTest()  {
        CompletableFuture.supplyAsync(() -> 1)
                .thenApply(i -> i + 1).thenApply(i -> {
                    System.out.println("thenApplyTest结果为："+i*i);
                    return i * i;
                });
    }

```

### CompletableFuture任务完成后

whenComplete()：任务完成后触发，该方法有返回值。还有两个参数，第一个参数是任务的返回值，第二个参数是异常。
```
public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action)
```
exceptionally()：当运行出现异常时,调用该方法可进行一些补偿操作,设置默认值.如果没有异常，则不会触发。
```
public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
```

示例如下：
```
    public static void exceptionallyTest() {
        // num作为分母，如果改成1，.exceptionally()方法就不会执行
        Integer num = 0;
        CompletableFuture<Integer> exceptionCf = CompletableFuture.supplyAsync(() -> 1)
                .thenApply(i -> i / num)
                .whenComplete((i, e) -> {
                    System.out.println(i);
                })
                .exceptionally(e -> {
                    System.out.println("exceptionallyTest error. "+ e);
                    System.out.println("异常处理");
                    return 0;
                });
        Integer result = exceptionCf.join();
        System.out.println("exceptionCf结果为："+result);
    }
```


### 多个CompletableFuture任务的组合

thenCombine()： 会将两个任务(CompletableFuture)的执行结果作为方法入参传递到指定方法中，且该方法有返回值；
```
public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn)
```

thenAcceptBoth()： 同样将两个任务(CompletableFuture)的执行结果作为方法入参，但是是个Void的返回值，相当于没有返回值；
```
 public <U> CompletableFuture<Void> thenAcceptBoth(
        CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action)
```
runAfterBoth()： 在两个任务(CompletableFuture)之后执行，但没有入参，而且是个Void的返回值，相当于没有返回值。
```
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other, Runnable action)
```

示例如下：
```
    public static void thenCombineTest() {
        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(()-> {
            System.out.println("任务1结束.");
            return "Hello";
        });

        CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(()-> {
            System.out.println("任务2结束.");
            return "World";
        });

        cf1.thenCombine(cf2, (result1, result2) -> result1 + result2)
            .whenComplete((r,e)-> System.out.println("任务1的返回值加上任务2的返回值，结果为："+r));

    }

```

### CompletableFuture任务执行后
allOf就是所有任务都完成时触发。但是是个Void的返回值，相当于没有返回值。

```
  //...表示不定参数，可以有多个CompletableFuture参数
  public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
```

anyOf是当入参的completableFuture组中有一个任务执行完毕就返回。返回结果是第一个完成的任务的结果。
```
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```

示例如下：
```
    public static void allOfTest() {
        ExecutorService executorService = Executors.newCachedThreadPool();
        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务1结束.");
            return "futureOneResult";
        }, executorService);
        CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务2结束.");
            return "futureTwoResult";
        },executorService);
        CompletableFuture.allOf(cf1, cf2).thenRun(()->{
            System.out.println("任务1和任务2都完成了");
        });

//        CompletableFuture completableFuture = CompletableFuture.anyOf(futureOne, futureTwo);
//        //返回结果是第一个完成的任务的结果
//        System.out.println(completableFuture.get());

    }
```

### 参考资料
《java8实战》
https://www.jianshu.com/p/547d2d7761db
https://www.cnblogs.com/fingerboy/p/9948736.html
https://www.jianshu.com/p/6bac52527ca4
