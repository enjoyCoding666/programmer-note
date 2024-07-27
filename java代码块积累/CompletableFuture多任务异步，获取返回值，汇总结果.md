### 线程池异步的基础知识
详情见：https://blog.csdn.net/sinat_32502451/article/details/133039624

### 线程池执行多任务，获取返回值
线程池的 submit()方法，可以提交任务，并返回 Future接口。
而 future.get()，可以获取到任务的结果，但是get()方法会阻塞，阻塞时间过长，会占用过多的系统资源。
因此在使用时，一般都会用 get(long timeout, TimeUnit unit) 设置超时时间。
```
//该线程池仅用于示例，实际建议使用自定义的线程池
ExecutorService executor = Executors.newCachedThreadPool();
Future<String> future = executor.submit(() -> "task1");
//阻塞，获取返回值，2秒超时
String result = future.get(2, TimeUnit.SECONDS);
```
不过，get(long timeout, TimeUnit unit) 比较适合设置单个任务的超时时间，在多任务的情况下，哪怕设置了超时时间，阻塞的时间也会特别长。
比如，有5个任务同时执行，每个任务设置2s的超时时间，在极端情况下，这些任务全部阻塞并超时，那总共要耗费的时间，可能会达到10s，这明显是不能接受的。
如果超时时间设置得太小，又可能出现频繁超时。在多任务获取返回值的场景，更适合使用 CompletableFuture。

### CompletableFuture基础知识
详情见： https://blog.csdn.net/sinat_32502451/article/details/132819472

### CompletableFuture多任务异步
CompletableFuture多任务异步，不需要返回值的话，主要使用 allOf()。
* allOf()：就是将多个任务汇总成一个任务，所有任务都完成时触发。allOf()可以配合get()一起使用。

```
public static void allOfTest() throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        CompletableFuture<Void> cf1 = CompletableFuture.runAsync(
                () -> System.out.println("cf1 ok."), executorService);
        CompletableFuture<Void> cf2 = CompletableFuture.runAsync(
                () -> System.out.println("cf2 ok."), executorService);
        
        //将两个任务组装成一个新的任务，总共的超时时间为2s
        CompletableFuture.allOf(cf1, cf2).get(2, TimeUnit.SECONDS);
    }
```

### CompletableFuture获取返回值
只有一个任务时，CompletableFuture的使用，跟线程池异步有点类似。
主要用到 CompletableFuture.supplyAsync(): 异步处理任务，有返回值。
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
        } catch (Exception e) {
            logger.error("completableFuture.get error.", e);
        }
        logger.info("result:"+result);
    }

    private static String runTask() {
        try {
            //任务耗时。可以分别设置1000和3000，看未超时和超时的不同结果。
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            logger.error("runTask error.", e);
        }
        return "taskResult";
    }
```

### CompletableFuture多任务异步，获取返回值，汇总结果
有几个方法比较关键：
* supplyAsync(): 异步处理任务，有返回值
* whenComplete()：任务完成后触发，该方法有返回值。还有两个参数，第一个参数是任务的返回值，第二个参数是异常。
* allOf()：就是所有任务都完成时触发。allOf()可以配合get()一起使用。

示例如下：
```
    /**
     *  异步，多任务。汇总返回值
     */
    public static void allOfGet()  {
        //该线程池仅用于示例，实际建议使用自定义的线程池
        ExecutorService executorService = Executors.newCachedThreadPool();

        //线程安全的list，适合写多读少的场景
        List<String> resultList = Collections.synchronizedList(new ArrayList<>(50));
        CompletableFuture<String> completableFuture1 = CompletableFuture.supplyAsync(
                () -> runTask("result1", 1000), executorService)
                .whenComplete((result, throwable) -> {
                    //任务完成时执行。用list存放任务的返回值
                    if (result != null) {
                        resultList.add(result);
                    }
                    //触发异常
                    if (throwable != null) {
                        logger.error("completableFuture1  error:{}", throwable);
                    }
                });

        CompletableFuture<String> completableFuture2 = CompletableFuture.supplyAsync(
                () -> runTask("result2", 1500), executorService)
                .whenComplete((result, throwable) ->{
                    if (result != null) {
                        resultList.add(result);
                    }
                    if (throwable != null) {
                        logger.error("completableFuture2  error:{}", throwable);
                    }

                });

        List<CompletableFuture<String>> futureList = new ArrayList<>();
        futureList.add(completableFuture1);
        futureList.add(completableFuture2);

        try  {
            //多个任务
            CompletableFuture[] futureArray = futureList.toArray(new CompletableFuture[0]);
            //将多个任务，汇总成一个任务，总共耗时不超时2秒
            CompletableFuture.allOf(futureArray).get(2, TimeUnit.SECONDS);
        } catch (Exception e) {
            logger.error("CompletableFuture.allOf Exception error.", e);
        }
        List<String> list = new ArrayList<>(resultList);

        list.forEach(System.out::println);
    }


    private static String runTask(String result, int millis) {
        try {
            //此处忽略实际的逻辑，用sleep代替
            //任务耗时。可以分别设置1000和3000，看未超时和超时的不同结果。
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            logger.error("supplyAsyncGet error.");
        }
        return result;
    }

```

### 相关资料：
https://blog.csdn.net/sinat_32502451/article/details/132819472

