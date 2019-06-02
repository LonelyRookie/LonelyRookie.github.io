---
layout:     post                    # 使用的布局（不需要改）
title:      Java8之Lambda表达式             # 标题 
subtitle:   Lambda表达式                 #副标题
date:       2019-06-02              # 时间
author:     HuangCanCan             # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
---

# Lambda表达式介绍
## Lambda表达式

Lambda表达式是 Java8 中最重要的新功能之一。使用 Lambda 表达式可以替代只有一个抽象函数的接口实现，告别匿名内部类，代码看起来更简洁易懂。Lambda表达式同时还提升了对集合、框架的迭代、遍历、过滤数据的操作。


## Lambda表达式特点

1、函数式编程；
2、参数类型自动推断；
3、代码量少，简洁


## Lambda表达式好处

1、更简洁的代码；
2、更容易的并行


## Lambda表达式使用场景

任何有函数式接口的地方


## Lambda表达式示例
    
    ` //匿名内部类方式
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("thread run");
        }
    }).start();

    //lambda表达式方式
    new Thread(() -> System.out.println("thread run")).start();


    //匿名内部类方式
    List<String> list = Arrays.asList("webspher", "nginx", "weblogic", "tomcat");
    Collections.sort(list, new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            return o1.length() - o2.length();
        }
    });

    //lambda表达式方式
    Collections.sort(list, (a, b) -> a.length() - b.length());`


# 函数式接口
## 函数式接口描述

只有一个抽象方法（Object类中的方法除外）的接口是函数式接口。
如：下面的Operation接口不是函数式接口

    `public interface Operation {
        int hashCode();

        default int insert() {
            return 1;
        }

        static int update() {
            return 1;
        }
    }`

但是，下面的这个是函数式接口

    `public interface Operation {
        int delete();

        int hashCode();

        default int insert() {
            return 1;
        }

        static int update() {
            return 1;
        }
    }`


## @FunctionalInterface

@FunctionalInterface注解表示标识接口是否是函数式接口。
        
    `@FunctionalInterface
    public interface Operation {
        int delete();
        
        int hashCode();

        default int insert() {
            return 1;
        }
        
        static int update() {
            return 1;
        }
    }`


## jdk1.8之前的函数式接口

java.lang.Runnable

java.util.concurrent.Callable<V>

java.util.Comparator<T>

java.io.Closeable


## jdk 1.8 的函数式接口

jdk1.8下的rt.jar中 java.util.function下都是函数式接口。

常用的函数式接口：
1、Supplier<T> 代表一个输出

2、Consumer<T> 代表一个输入

3、BiConsumer<T,U> 代表两个输入

4、Function<T,R> 代表一个输入，一个输出（一般输入和输出是不同类型的）

5、UnaryOperator<T> 继承Function<T,R> 代表一个输入，一个输出（输入和输出是相同类型的）

6、BiFunction<T,U,R> 代表两个输入，一个输出（一般输入和输出是不同类型的）

7、BinaryOperator<T>继承BiFunction<T,U,R> 代表两个输入，一个输出（输入和输出是相同类型的）


# Lambda表达式详解

[官网](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.27)


## Lambda表达式

Lambda表达式是对象，是一个**函数式接口的实例**.
主要看参数及返回值


## Lambda表达式语法

LambdaParameters -> LambdaBody 

args -> expr  或者 
(Object… args) -> {函数式接口抽象方法实现逻辑}

参数列表：输入的参数

箭头： 箭头是lambda表达式的标志，用于分割参数列表与表达式主体。

Lambda主体 ：便是执行代码主体。Lambda主体就相当于函数体，其可以使用参数列表中的变量。


**()里面参数的个数，根据函数式接口里面抽象方法的参数个数来决定。**

**当只有一个参数的时候，()可以省略。**

**当expr逻辑非常简单的时候，{}和return可以省略。**


## Lambda表达式写法：

1、() -> {}                           // 无参，无返回值

2、
() -> { System.out.println(1); }    // 无参，无返回值

() -> System.out.println(1)       // 无参，无返回值（上面的简写）

3、() -> { return 100; }      // 无参，有返回值

() -> 100                 // 无参，有返回值（上面的简写）

4、
() -> null                  // 无参，有返回值（返回null）

5、
(int x) -> { return x+1; }      // 单个参数，有返回值

(int x) -> x+1         // 单个参数，有返回值（上面的简写）

(x) -> x+1       // 单个参数，有返回值（不指定参数类型，多个参数必须用括号）

x -> x+1           // 单个参数，有返回值（不指定参数类型）



## 错误写法：

(x, int y) -> x+y  

(x, final y) -> x+y 

Object obj = () -> “hello”

Object obj = (Supplier<?>)() -> "hello"

不需要也不允许使用throws语句来声明，它可能会抛出的异常


## 示例

    `Callable<String> callable1 =  new Callable<String>(){
        @Override
        public String call() throws Exception {
            return "hello";
        }
    };
    Callable<String> callable2 =()->{return "hello";};
    Callable<String> callable3 =()-> "hello";
    
    System.out.println(callable1.call());
    System.out.println(callable2.call());
    System.out.println(callable3.call());`


# 方法的引用

方法引用是**用来直接访问类或者实例的已经存在的方法或者构造方法**，方法引用提供了一种**引用而不执行方法的方式**。如果抽象方法的实现恰好可以使用调用另外一个方法来实现，就有可能可以使用方法引用。


## 方法的引用分类

|  类型  |  语法  |  对应的lambda表达式  |
| -----------:|-----------:|-----------:|-------------:|
|  静态方法引用  |  类名::staticMethod  |  (args)->类名.staticMethod(args)  |
|  实例方法引用  |  inst::instMethod  |  (args)->inst.instMethod(args)  |
|  对象方法引用  |  类名::instMethod  |  (inst,args)->类名.instMethod(args)  |
|  构造方法引用  |  类名::new  |  (args)->new 类名(args)  |



## 静态方法引用

如果**函数式接口的实现恰好可以通过调用一个静态方法**来实现，那么就可以使用静态方法引用。

语法：类名::staticMethod

示例：

    `public class Example2 {
        public static String put() {
            System.out.println("put method invoke");
            return "hello";
        }

        public static void size(int size) {
            System.out.println("size：" + size);
        }

        public static String toUpperCase(String str) {
            return str.toUpperCase();
        }

        public static Integer len(String s1, String s2) {
            return s1.length() + s2.length();
        }

        public static void main(String[] args) {

            Supplier<String> s = () -> Example2.put();
            //静态方法引用
            Supplier<String> s2 = Example2::put;  //输出一个参数
            System.out.println(s2.get());

            /**************************************************************/
            Consumer<Integer> c1 = (size) -> Example2.size(size);
            Consumer<Integer> c2 = Example2::size;  //输入一个参数
            c2.accept(100);

            /****************************************************************/
            Function<String, String> f1 = str -> str.toUpperCase();
            Function<String, String> f2 = str -> Example2.toUpperCase(str);
            Function<String, String> f3 = Example2::toUpperCase;//输入一个参数，输出一个值
            System.out.println(f3.apply("lambda"));

            /**************************************************************/
            BiFunction<String, String, Integer> bf1 = (ss1, ss2) -> ss1.length() + ss2.length();
            BiFunction<String, String, Integer> bf2 = (ss1, ss2) -> Example2.len(ss1, ss2);
            BiFunction<String, String, Integer> bf3 = Example2::len;//输入两个参数，输出一个值
            System.out.println(bf3.apply("python", "js"));

        }
    }`


## 实例方法引用

如果**函数式接口的实现恰好可以通过调用一个实例的实例方法**来实现，那么就可以使用实例方法引用

语法：inst::instMethod

示例：

    `public class Example3 {
        public String put() {
            return "hello";
        }

        public void size(int size) {
            System.out.println("size：" + size);
        }

        public String toUpperCase(String str) {
            return str.toUpperCase();
        }

        public Integer len(String s1, String s2) {
            return s1.length() + s2.length();
        }

        public void test() {
            Function<String, String> f3 = this::toUpperCase;//输入一个参数，输出一个值
            System.out.println(f3.apply("lambda"));
        }

        public static void main(String[] args) {
            Supplier<String> s = () -> new Example3().put();
            Supplier<String> s2 = () -> { return new Example3().put(); };
            //实例方法引用
            Supplier<String> s3 = new Example3()::put;  //输出一个参数
            System.out.println(s2.get());

            /**************************************************************/
            Consumer<Integer> c1 = (size) -> new Example3().size(size);
            Consumer<Integer> c2 = new Example3()::size;  //输入一个参数
            c2.accept(100);

            /****************************************************************/
            Function<String, String> f1 = str -> new Example3().toUpperCase(str);
            Function<String, String> f3 = new Example3()::toUpperCase;//输入一个参数，输出一个值
            System.out.println(f3.apply("lambda..."));

            /**************************************************************/
            BiFunction<String, String, Integer> bf2 = (ss1, ss2) -> new Example3().len(ss1, ss2);
            BiFunction<String, String, Integer> bf3 = new Example3()::len;//输入两个参数，输出一个值
            System.out.println(bf3.apply("python", "js"));
        }

    }`


## 对象方法引用

抽象方法的**第一个参数类型刚好是实例方法的类型(类的对象)**，抽象方法**剩余的参数恰好可以当作实例的方法的参数**。如果函数式接口的实现能由上面说的实例方法调用来实现的话，那么就可以使用对象方法引用。

第一个参数类型最好是自定义的类型

语法：类名::instMethod

**抽象方法没有输入参数，不能使用对象方法引用**，如以下函数式接口：

Rnunable run = () -> {};

Closeable c = () -> {};

Supplier<String> s = () -> "";


示例：

    `public class Example4 {

        public static void main(String[] args) {
    //        一个参数类型
            Consumer<Too> c1 = (Too too) -> new Too().foo();
            Consumer<Too> c2 = Too::foo;
    //        Consumer<Too> c3  = Too2::foo; //第一个参数类型不是Too2，而是Too，不一致报错
            c1.accept(new Too());
            c2.accept(new Too());

            /**************************************************/
            //两个参数类型
            BiConsumer<Too2, String> c4 = (too2, str) -> new Too2().fo(str);//第一个参数类型是实例方法的类型，剩余的参数恰好可以当作实例方法的参数
            BiConsumer<Too2, String> c5 = Too2::fo;


            /*********************************************************/
            Exceute e1 = (p, name, size) -> new Prod().run(name, size);
            Exceute e2 = Prod::run;
        }


    }

    class Too {
        public void foo() {
            System.out.println("foo1.........");
        }
    }

    class Too2 {
        public void foo() {
            System.out.println("foo2.........");
        }

        public void fo(String str) {
            System.out.println(str);
        }
    }`


## 构造方法引用

如果**函数式接口的实现恰好可以通过调用一个类的构造方法**来实现，那么就可以使用构造方法引用

语法：类名::new

示例：

    `public class Example5 {

        public static void main(String[] args) {

            Supplier<Account> s1 = () -> new Account();
            Supplier<Account> s2 = Account::new;//无参的构造函数
            s1.get();
            s2.get();
            

            Supplier<List> s3 = ArrayList::new;
            Supplier<Thread> s4 = Thread::new;
            Supplier<Set> s5 = HashSet::new;
            Supplier<String> s6 = String::new;


            Consumer<Integer> c1 = (age) -> new Account(age);
            Consumer<Integer> c2 = Account::new;//有参的构造函数

            Function<String, Integer> fu1 = (str) -> Integer.valueOf(str);
            Function<String, Integer> fu2 = Integer::valueOf;

            Function<String, Account> fu3 = (str) -> new Account();
            Function<String, Account> fu4 = Account::new;

        }
    }


    class Account {
        public Account() {
            System.out.println("Account");
        }

        public Account(int age) {
            System.out.println("Account(age)");
        }

        public Account(String name) {
            System.out.println("Account(name)");
        }
    }`


# Stream API

## Stream

A sequence of elements supporting sequential and parallel aggregate operations(支持顺序和并行聚合操作的一系列元素)
Stream是一组用来处理数组、集合的API

## Stream特性

1、不是数据结构，没有内部存储

2、不支持索引访问

3、延迟计算

4、支持并行

5、很容易生成数组或集合（List，Set）

6、支持过滤，查找，转换，汇总，聚合等操作


## Stream运行机制

Stream分为 源source，中间操作，终止操作

流的源可以是一个数组、一个集合、一个生成器方法，一个I/O通道等等。

一个流可以有零个或者多个中间操作，每一个中间操作都会返回一个新的流，供下一个操作使用。

一个流只会有一个终止操作。

Stream只有遇到终止操作，它的源才开始执行遍历操作。


## Stream常用API

Collectors类中有一些静态方法供使用，如: toList(),toSet()等


### 中间操作

过滤 filter 

去重 distinct 

排序 sorted 

截取 limit、skip(略过，跳过)

转换 map/flatMap

其他 peek(相当于调试)

迭代 iterate

并行流 parallel

顺序流sequential


### 终止操作

循环 forEach

计算 min、max、count、 average

匹配 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny

汇聚 reduce

收集器 toArray collect


### Stream的创建

1、通过数组来创建 Stream.of()方法

2、通过集合来创建 list.stream()方法

3、通过Stream.generate()方法来创建

4、通过Stream.iterate()方法来创建

5、其他API创建

如：

    `String[] arr = {"a", "b", "c", "d"};
    Stream<String> s1 = Stream.of(arr);//通过数组创建Stream

    List<String> asList = Arrays.asList(arr);
    Stream<String> s2 = asList.stream();//通过集合创建Stream

    Stream<Integer> generate = Stream.generate(() -> 1);//通过Stream.generate()方法来创建

    Stream<Integer> iterate = Stream.iterate(1, x -> x + 1);//通过Stream.iterate()方法来创建

    String str = "abcdef123";
    IntStream chars = str.chars();//其他API创建`


遍历输出：

    `String str = "abcdef123";
    IntStream chars = str.chars();//其他API创建
    chars.forEach(x -> System.out.println(x));//遍历输出字符
    chars.forEach(System.out::println);//实例方法引用

    Files.lines(Paths.get("d:/Person.java")).forEach(System.out::println);//读取一个文件输出到控制台`


### 简单练习:

    `Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).forEach(System.out::println);//输出被2整除的数

    int sum = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).mapToInt(x -> x).sum();//把普通stream转换成IntStream
    System.out.println(sum);//求和

    Integer max = Arrays.asList(1, 2, 3, 4, 5).stream().max((a, b) -> a - b).get();
    System.out.println(max);//最大值

    Integer min = Arrays.asList(1, 2, 3, 4, 5).stream().min((a, b) -> a - b).get();
    System.out.println(min);//最小值


    Optional<Integer> op = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).findAny();
    System.out.println(op.get());//查找

    op = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).findFirst();
    System.out.println(op);//查找第一个

    op = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).sorted((a, b) -> b - a).findFirst();
    System.out.println(op.get());//降序排序

    Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).sorted().forEach(System.out::println);//排序`




    `//从1-50里面的所有偶数找出来，放到一个list里面
    List<Integer> list = Stream.iterate(1, x -> x + 1).limit(50).filter(x -> x % 2 == 0).collect(Collectors.toList());
    System.out.println(list);

    //去重
    Arrays.asList(1, 2, 3,4, 2, 4, 5, 6).stream().distinct().forEach(System.out::println);

    //放到set集合
    Set set = Arrays.asList(1, 2, 3,4, 2, 4, 5, 6).stream().collect(Collectors.toSet());
    System.out.println(set);


    //截取前50个，降序排序，忽略前20个，截取10个，放到list中
    list = Stream.iterate(1, x -> x + 1).limit(50).sorted((a, b) -> b - a).skip(20).limit(10).collect(Collectors.toList());
    System.out.println(list);

    //把下列字符串分割，依次转换为int，然后求和
    String str = "11,22,33,44,55";
    int sum = Stream.of(str.split(",")).mapToInt(x -> Integer.valueOf(x)).sum();
    System.out.println(sum);

    sum = Stream.of(str.split(",")).peek(System.out::println).mapToInt(x -> Integer.valueOf(x)).sum();
    System.out.println(sum);`



## 顺序流和并行流

顺序流和并行流取决于parallel,sequential的放置顺序。如果没有parallel,sequential方法默认的是顺序流，程序运行的进程是main。


### 并行流（加parallel后变为并行流）

    `public static void main(String[] args) {
        //设置并行流线程数为5个，但是加上main线程，总的6个
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "5");

        Optional<Integer> max = Stream.iterate(1, x -> x + 1).limit(200).peek(x -> {
        System.out.println(Thread.currentThread().getName());
        }).parallel().max(Integer::compare);
        System.out.println(max);
    }`

输出：五个线程 + 一个主线程

    `ForkJoinPool.commonPool-worker-5
    main
    main
    ForkJoinPool.commonPool-worker-1
    ForkJoinPool.commonPool-worker-1
    ForkJoinPool.commonPool-worker-1
    ForkJoinPool.commonPool-worker-3
    ForkJoinPool.commonPool-worker-3
    ForkJoinPool.commonPool-worker-3
    ForkJoinPool.commonPool-worker-3
    ForkJoinPool.commonPool-worker-3
    ForkJoinPool.commonPool-worker-4
    ForkJoinPool.commonPool-worker-4
    ForkJoinPool.commonPool-worker-2
    ForkJoinPool.commonPool-worker-2
    ...`



### 顺序流（加sequential后变为顺序流）

    `public static void main(String[] args) {
        Optional<Integer> max = Stream.iterate(1, x -> x + 1).limit(200).peek(x -> {
            System.out.println(Thread.currentThread().getName());
        }).sequential().max(Integer::compare);
        System.out.println(max);
    }`

输出：一个主线程

    `main
    main
    main
    ...`


