---
layout:     post                    # 使用的布局（不需要改）
title:      Java8特性之Optional类使用               # 标题
subtitle:   Optional类使用          #副标题
date:       2019-11-02            # 时间
author:     HuangCanCan             # 作者
header-img: img/post-bg-alibaba.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
---



# 前言

这周写代码遇到了一个很大的bug，怪自己学艺不精，无法认清of方法和ofNullable方法的使用，用错了这两个方法。出现了空指针异常。在这里必须记录一下该两个方法的区别。

# 简介

Optional类为了解决NullPointerException（NPE）问题，减少代码中的判空，实现函数式编程，给工程师们提供函数式的API。

我们平时在编码的时候需要不断的判断对象是否为空来做大量的处理，如：

```java
public void errorNullStyle(User user){
    if(user != null){
        Address address = user.getAddress();
        if(address != null){
            City city = address.getCity();
            if(city != null && "".equals(city)){
                // do something....
            }
		}
    }
}

```

上面的代码很容易就变得冗长，难以维护。使用Optional类后如下：

```java
public void errorNullStyle(User user){
    Optional.ofNullable(user).map(User::getAddress).map(Address::getCity).filter(city->!"".equals(city)).ifPresent(city->{
        // do something....
    });
}
```



# 分析Optional类中的方法的使用

Optional类中的方法依赖Objects类，Objects类是一个对象工具类，提供操作对象的方法，如计算对象hash操作；null空值处理以及对象比较等方法。

## empty()

empty()返回一个空的Optional对象，通过new Optional<>()来返回一个空的Optional

```java
    private static final Optional<?> EMPTY = new Optional<>();

    private final T value;

    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
    
    private Optional() {
        this.value = null;
    }

// Optional构造方法是私有的，也就是说不允许外部通过new的形式创建对象。构造方法是供Optional类内部使用。Optional内部维护这个value的变量，无参数构建的时候value为null

```

## of() 、ofNullable()

通过of和ofNullable方法来为Optional中的value赋值，那这两个方法有什么区别呢？看一下源码

of方法的的源码：

```java
// of方法通过带参的私有构造方法创建一个Optional对象，这个构造器都做了哪些操作呢？
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}

// 带参的私有构造方法内部调用Objects的requireNonNull方法来给Optional的value赋值。
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}

public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

```

通过以上源代码分析，我们可以得出结论：

**通过of方法所构造出的Optional对象。当value值为空时，会报NullPointerException异常；当value值不为空时，正常构造Optional对象**。



ofNullable方法的源码：

```java
public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}
```

通过源码可以看出，**ofNullable方法在构造Optional的时候如果value为空，那么返回empty方法构建的Optional对象（一个Optional中value为空的Optional对象）,也就是说ofNullable支持空值得创建**



## get()

get方法就是返回Optional中的value的值

```java
public T get() {
    if (value == null) {
    	throw new NoSuchElementException("No value present");
    }
    return value;
}
```



## isPresent()、ifPresent()

ifPresent方法判断Optional中的value是否为空，不为空返回true；

```java
public boolean isPresent() {
	return value != null;
}
```



而ifPresent方法是在Optional中value不为空的情况下做一些操作。

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
    	consumer.accept(value);
}
```

```java
// 如：user不为null的情况下输出user
Optional.ofNullable(user).ifPresent(System.out::println);
```



## filter()

filter方法在Optional中value不为空的情况下对Optional中的值进行过滤

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
    	return this;
    else
    	return predicate.test(value) ? this : empty();
}
```

```java
// 滤掉Optional中user对象的name值不为“”的返回Optional对象
Optional.ofNullable(user).filter(u->!"".equals(u.getName()));

```



## map()、flatMap()

map和flatMap对Optional中的对象进行转换值的操作，这两个方法唯一的区别就是接受的参数不同。

flatMap处理的参数为Optional类型

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
    	return empty();
    else {
    	return Optional.ofNullable(mapper.apply(value));
    }
}


public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
    
```

```java
// 都是获取Address的写法
Optional.ofNullable(user).map(User::getAddress);
Optional.ofNullable(user).map(u->Optional.ofNullable(u.getAddress()));
```



## orElse()、orElseGet()、orElseThrow()

orElse和orElseGet以及orElseThrow都是处理Optional值为空的情况，如果传入的value为空，进行操作。

orElseThrow在value为空的情况抛出异常。

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
    	return value;
    } else {
    	throw exceptionSupplier.get();
    }
}
```



orElse和orElseGet的区别在与，如果value为空，orElse方法就返回传入的参数other值， orElseGet方法就触发other.get()的调用


```java
// orElse 如果value为空则返回other；
public T orElse(T other) {
	return value != null ? value : other;
}

// orElseGet如果为空则触发other.get()的调用；
public T orElseGet(Supplier<? extends T> other) {
	return value != null ? value : other.get();
}

```

```java
// 在user不为空的情况下，仍然会输出display和创建一个新的User对象
Optional.ofNullable(user).orElse(dispaly());
Optional.ofNullable(user).orElseGet(this::dispaly);
Optional.ofNullable(user).orElse(() -> new Exception("display"));

public User dispaly(){
    System.out.println("display");
    return new User();
}
```



# Java9 Optional类新增三个方法

Java 9 为 Optional 类添加了三个方法：or()、ifPresentOrElse() 和 stream()。

*or()* 方法与 *orElse()* 和 *orElseGet()* 类似，它们都在对象为空的时候提供了替代情况。*or()* 的返回值是由 *Supplier* 参数产生的另一个 *Optional* 对象。

```java
// 如果user变量是 null，它会返回一个 Optional，它所包含的 User 对象，其电子邮件为 “default”。
@Test
public void whenEmptyOptional_thenGetValueFromOr() {
    User result = Optional.ofNullable(user)
      .or( () -> Optional.of(new User("default","1234"))).get();

    assertEquals(result.getEmail(), "default");
}
```



*ifPresentOrElse()* 方法需要两个参数：一个 *Consumer* 和一个 *Runnable*。如果对象包含值，会执行 *Consumer* 的动作，否则运行 *Runnable*。

```java
// 你想在有值的时候执行某个动作，或者只是跟踪是否定义了某个值，那么这个方法非常有用：

// 如果user不为NULL，则输出‘User is:’,否则输出‘User not found’
Optional.ofNullable(user).ifPresentOrElse( u -> logger.info("User is:" + u.getEmail()),
  () -> logger.info("User not found"));
```



*stream()* 方法，它通过把实例转换为 Stream 对象，让你从 *Stream* API 中受益。如果没有值，它会得到空的 *Stream*；有值的情况下，*Stream* 则会包含单一值。

```java
@Test
public void whenGetStream_thenOk() {
    User user = new User("john@gmail.com", "1234");
    List<String> emails = Optional.ofNullable(user)
      .stream()
      .filter(u -> u.getEmail() != null && u.getEmail().contains("@"))
      .map( u -> u.getEmail())
      .collect(Collectors.toList());

    assertTrue(emails.size() == 1);
    assertEquals(emails.get(0), user.getEmail());
}
```