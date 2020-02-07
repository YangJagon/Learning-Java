# Stream

## 流的创建

1. 用 **Collection** 接口的 **stream** 方法将任何集合转换为一个流

   ```java
   // java.util.Collection<E>
   Stream<E> stream();
   Stream<E> parallelStream();
   ```

2. 用静态的 **Stream.of** 方法或 **Array.stream** 方法将数组转换成流

   ```java
   Stream<String> words = Stream.of(contents.split("\\PL+"));
   
   Stream<String> song = Array.stream(array, from, to);
   
   Stream<String> silence = Stream.empty(); // 创建一个空流
   ```

3. 创建无限流

   ```java
   Stream<String> echos = Stream.generate(() -> "Echo");
   // Stream<Double> randoms = Stream.generate(Math::random);
   
   Stream<Integer> integers = Stream.iterate(0, n -> n+1);
   Stream<Integer> integers = Stream.iterate(0, n->n<10, n -> n+1); //输出 0~9
   ```

## 流的基本操作

```java
// java.util.stream.Stream<T>
Stream<T> filter(Predicate<? super T> p); 
// 产生一个包含当前流中满足P的元素的流

Stream<R> map(Function<? super T, ? extends R> mapper);
// 产生一个流，包含将 mapper 应用于当前流中所有元素所产生的结果（输入T，输出R）

Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
// 产生一个流，它是通过将mapper应用于当前流中所有元素，所产生的所有流连接到一起而获得的。
```

