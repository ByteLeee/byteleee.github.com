+++
title="Stream入门"
tags=["java"]
categories=["Java"]
date= 2020-01-11T17:01:19+08:00
+++
# Stream
[toc]
## 0. 前置知识点
* 函数式接口
* lambda表达式和方法引用传递
## 1. 概述
Stream API提供一种”函数式风格的、声明式的“处理数据集合的方式，使用Stream API，可以使得集合处理更加易读（声明式的特征），甚至更加高效（内置的并发处理的特性）。
## 2.命令式和函数式极简对比
> 对1到n(n>1)的自然数求和，命令式和函数式是如何实现的。
* 命令式（Java实现为例）
```java
public int sum(int n){
    int sum = 0;
    for (int i = 1; i <= n; i++) {
        sum+=i;
    }
    return  sum;
}
```
* 函数式（Haskell实现为例）
```hs
sumN::Int -> Int
sumN 0 = 0
sumN x = x + sumN (x-1)
```
再看一下函数式的解决方法脱离语法，是不是接近以下的函数定义：
$$
f(x) = \begin{cases} 
0 &\text{if } (x \leq 0) 
\\
x + f( x-1 ) &\text{if } (x >0) 
\end{cases} 
$$

函数式的定义在描述问题，命令式则是在描述一步一步解决问题。
## 3.Unix管道和流处理
为什么要提到Unix的管道呢，因为从他们的工作流程上看，他们太像了。接触过Linux命令的同学应该很熟悉类似以下的这种命令组合`cat tmp.txt | grep 'test' | sed xx | tail -n 1`,其中的`|`就是管道操作符，它的作用就是将多个应用在”标准输出流”上的操作组合起来，类似一个流水线的处理，针对同一个物品，一步一步加工，最后返回成品。Stream API也有这种特性，看一段实例代码感受一下：
```java
List<Integer> collect = Stream.of(10, 1, 2, 3, 4, 5, 6, 6, 7, 10, 2)
        .filter(x -> x % 2 == 0)
        .distinct()
        .sorted()
        .collect(Collectors.toList());
```
这段代码就很有“声明式”的味道了，有点英文基础的，就能明白这段代码是将一串数字中的偶数去重再排序。
> 如果不使用Stream API，要完成这种操作，该如何编写代码呢？

## 4.Stream中的几个概念
1. 中间操作和终结操作
想象一个流水线，上面的商品经过各种贴标签、注入内容、盖上盖子等机械臂，是不是依然在流水线上徘徊，这种处理前后不改变其所在“位置”的操作，也就是能继续处理的操作，就是中间操作。回到流中就是处理后流还是流，能继续施加流的api。那么终结操作就更好理解了，处于流水线的最后了，收集、打包…使其终结流水线生涯，就是终结操作。回到流中就是归约（比如求和等数学运算取最终结果、取某一个值等等）、收集（收集到容器中供后续使用）等操作，简而言之就是流处理到此为止该看结果了。
2. 有状态和无状态
有无状态实际上要看处于生产线上的物体的性质，在并发编程的范畴里面，状态分可变和不可变，在此处有状态对应状态可变，无状态对应状态不可变；无状态的肯定线程安全的，所以可以放心使用并行处理，而有状态的元素，就需要考虑并发安全性了。
3. 并行操作
Stream提供parallelStream来实现并行处理。

## 5.函数式编程的几个概念
函数式编程（FP）是和面向对象（OOP）不同的编程范式，因为Stream API实际上借鉴的是函数式的思想，在此列出几个概念。
1. 函数，函数式编程世界中，一级公民是函数，怎么理解，OOP到处是对象，FP则处处是函数，函数可以像OOP的对象一样传给函数，Stream API站在lambda表达式的肩膀上，实现了接近传递函数的语义。
2. 无状态，FP中只要定义了变量，一旦赋值就是不可变的，所以没有传递引用的语义，不存在一个对象在一个链路中从头到尾千变万化最后亲妈都认不出来。
3. 惰性计算，这是FP非常重要的特征，所谓惰性计算，就是不到实际要计算的时候，是不会执行的。上面的示例 代码，filter、distinct、sorted这些，并不是对数据遍历了三遍做处理，而是在需要最后collect的时候 ，才真正触发了这些 “函数”。
## 6.Stream API入门 
### 6.1流水线第一步—生成流
*  从集合对象生成流，一般是使用`<Collection>.stream()`就可以得到流对象，这是最为常用的方式
*  使用`Stream.of()`封装
*  使用`Stream.generate()`生成流 （无限流）

![dddf50a0f65071fb9d054f3325f80188.png](evernotecid://3944D88F-A777-4897-BA93-E849C9DFE7EB/appyinxiangcom/6957898/ENResource/p2086)


示例：
```java
public class 生成流 {

    /**
     * date:2019/12/18 11:00 PM
     */
    @Test
    public void testOf() {
        Stream.of(1, 2, 3);
        Stream.of(new int[]{1, 2, 3});
    }

    /**
     * 生成无限流
     * date:2019/12/18 11:01 PM
     */
    @Test
    public void testGenerate() {
        AtomicInteger i = new AtomicInteger(0);
        Stream.generate(i::incrementAndGet);

    }

    /**
     * 生成无限流
     * date:2019/12/18 11:01 PM
     */
    @Test
    public void testIterate() {
        Stream.iterate(1, x -> x + 2);
    }

    /**
     * date:2019/12/18 11:05 PM
     */
    @Test
    public void testFromCollections() {
        Lists.newArrayList(1, 2, 4).stream();
        Sets.newHashSet().stream();
        Maps.newHashMap().entrySet().stream();
        Arrays.stream(new int[]{1, 2, 3});
    }

    /**
     * date:2019/12/18 11:03 PM
     */
    @Test
    public void testFromFileReader() {

        try {
            Stream<String> lines = Files.newBufferedReader(Paths.get("/tmp/test.txt")).lines();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * date:2019/12/18 11:07 PM
     */
    @Test
    public void testStreamBuilder() {
        IntStream intStream = IntStream.builder().add(1).add(2).add(3).build();

    }

    /**
     */
    @Test
    public void testEmpty() {
        Stream.empty();
    }

}

```
### 6.2流水线第二步—操作流
1. 中间操作（intermediate operation）
是施加操作到流上后流依旧是流，可以叠加中间操作。常见的中间操作有：
* 谓词（filter即过滤）
* 映射（map）
* 去重（distinct）
* 排序（sort）
* 扁平化（flatMap）
![f327c65e971c9b21acdf1c948974764d.png](evernotecid://3944D88F-A777-4897-BA93-E849C9DFE7EB/appyinxiangcom/6957898/ENResource/p2087)

2. 终结操作（terminal operation）
终结操作施加在流上就会使流结束，也就是不会再返回Stream对象了，终结操作是使用流的最终目的，不然流水线 永远不产生最终的产品，流水线是没有价值的。
终结操作一般有：
* 收集（collect，findFirst，findAny）
* 归约（reduce）实际上sum(),count()，min(),max()等这些终结操作都只是reduce的具体实现。
* foreach（这个其实可以看做流的可迭代的特性了，不过此方法也会将流终结）
![4bdcd0e6fc76f8ab741c11286e251423.png](evernotecid://3944D88F-A777-4897-BA93-E849C9DFE7EB/appyinxiangcom/6957898/ENResource/p2088)


示例：
```java
public class 操作流 {
 

    /**
     * 1-100中偶数求和
     */
    @Test
    public void testStream() {
        AtomicInteger i = new AtomicInteger(0);
        List<Integer> integers = Stream.generate(i::incrementAndGet).limit(100).collect(Collectors.toList());
        // 1-100中偶数求和
        System.out.println(integers);
        long count = integers.stream().filter(x -> x % 2 == 0).reduce(0, (a, b) -> a + b).longValue();
        System.out.println(count);
    }

    /**
     * 数据去重排序
     * 12:13 PM
     */
    @Test
    public void testSort() {
        List<Integer> collect = Stream.of(10, 1, 2, 3, 4, 5, 6, 6, 7, 10, 2)
                .filter(x -> x % 2 == 0)
                .distinct()
                .sorted()
                .collect(Collectors.toList());
        System.out.println(collect);
    }

    /**
     * reduce相关
     */
    @Test
    public void testReduce() {
        Integer sum = IntStream.of(1, 2, 3).sum();
        System.out.println(sum);
        // 等价于
        sum = Stream.of(1, 2, 3).reduce((a, b) -> a + b).get();
        System.out.println(sum);
    }

    /**
     * 数据映射 7:25 PM
     */
    @Test
    public void testMap() {
        String s = "123,1234,1,1234,15";
        int sum = Arrays.stream(s.split(",")).mapToInt(Integer::valueOf).sum();
        System.out.println(sum);
    }

    /**
     * 12:29 PM
     * 类似mapToStream,将流中的每一个元素转换成stream,其实最后就是返回了一个Stream的Stream
     */
    @Test
    public void testFlatMap() {
        List<String> list = Arrays.asList("5.6", "7.4", "4",
                "1", "2.3");
        list.stream().flatMap(num -> Stream.of(num)).
                forEach(System.out::println);
    }

    /**
     * 扁平化list--->降维
     * date:2019/12/19 12:32 PM
     */
    @Test
    public void testFlatMap2() {
        // Creating a list of Prime Numbers
        List<Integer> PrimeNumbers = Arrays.asList(5, 7, 11, 13);

        // Creating a list of Odd Numbers
        List<Integer> OddNumbers = Arrays.asList(1, 3, 5);

        // Creating a list of Even Numbers
        List<Integer> EvenNumbers = Arrays.asList(2, 4, 6, 8);

        List<List<Integer>> listOfListofInts =
                Arrays.asList(PrimeNumbers, OddNumbers, EvenNumbers);
        // 相当于一个二维的数组[[5, 7, 11,13],[1, 3, 5],[2, 4, 6, 8]]
        System.out.println("The Structure before flattening is : " + listOfListofInts);

        // 降维
        List<Integer> listofInts = listOfListofInts.stream()
                .flatMap(list -> list.stream())
                .collect(Collectors.toList());

        System.out.println("The Structure after flattening is : " + listofInts);
    }

    // 扁平化demo
    static class Student {
        String name;
        List<String> foodList;

        public static void main(String[] args) {
            // 随机生成20个学生数据,每个学生两个菜
            final int nameNo = 0;
            List<Student> students = Stream.generate(() -> {
                Student student = new Student();
                student.name = "学生" + nameNo;
                student.foodList = Lists.newArrayList(Constants.randName(), Constants.randName());
                return student;
            }).limit(20).collect(Collectors.toList());

            // 输出全班同学的所有菜
            List<String> collect = students.stream().flatMap(x -> x.foodList.stream()).distinct().collect(Collectors.toList());
            System.out.println(collect);
        }
    }
}
```
## 7.惰性计算(延迟计算)
java8实战中对Stream延迟计算的描述：
> Java 8的Stream以其延迟性而著称。它们被刻意设计成这样，即延迟操作，有其独特的原因: Stream就像是一个黑盒，它接收请求生成结果。当你向一个 Stream发起一系列的操作请求时，这 些请求只是被一一保存起来。只有当你向Stream发起一个终端操作时，才会实际地进行计算。这 种设计具有显著的优点，特别是你需要对Stream进行多个操作时(你有可能先要进行filter操 作，紧接着做一个map，最后进行一次终端操作reduce);这种方式下Stream只需要遍历一次， 不需要为每个操作遍历一次所有的元素。
```java
    /**
     * 测试是否是惰性计算
     */
    @Test
    public void testLazyCompute() {
        Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8)
                .filter(x -> {
                    System.out.println("filter 1 start");
                    boolean res = x > 0;
                    System.out.println("filter 1 end");
                    return res;
                })
                .filter(x -> {
                    System.out.println("filter 2 start");
                    boolean res = x % 2==0;
                    System.out.println("filter 2 end");
                    return res;
                });
        // 不会执行filter函数
        System.out.println(stream);
        // 会执行filter函数,但是因为findFirst,所以不会遍历全部元素
        Integer integer = stream.findFirst().get();
        System.out.println(integer);

    }
```
输出的结果是：
```
java.util.stream.ReferencePipeline$2@5d5eef3d
filter 1 start,param is:1
filter 1 end
filter 2 start,param is:1
filter 2 end
filter 1 start,param is:2
filter 1 end
filter 2 start,param is:2
filter 2 end
2
```
可见在一次遍历中执行两个filter函数，然后当遍历到第二个元素时，已经取得满足条件的一个元素，所以停止遍历，返回结果。

惰性计算的原理其实就是把一系列“行为”暂存到内存中，在执行终结操作时才将这些行为施加上去，当条件满足即结束操作返回结果。相较于多次遍历的重复计算，惰性计算必然是提高了效率，但是暂存了行为，无疑是一种“空间换时间”的策略使用。

## reference
[1] Java 8实战
[2] 趣学Haskell
[3] [Stream flatMap() in Java with examples](https://www.geeksforgeeks.org/stream-flatmap-java-examples/)
[4] [由浅入深体验 Stream 流](https://www.ibm.com/developerworks/cn/java/j-experience-stream/index.html)
[5] [管道与Unix哲学](https://bindog.github.io/blog/2014/09/25/pipes-and-filters/)
[6] [Java Streams，第 3 部分: Streams 的幕后原理](https://www.ibm.com/developerworks/cn/java/j-java-streams-3-brian-goetz/index.html)
