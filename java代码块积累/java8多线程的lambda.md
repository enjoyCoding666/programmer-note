### Lambda表达式
lambda表达式，实际上就是匿名函数。

格式如下：
()里面是函数的参数，中间是箭头， {}是函数的代码块，{}包含了函数的执行以及返回结果。
```
()->{}
```

### 新建线程
* 不使用lambda：
```
  Runnable runnable = new Runnable() {
      @Override
      public void run() {
          System.out.println("执行run()方法.");
      }
  };
```
使用lambda：
```
  Runnable runnable = () -> System.out.println("执行run()方法.");
```

### 提交任务到线程池：
* 不使用lambda：
```
    //实战建议使用ThreadPoolExecutor自定义线程池，避免OOM，此处是为了方便示例
    ExecutorService executor = Executors.newFixedThreadPool(5);
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("执行任务.");
        }
    };
    executor.execute(runnable);
```
* 使用lambda：
```
    ExecutorService executor = Executors.newFixedThreadPool(5);
    executor.execute(() -> {
        System.out.println("执行任务.");
    });
```


### 线程池异步并获取结果
* 不使用lambda：

```
    ExecutorService executor = Executors.newFixedThreadPool(5);
    executor.submit(new Callable<String>() {
        @Override
        public String call()  {
            System.out.println("执行异步任务.");
            return "异步结果";
        }
    });
```
使用lambda简化：
```
    ExecutorService executor = Executors.newFixedThreadPool(5);
    Future<String> future = executor.submit(() -> {
        System.out.println("执行异步任务.");
        return "异步结果";
    });
    String result = future.get();
```