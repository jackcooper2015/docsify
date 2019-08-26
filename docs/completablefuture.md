CompletableFuture特别是对微服务架构而言，会有很大的作为。举一个具体的场景，电商的商品页面可能会涉及到商品详情服务、商品评论服务、相关商品推荐服务等等。获取商品的信息时（/productdetails?productid=xxx），需要调用多个服务来处理这一个请求并返回结果。这里可能会涉及到并发编程，我们完全可以使用Java 8的CompletableFuture或者RxJava来实现。

`使用demo`
```java

	public List<String> findPriceExecutorsCompletableFuture(String product){
		Executor executor = Executors.newFixedThreadPool(Math.min(shops.size(), 100));
		List<CompletableFuture<String>> priceFuture = shops.stream()
				.map(shop -> CompletableFuture
						.supplyAsync(()-> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)), executor))
				.collect(Collectors.toList());
		return priceFuture.stream().map(CompletableFuture::join).collect(Collectors.toList());
	}
```

[https://my.oschina.net/u/3703858/blog/1799785](https://my.oschina.net/u/3703858/blog/1799785)

`建议如下`:

 * 如果你进行的是计算密集型的操作,并且没有I/O,那么推荐使用Stream接口,因为实现简单,同时效率也可能是最高的
 * 反之,如果你并行的工作单元还涉及等待I/O的操作(包括网络连接等待).那么使用CompletableFuture是灵活性更好,你可以像前面讨论的那样,依据等待/计算,或者W/C的比率设定需要使用的线程数

### Future  Callable例子：
```java
public void renderPage(CharSequence source) {
List<ImageInfo> info = scanForImageInfo(source);
//创建Callable，它代表了下载所有的图片
final Callable<List<ImageData>> task = () ->
  info.stream()
    	.map(ImageInfo::downloadImage)
    	.collect(Collectors.toList());
	// 将下载任务提交到executor
	Future<List<ImageData>> images = executor.submit(task);
	// renderText(source);
try {
   // 获得所有下载的图片（在所有图片可用之前会一直阻塞）
   final List<ImageData> imageDatas = images.get();
   // 渲染图片
   imageDatas.forEach(this::renderImage);
} catch (InterruptedException e) {
   // 重新维护线程的中断状态
   Thread.currentThread().interrupt();
   // 我们不需要结果，所以取消任务
   images.cancel(true);
} catch (ExecutionException e) {
  throw launderThrowable(e.getCause()); }
}
```

---

## CompletableFuture

CompletableFuture类实现了CompletionStage和Future接口。Future是Java 5添加的类，用来描述一个异步计算的结果，但是获取一个结果时方法较少,要么通过轮询isDone，确认完成后，调用get()获取值，要么调用get()设置一个超时时间。但是这个get()方法会阻塞住调用线程，这种阻塞的方式显然和我们的异步编程的初衷相违背。
为了解决这个问题，JDK吸收了guava的设计思想，加入了Future的诸多扩展功能形成了CompletableFuture。
CompletionStage是一个接口，从命名上看得知是一个完成的阶段，它里面的方法也标明是在某个运行阶段得到了结果之后要做的事情。

### 1、进行变换
```java
public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn,Executor executor);
```
首先说明一下已Async结尾的方法都是可以异步执行的，如果指定了线程池，会在指定的线程池中执行，如果没有指定，默认会在ForkJoinPool.commonPool()中执行，下文中将会有好多类似的，都不详细解释了。关键的入参只有一个Function，它是函数式接口，所以使用Lambda表示起来会更加优雅。它的入参是上一个阶段计算后的结果，返回值是经过转化后结果。
例如：
```java
@Test
    public void thenApply() {
        String result = CompletableFuture.supplyAsync(() -> "hello").thenApply(s -> s + " world").join();
        System.out.println(result);
    }

```
结果为：
```java
hello world
```
### 2、进行消耗
```java
public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor);
```
thenAccept是针对结果进行消耗，因为他的入参是Consumer，有入参无返回值。
例如：
```java
@Test
public void thenAccept(){
       CompletableFuture.supplyAsync(() -> "hello").thenAccept(s -> System.out.println(s+" world"));
}
```
结果为：hello world
### 3、对上一步的计算结果不关心，执行下一个操作
```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```
thenRun它的入参是一个Runnable的实例，表示当得到上一步的结果时的操作。
例如：

```java
  @Test
    public void thenRun(){
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenRun(() -> System.out.println("hello world"));
        while (true){}
    }
```

### 4、结合两个CompletionStage的结果，进行转化后返回

```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```
它需要原来的处理返回值，并且other代表的CompletionStage也要返回值之后，利用这两个返回值，进行转换后返回指定类型的值。
例如：

```java
 @Test
    public void thenCombine() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "world";
        }), (s1, s2) -> s1 + " " + s2).join();
        System.out.println(result);
    }

```

### 5、结合两个CompletionStage的结果，进行消耗
```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);
```
它需要原来的处理返回值，并且other代表的CompletionStage也要返回值之后，利用这两个返回值，进行消耗。
例如：
```java
  @Test
    public void thenAcceptBoth() {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenAcceptBoth(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "world";
        }), (s1, s2) -> System.out.println(s1 + " " + s2));
        while (true){}
    }

```
### 6、在两个CompletionStage都运行完执行

```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);
```
不关心这两个CompletionStage的结果，只关心这两个CompletionStage执行完毕，之后在进行操作（Runnable）。

例如：
```java
 @Test
    public void runAfterBoth(){
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).runAfterBothAsync(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s2";
        }), () -> System.out.println("hello world"));
        while (true){}
    }

```

### 7、两个CompletionStage，谁计算的快，我就用那个CompletionStage的结果进行下一步的转化操作
```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);
```
我们现实开发场景中，总会碰到有两种渠道完成同一个事情，所以就可以调用这个方法，找一个最快的结果进行处理。
例如：
```java
@Test
    public void applyToEither() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).applyToEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello world";
        }), s -> s).join();
        System.out.println(result);
    }
```
### 8、两个CompletionStage，谁计算的快，我就用那个CompletionStage的结果进行下一步的消耗操作。

```java
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);
```
例如：
```java
@Test
    public void acceptEither() {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).acceptEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello world";
        }), System.out::println);
        while (true){}
    }
```
### 9、两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）
```java

public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);

```
例如：
```java
 @Test
    public void runAfterEither() {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).runAfterEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s2";
        }), () -> System.out.println("hello world"));
        while (true) {
        }
    }
```
### 10、当运行时出现了异常，可以通过exceptionally进行补偿。
```java
public CompletionStage<T> exceptionally(Function<Throwable, ? extends T> fn);
```
例如：
```java
 @Test
    public void exceptionally() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (1 == 1) {
                throw new RuntimeException("测试一下异常情况");
            }
            return "s1";
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            return "hello world";
        }).join();
        System.out.println(result);
    }
```
### 11、当运行完成时，对结果的记录。这里的完成时有两种情况，一种是正常执行，返回值。另外一种是遇到异常抛出造成程序的中断。这里为什么要说成记录，因为这几个方法都会返回CompletableFuture，当Action执行完毕后它的结果返回原始的CompletableFuture的计算结果或者返回异常。所以不会对结果产生任何的作用。
```java
public CompletionStage<T> whenComplete(BiConsumer<? super T, ? super Throwable> action);
public CompletionStage<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action);
public CompletionStage<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action,Executor executor);
```
例如：
```java
 @Test
    public void whenComplete() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (1 == 1) {
                throw new RuntimeException("测试一下异常情况");
            }
            return "s1";
        }).whenComplete((s, t) -> {
            System.out.println(s);
            System.out.println(t.getMessage());
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            return "hello world";
        }).join();
        System.out.println(result);
    }
```
结果:
```
null
java.lang.RuntimeException: 测试一下异常情况
java.lang.RuntimeException: 测试一下异常情况
hello world
```

### 12、运行完成时，对结果的处理。这里的完成时有两种情况，一种是正常执行，返回值。另外一种是遇到异常抛出造成程序的中断。
```java
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,Executor executor);

```
例如：
出现异常时
```java
@Test
    public void handle() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //出现异常
            if (1 == 1) {
                throw new RuntimeException("测试一下异常情况");
            }
            return "s1";
        }).handle((s, t) -> {
            if (t != null) {
                return "hello world";
            }
            return s;
        }).join();
        System.out.println(result);
    }
```
结果：hello world
未出现异常时
```java
@Test
    public void handle() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).handle((s, t) -> {
            if (t != null) {
                return "hello world";
            }
            return s;
        }).join();
        System.out.println(result);
    }

```
结果为：s1

上面就是CompletionStage接口中方法的使用实例，CompletableFuture同样也同样实现了Future，所以也同样可以使用get进行阻塞获取值，总的来说，CompletableFuture使用起来还是比较爽的，看起来也比较优雅一点。

---
摘自: [https://www.jianshu.com/p/6f3ee90ab7d3](https://www.jianshu.com/p/6f3ee90ab7d3)
[https://leokongwq.github.io/2017/01/17/java8-CompletableFuture.html](https://leokongwq.github.io/2017/01/17/java8-CompletableFuture.html)
[https://www.jdon.com/idea/java/java-8-completablefuture-vs-parallel-stream.html](https://www.jdon.com/idea/java/java-8-completablefuture-vs-parallel-stream.html)
