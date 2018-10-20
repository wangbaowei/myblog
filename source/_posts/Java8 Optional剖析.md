---
title: Java8 Optional剖析
author: 争夕
date: 2017-08-16 10:05:55
categories:
- 后端
- JDK
tags:
- Java8
- Optional
---



## Optional存在的意义 ##

在java8出现之前，通常在调用一个对象的方法的时候，难免会判断对象是否为Null，只有不为null才敢去调用对象的方法，这是一种安全措施，如果不注重这个null的情况，会常常出现NullPointException。相信java程序员对这个异常并不陌生。而java8出现之后使用到了Optional这个接口，它用一种更优雅的方式来处理这种问题，并且使得程序可读性更高。

## of ##
为非null的值创建一个Optional。
通过Optional的静态方法of来创建Optional实例。注意：传入的参数不能为null。如果传入参数为null，则抛出NullPointerException 。

```java
    @Test
    public void testOf() {
        Optional<String> helloWorld = Optional.of("Hello World");
        Optional<String> nullOptional = Optional.of(null);
    }
```

## ofNullable ##
为指定的值创建一个Optional，如果指定的值为null，则返回一个空的Optional。
ofNullable与of唯一的区别是可以接受null。

```java
    @Test
    public void testOfNullable() {
        Optional<String> nullOptional = Optional.ofNullable(null);
    }
```

## isPresent ##
如果值存在返回true，否则返回false。以此可用来判断optional实例所封装的对象是否为null。

```java
    @Test
    public void testIsPresent() {
        Optional<String> hello = Optional.ofNullable("Hello World");
        System.out.println(hello.isPresent());
        Optional<String> nullHello = Optional.ofNullable(null);
        System.out.println(nullHello.isPresent());
    }
```

## ifPresent ##
如果Optional实例有值则为其调用consumer，否则不做处理。

这个方法让我们可以以此替代原来的判空语句，在非空的情况下做某些事情，不再显示用到if语句。
```java
    @Test
    public void testIfPresent() {
        Optional<String> hello = Optional.ofNullable("Hello World");
        hello.ifPresent(value -> System.out.println("present!"));
        Optional<String> nullHello = Optional.ofNullable(null);
        nullHello.ifPresent(value -> System.out.println("present too?"));
    }
```

## orElse ##
如果有值则将其返回，否则返回指定的其它值。
```java
    @Test
    public void testOrElse() {
        Optional<String> hello = Optional.ofNullable("Hello World");
        String str1 = hello.orElse("I will not be show");
        System.out.println(str1);
        Optional<String> nullHello = Optional.ofNullable(null);
        String str2 = nullHello.orElse("I am null");
        System.out.println(str2);
    }
```

## orElseGet ##

orElseGet与orElse方法类似，区别在于orElseGet方法接受Supplier接口的实现用来生成默认值。
```java
    @Test
    public void testOrElseGet() {
        Supplier<String> strSupplier = () -> "Default Str";

        Optional<String> hello = Optional.ofNullable("Hello World");
        String str1 = hello.orElseGet(strSupplier);
        System.out.println(str1);

        Optional<String> nullHello = Optional.ofNullable(null);
        String str2 = nullHello.orElseGet(strSupplier);
        System.out.println(str2);
    }
```

## orElseThrow ##

如果有值则将其返回，否则抛出supplier接口创建的异常。
```java
    @Test
    public void testOrElseThrow() {
        try {
            Optional<String> hello = Optional.ofNullable("Hello World");
            hello.orElseThrow(Exception::new);

            Optional<String> nullHello = Optional.ofNullable(null);
            nullHello.orElseThrow(Exception::new);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## map ##

如果有值，则对其执行调用mapping函数得到返回值。如果返回值不为null，则创建包含mapping返回值的Optional作为map方法返回值，否则返回空Optional。
```java
    @Test
    public void testMap() {
        Optional<String> hello = Optional.ofNullable("Hello World");
        Optional<String> nullHello = Optional.ofNullable(null);
        Optional<Integer> integer = hello.map(value -> 1);
        Optional<Integer> integer1 = nullHello.map(value -> 1);
        System.out.println(integer.isPresent());
        System.out.println(integer1.isPresent());
    }
```

## flatMap ##
如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。flatMap与map类似，区别在于flatMap中的mapper返回值必须是Optional。
```java
    @Test
    public void testFlatMap() {
        Optional<String> hello = Optional.ofNullable("Hello World");
        hello.flatMap(value -> Optional.of(value + " boy"));
        String str1 = hello.orElse("no value");
        System.out.println(str1);

        Optional<String> nullHello = Optional.ofNullable(null);
        nullHello.flatMap(value -> Optional.of(value + " boy"));
        String str2 = nullHello.orElse("no value");
        System.out.println(str2);
    }
```

## filter ##
filter个方法通过传入限定条件对Optional实例的值进行过滤。如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。
```java
    @Test
    public void testFilter() {
        Optional<Integer> integer = Optional.of(4);
        integer.filter(v -> v > 3).ifPresent(v -> System.out.println(v + " > 3"));
        System.out.println(integer.orElse(0));;
    }
```