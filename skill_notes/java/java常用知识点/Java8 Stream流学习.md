[toc]

# Java8 Stream流学习

## 为什么会有stream？

​	可以让程序实现向sql一样直接。Stream是用函数编程方式 在集合类上进行复杂操作的工具。

## Stream流的定义

Stream 是Java 8新加入的最常用的流接口。（这并不是一个函数式接口。）
特点：

1. Stream流是一个集合元素的函数模型，`其本身并不存储任何元素或地址值`
2. Stream流是JDK1.8之后出现的，具有更优雅的书写方式。
3. 中间操作都会返回流对象本身，这样多个操作可以串联成一个管道。可以对操作进行优化，比如延迟执行和短路.
4. 以前都是使用Iterator进行外部迭代，Stream流提供了内部迭代的方式，流可以直接调用遍历对象。

获取一个流有以下几种常用的方式：

1. 所有的 Collection 集合都可以通过 stream 默认方法获取流；
2. Stream 接口的静态方法 of 可以获取数组对应的流。

例如：

```java
public static void main(String[] args) {
    List<String> strings = List.of("张三丰","周芷若","赵敏","张无忌","张强");
    strings.stream()//Collection类中的方法，返回一个Stream对象
        //Stream<T> filter(Predicate<? super T> predicate)
        //主要用于写过滤器
        .filter( name ->name.startsWith("张"))
        .filter( name ->name.length() ==3)
        //void forEach(Consumer<? super T> action);
        //主要用于对其中的每一个元素遍历
        .forEach( name -> System.out.println(name));
}
```



## Stream流常用方法

常用方法分为两类：延迟方法和终结方法。

### 延迟方法

​	就是返回值仍然是Stream流，所以支持链式调用。

* Stream< T > filter(Predicate<? super T> predicate);

  * 可以通过 filter 方法将一个流转换成另一个Stream流。筛选符合条件的流数据，对数据进行过滤

    ```java
    //使用匿名内部类
    @org.junit.Test
    public void test3(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        stream.filter(new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.endsWith("s");
            }
        }).forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });
    }
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        Stream<String> stream1 = stream.filter(name -> name.endsWith("s"));
        stream1.forEach(name-> System.out.println(name));
        //这时stream已经关闭，数据已经流到stream1 中，stream不能再执行任何方法
        stream.forEach(name-> System.out.println(name));//IllegalStateException
    }
    ```

*  Stream< R > map(Function<? super T, ? extends R> mapper);

  * 将流中的元素映射到另一个流中，把一个类型的数据转换为另一个类型的数据

    ```java
    //使用匿名内部类
    @org.junit.Test
    public void test4(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        stream.map(new Function<String,String>() {
    
            @Override
            public String apply(String s) {
                if (s.endsWith("s")){
                    s = s.substring(0, s.length()-1);
                }
                return s;
            }
    
        }).forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });
    }
    //使用lambda表达式
    @org.junit.Test
    public void test4(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        stream.map(s -> {
            if (s.endsWith("s")){
                s = s.substring(0, s.length()-1);
            }
            return s;
        }).forEach(s -> System.out.println(s));
    }
    ```

* Stream< T > limit(long maxSize);

  * 获取流中的前n个数据，组成新的流,注意这里不是索引值

    ```java
    @org.junit.Test
    public void test6(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        stream.limit(3).forEach(s -> System.out.println(s));
    }
    ```

* Stream< T > skip(long n);

  * 跳过前n个元素，返回新的流

    ```java
    @org.junit.Test
    public void test7(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        stream.skip(2).forEach(s -> System.out.println(s));
    }
    //结果：
    阿拉
    阿朱
    ```

* static Stream concat(Stream<? extends T> a, Stream<? extends T> b)

  * Stream流中的静态方法concat，合并两个流，组合为新流。

    ```java
    @org.junit.Test
    public void test8(){
        Stream<String> stream = Stream.of("阿明", "阿花", "阿拉", "阿朱");
        String[] arr = {"阿明s", "阿花s", "阿拉", "阿朱"};
        Stream<String> stream1 = Arrays.stream(arr);
        Stream.concat(stream, stream1).forEach(s -> System.out.println(s));
    }
    //结果
    阿明
    阿花
    阿拉
    阿朱
    阿明s
    阿花s
    阿拉
    阿朱
    ```

* Stream <T> distinct();

  * 去除重复元素

    ```java
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("阿明", "阿花s", "阿拉", "阿朱");
        String[] arr = {"阿明s", "阿花s", "阿拉", "阿朱"};
        Stream<String> stream1 = Arrays.stream(arr);
    
        Stream.concat(stream,stream1).distinct().forEach(name -> System.out.println(name));
    }
    //结果：
    阿明
    阿花s
    阿拉
    阿朱
    阿明s
    ```

* Stream<T> sorted();

  * 如果是空参，使用自然排序；如果传入比较器，那就用自定义的方式排序

    ```java
    //第一个集合自然排序：小到大
    //第二个集合自定义排序：大到小
    @org.junit.Test
    public void test10(){
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 7, 6);
        integers.stream().sorted().forEach(s-> System.out.println(s));
        List<Integer> integers_01 = Arrays.asList(1, 2, 3, 4, 5, 12,7, 6);
        integers_01.stream().sorted((o1, o2) -> o2-o1).forEach(s-> System.out.println(s));
    }
    ```

    

### 终结方法

​	就是返回值不再是Stream流，所以不能再链式调用了。常用终结方法有count和forEach方法。

* void forEach(Consumer<? super T> action);

  * 该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理。用于遍历元素。

    ```java
    //使用匿名内部类
    @org.junit.Test
    public void test2(){
        Stream<String> stream = Stream.of("阿明", "阿花", "阿拉", "阿朱");
        stream.forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });
    
    }
    //使用lambda表达式
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("阿明", "阿花", "阿拉", "阿朱");
        stream.forEach(name -> System.out.println(new StringBuilder(name).reverse().toString()));
    }
    ```

* long count()

  * 统计流中元素的个数

    ```java
    @org.junit.Test
    public void test5(){
        Stream<String> stream = Stream.of("阿明s", "阿花s", "阿拉", "阿朱");
        long count = stream.count();
        System.out.println(count);
    }
    ```

* Optional<T> max/min(Comparator<? super T> comparator);

  * 获取最大值/最小值

    ```java
    @org.junit.Test
        public void test9(){
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 7, 6);
        Optional<Integer> max = integers.stream().max(Comparator.comparing(value -> value));
        Integer integer = max.get();
        System.out.println(integer);
    }
    ```

    