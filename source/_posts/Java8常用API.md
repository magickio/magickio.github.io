---
title: Java8常用API
date: 2021-06-18 11:27:02
tags:
---

# 📘 Lambda表达式

## 1. Lambda 表达式演示

###### *一般的匿名类:*



```java
//Runnable匿名内部类
Runnable runnable = new Runnable(){
    @Override
    public void run() {
        System.out.println("bbb");    
    }  
};
new Thread(runnable).start();
```



*lambda表达式：*



```java
/*
 *第一种：声明一个父类接口，初始化该接口的匿名实现类
 */
Runnable r1=()->{
    System.out.println(Thread.currentThread().getName());
};
//线程任务对象包装成线程对象。
Thread t=new Thread(r1);
//启动线程。
t.start();

/*
 *第二种：无需声明，直接使用匿名内部类
 */
//线程任务对象包装成线程对象。
Thread t1=new Thread(()->{
    System.out.println(Thread.currentThread().getName());
  });
//启动线程。
t1.start();

/*
 *两种方式的区别在于你是否需要重复使用该接口实现类
 */
```





## 2. 理解Lambda步骤：



### 2.1：**函数式接口**



**lambda的前置条件：**必须是函数式接口，才可以用lamnda表达式。

### 2.2：**语法格式**



#### 基础语法



```
(参数列表) -> {
    Lambda体
}
```



#### “参数列表”详细语法



```java
/*
 *无参，小括号()必写
 */
Thread thread = new Thread(()->{
    System.out.println("hello world");
});

//----------------------------------------------------------------------------------------

/*
 *有一个参数
 */
Consumer<String> con = (x) -> {
    System.out.println(x);
};
//一个参数时，参数列表的括号“()”可以省略不写，直接写参数列表
Consumer<String> con = x -> {
    System.out.println(x);
};

//----------------------------------------------------------------------------------------

/*
 *有多个参数
 */
new TreeSet<Integer>((x,y) ->  {
    return Integer.compare(x, y);
});
```



#### “Lambda体”详细语法



```java
/*
 *Lambda体只有一行，无返回值
 */
Thread thread = new Thread(()->{
    System.out.println("hello world");
});
//Lambda体只有一行时，也可以省略“{}”符号
Thread thread = new Thread(()->System.out.println("hello world"));

//----------------------------------------------------------------------------------------

/*
 *Lambda体有多行，无返回值
 */
Thread thread = new Thread(()->{
    System.out.println("hello");
    System.out.println("world");
});

//----------------------------------------------------------------------------------------

/*
 *Lambda体只有一行，而且有返回值
 */
new TreeSet<Integer>((x,y) ->  {
    return Integer.compare(x, y);
});
//这种情况下，可以同时省略“return”关键字 和 “{}”符号
new TreeSet<Integer>((x,y) ->  Integer.compare(x, y));

//----------------------------------------------------------------------------------------

/*
 *Lambda体有多行，而且有返回值
 */
new TreeSet<Integer>((x,y) ->  {
    System.out.println("hello world");
    return Integer.compare(x, y);
});
```



## 3. 拓展：Lambda表达式中的类型推断

Lambda 表达式的参数列表实质上是带有参数类型的，如下

```java
new TreeSet<Integer>((Integer x,Integer y) ->  {
    return Integer.compare(x, y);
});
```

###### 但是因为JVM编译器通过上下文推断出数据类型（即“类型推断”），所以我们可以省略参数类型不写。实际上我们在书写Lambda表达式时都是不写参数类型的。如下

```java
new TreeSet<Integer>((x,y) ->  {
    return Integer.compare(x, y);
});
```



## 4. Lambda表达式特征总结

1. 不用声明参数类型，编译器可统一识别参数值。
2. 一个参数不用定义圆括号, 多个参数则需要圆括号。
3. 方法体里只包含一个语句，不需要使用大括号。
4. 如果方法体只有一个表达式返回值则编译器会自动返回值，而大括号需指定表达式返回了一个数值。

# 📘 函数式接口

## **一、函数式接口特征：**



接口中有且仅有一个抽象方法。（但是可以有多个非抽象方法，如静态方法和默认方法）



## 二、 四大内置核心函数式接口

|                                    | 接口          | 抽象方法          |
| ---------------------------------- | ------------- | ----------------- |
| 消费型接口（有一个参数且无返回值） | Consumer<T>   | void accept(T t)  |
| 供给型接口（无参且有返回值）       | Supplier<T>   | T get( )          |
| 函数型接口（有一个参数且有返回值） | Function<T,R> | R apply(T t)      |
| 断言型接口（有参且返回布尔值）     | Predicate<T>  | boolean test(T t) |



*注：这四个函数式接口基本囊括了业务中90%的技术需求。但如果遇到这四个接口无法解决的技术需求，实际上在 java.lang.funtion 包中还有非常多的函数式接口供使用。这些接口大部分继承了四大核心函数式接口或继承了四大核心函数式接口的思想。以下举了一些例子，更多可以自己了解：*

|                                                     | 接口              | 抽象方法               |
| --------------------------------------------------- | ----------------- | ---------------------- |
| 与消费型接口Consumer类似（有两个参数且无返回值）    | BiConsumer<T,U>   | void accept(T t, U u)  |
| 与函数型接口Function类似（有两个参数且有返回值）    | BiFunction<T,U,R> | R apply(T t, U u)      |
| 与断言型接口Predicate类似（有两个参数且返回布尔值） | BiPredicate<T, U> | boolean test(T t, U u) |

## 拓展、@**FunctionalInterface注解**

###### @FunctionalInterface 注解用于表示一个接口为函数式接口。

同时，我们也可以使用@FunctionalInterface注解来检查某个接口是否为函数式接口：如果使用了@FunctionalInterface注解，接口中的抽象方法只允许有一个。

# 📘 Stream API

## 1、概述

​	Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用Stream API 对集合数据进行操作，**就类似于使用 SQL 执行的数据库查询**。也可以使用 Stream API 来并行执行操作。简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式。



## 2、实例



定义一个员工类，用于实例演示



```java
public class Employees {

    //姓名
    private String name;

    //年龄
    private int age;

    //工资
    private int salary;

    //地址
    private String address;

    //构造器
    public Employees(String name, int age, int salary, String address) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.address = address;
    }

    //提供get、set方法；重写toString方法；重写equals方法；重写hashCode方法；
    //以上方法为了省略篇幅，所以没有贴出
```



定义一个Stream测试类，用于初始化数据



```java
public class StreamTest {

    public static void main(String[] args) {
        Employees employees1 = new Employees("小明",11,2000,"东莞");
        Employees employees2 = new Employees("小黑",12,3000,"深圳");
        Employees employees3 = new Employees("小黄",13,4000,"汕头");
        Employees employees4 = new Employees("小郭",14,5000,"北京");
        Employees employees5 = new Employees("小郭",14,5000,"北京");
        Employees employees6 = new Employees("小肖",14,6000,"黑龙江");
        Employees employees7 = new Employees("小明",11,2000,"东莞");
        
        List<Employees> list = new ArrayList<>();
        list.add(employees1);
        list.add(employees2);
        list.add(employees3);
        list.add(employees4);
        list.add(employees4);
        list.add(employees6);
        list.add(employees7);
    }
}
```

## 3、操作分类



![image](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233536.png)



- 无状态：指元素的处理不受之前元素的影响；
- 有状态：指该操作只有拿到所有元素之后才能继续下去。
- 非短路操作：指必须处理所有元素才能得到最终结果；
- 短路操作：指遇到某些符合条件的元素就可以得到最终结果，如 A || B，只要A为true，则无需判断B的结果。

### 3.2 中间操作

## 📘 Stream——中间操作

###### 沿用 [Stream API](https://www.yuque.com/eae60m/iwgwlq/cagtia) 中的测试类和数据。

定义一个员工类，用于实例演示



```java
public class Employees {

    //姓名
    private String name;

    //年龄
    private int age;

    //工资
    private int salary;

    //地址
    private String address;

    //构造器
    public Employees(String name, int age, int salary, String address) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.address = address;
    }

    //提供get、set方法；重写toString方法；重写equals方法；重写hashCode方法；
    //以上方法为了省略篇幅，所以没有贴出
```



定义一个Stream测试类，用于初始化数据



```java
public class StreamTest {

    public static void main(String[] args) {
        Employees employees1 = new Employees("小明",11,2000,"东莞");
        Employees employees2 = new Employees("小黑",12,3000,"深圳");
        Employees employees3 = new Employees("小黄",13,4000,"汕头");
        Employees employees4 = new Employees("小郭",14,5000,"北京");
        Employees employees5 = new Employees("小郭",14,5000,"北京");
        Employees employees6 = new Employees("小肖",14,6000,"黑龙江");
        Employees employees7 = new Employees("小明",11,2000,"东莞");
        
        List<Employees> list = new ArrayList<>();
        list.add(employees1);
        list.add(employees2);
        list.add(employees3);
        list.add(employees4);
        list.add(employees4);
        list.add(employees6);
        list.add(employees7);
    }
}
```

### 1. 筛选与切片

### 1.1 过滤 filter

通过一个断言，排除某些元素

```java
public static void main(String[] args) {
    ...
    //过滤出年龄大于13的Employees
    list.stream()
        .filter(emp -> emp.getAge() > 13)
        .forEach(System.out::println);
}

========================输出结果============================
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
```

###  

### 1.2 去重 distinct

distinct 是同时依赖于 equals 方法和 hashCode 方法的，其中一个方法没有正确重写都不会去重成功

```java
public static void main(String[] args) {
    ...
    //去重
    list.stream()
        .distinct()
        .forEach(System.out::println);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小黑, age=12, salary=3000, address=深圳]
Employees [name=小黄, age=13, salary=4000, address=汕头]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
```

###  

### 1.3 截断 limit

使元素不超过指定数量

```java
public static void main(String[] args) {
    ...
    //截断
    list.stream()
        .limit(3)
        .forEach(System.out::println);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小黑, age=12, salary=3000, address=深圳]
Employees [name=小黄, age=13, salary=4000, address=汕头]
```

###  

### 1.4 跳过 skip

去除前n个元素。如果流中元素不足n个，则返回一个空流

```java
public static void main(String[] args) {
    ...
    //跳过
    list.stream()
        .skip(3)
        .forEach(System.out::println);
}

========================输出结果============================
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
Employees [name=小明, age=11, salary=2000, address=东莞]
```



### 2. 映射

### 2.1 map

对流中的每一个元素都进行同一个指定操作，操作结束后最终返回一个新的元素

```java
public static void main(String[] args) {
    ...
    //映射
    list.stream()
        .map(emp -> emp.getAddress())
        .forEach(System.out::println);
}

========================输出结果============================
东莞
深圳
汕头
北京
北京
黑龙江
东莞
```



### 2.2 mapToDouble

与 map 类似，但是返回类型必须是 Double 型。

### 2.3 mapToInt

与 map 类似，但是返回类型必须是 Int 型。

### 2.4 mapToLong

与 map 类似，但是返回类型必须是 Long 型。

### 2.5 flatMap

将流中的每个值都换成另一个流，然后把所有流连接成一个流。

这个不太好演示且使用率不太高，可自行搜索



### 3. 排序

### 3.1 sorted

将流中元素按指定顺序（自然排序或者定制排序）排序

如果是自然排序，则必须实现Comparable接口，如果没有实现会抛出异常

```java
public static void main(String[] args) {
    ...
    //自然排序（演示自然排序时为Employees类实现了Comparable接口，实现为按照年龄升序排序）
    list.stream()
        .sorted()
        .forEach(System.out::println);
    
    //----------------------------------------------------------------------
    
    //定制排序，使用Comparator匿名类，实现为按照工资升序排序
    list.stream()
        .sorted((emp1,emp2) -> {
            int salary1 = emp1.getSalary();
            int salary2 = emp2.getSalary();
            return Integer.compare(salary1, salary2);
        })
        .forEach(System.out::println);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小黑, age=12, salary=3000, address=深圳]
Employees [name=小黄, age=13, salary=4000, address=汕头]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]    
//----------------------------------------------------------------------------
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小黑, age=12, salary=3000, address=深圳]
Employees [name=小黄, age=13, salary=4000, address=汕头]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小郭, age=14, salary=5000, address=北京]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
```

### 3.3 终止操作

## 📘 Stream——终止操作

###### 📘 沿用 [Stream API](https://www.yuque.com/eae60m/iwgwlq/cagtia) 中的测试类和数据。

定义一个员工类，用于实例演示



```java
public class Employees {

    //姓名
    private String name;

    //年龄
    private int age;

    //工资
    private int salary;

    //地址
    private String address;

    //构造器
    public Employees(String name, int age, int salary, String address) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.address = address;
    }

    //提供get、set方法；重写toString方法；重写equals方法；重写hashCode方法；
    //以上方法为了省略篇幅，所以没有贴出
}
```



定义一个Stream测试类，用于初始化数据



```java
public class StreamTest {

    public static void main(String[] args) {
        Employees employees1 = new Employees("小明",11,2000,"东莞");
        Employees employees2 = new Employees("小黑",12,3000,"深圳");
        Employees employees3 = new Employees("小黄",13,4000,"汕头");
        Employees employees4 = new Employees("小郭",14,5000,"北京");
        Employees employees5 = new Employees("小郭",14,5000,"北京");
        Employees employees6 = new Employees("小肖",14,6000,"黑龙江");
        Employees employees7 = new Employees("小明",11,2000,"东莞");
        
        List<Employees> list = new ArrayList<>();
        list.add(employees1);
        list.add(employees2);
        list.add(employees3);
        list.add(employees4);
        list.add(employees4);
        list.add(employees6);
        list.add(employees7);
    }
}
```



### 1. 查找与匹配

### 1.1 是否所有元素匹配指定条件 allMatch

```java
public static void main(String[] args) {
    ...
    //判断是否所有元素的地址都是北京
    boolean allMatch = list.stream()
        .allMatch(emp -> {
            return "北京".equals(emp.getAddress());
        });
    System.out.println(allMatch);
}

========================输出结果============================
false
```

###  

### 1.2 是否至少一个元素匹配指定条件 anyMatch

与 allMatch 类似，可参考 allMatch

### 1.3 是否所有元素都不匹配指定条件 noneMatch

与 allMatch 类似，可参考 allMatch

### 1.4 获取第一个元素 findFirst

```java
public static void main(String[] args) {
    ...
    Optional<Employees> findFirst = list.stream().findFirst();
    Employees employees = findFirst.get();
    System.out.println(employees);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
```



### 1.5 获取一个随机元素 findAny

###### findAny方法在并行流跟串行流上是有区别的。串行流是单线程，先找到哪个元素就会返回哪个元素，所以一般都是返回第一个元素；而并行流是多个线程的，哪个线程先找到元素就返回该线程的元素。

```java
public static void main(String[] args) {
    ...
    Optional<Employees> findFirst = list.parallelStream().findAny();
    Employees employees = findFirst.get();
    System.out.println(employees);
}

========================输出结果============================
Employees [name=小郭, age=14, salary=5000, address=北京]
```



### 1.6 获取总个数 count

```java
public static void main(String[] args) {
    ...
    long count = list.stream().count();
    System.out.println(count);
}

========================输出结果============================
7
```



### 1.7 获取最大值 max

需要指定一个比较器Comparator，程序使用该比较器的规则来获取最大值

```java
public static void main(String[] args) {
    ...
    //查找工资最大的Employees
    Optional<Employees> max = list.stream().max((emp1,emp2) -> {
        int salary1 = emp1.getSalary();
        int salary2 = emp2.getSalary();
        return Integer.compare(salary1, salary2);
    });
    Employees employees = max.get();
    System.out.println(employees);
}

========================输出结果============================
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
```



### 1.8 获取最小值 min

需要指定一个比较器Comparator，程序使用该比较器的规则来获取最小值

```java
public static void main(String[] args) {
    ...
    //查找年龄最小的Employees
    Optional<Employees> min = list.stream().min((emp1,emp2) -> {
        int age1 = emp1.getAge();
        int age2 = emp2.getAge();
        return Integer.compare(age1, age2);
    });
    Employees employees = min.get();
    System.out.println(employees);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
```





### 2. 归约

### 2.1 reduce

将流中所有元素或者所有元素的某个属性值反复组合起来，最终得到一个值

在执行第一遍操作时，是将指第一个元素作为 x ，第二个元素作为 y 相组合。在执行第一遍以后的操作时，是将第一遍操作的结果作为 x ，上一次操作后面的下一个元素作为 y 相组合。直到所有元素组合完成

```java
public static void main(String[] args) {
    ...
    //求所有工资总和（先用map取出所有工资，再用reduce相加）
    Optional<Integer> reduce = list.stream()
        .map(emp -> emp.getSalary())
        .reduce((sal1,sal2) -> {
            return sal1 + sal2;
        });
    Integer total = reduce.get();
    System.out.println(total);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
```

也可以指定初始值。那么在执行第一遍操作时是将指定的初始值作为 x 与流中的第一个元素作为 y 相组合。

这样子也能更好地避免空指针

```java
public static void main(String[] args) {
    ...
    //求所有工资总和（先用map取出所有工资，再用reduce相加(同时指定初始值为0)）
    Integer total = list.stream()
            .map(emp -> emp.getSalary())
            .reduce(0, (sal1,sal2) -> {
                return sal1 + sal2;
        });
    System.out.println(total);
}

========================输出结果============================
Employees [name=小明, age=11, salary=2000, address=东莞]
```



### 3. 收集

### 3.1 collect

将流转换为其他形式。接收一个 Collector 接口的实现，用于给 Stream 中元素做汇总的方法。

而且一般情况下我们不需要真的去实现 Collector 接口，已经有一个工具类 Collectors 提供了大量的 API

Collectors 工具类可自己去做了解，里面的 API 非常多，能完成非常多的需求，功能强大。

```java
public static void main(String[] args) {
    ...
    //汇总为set
    Set<Employees> set = list.stream().collect(Collectors.toSet());
    set.forEach(System.out::println);
}

========================输出结果============================
Employees [name=小黑, age=12, salary=3000, address=深圳]
Employees [name=小明, age=11, salary=2000, address=东莞]
Employees [name=小肖, age=14, salary=6000, address=黑龙江]
Employees [name=小黄, age=13, salary=4000, address=汕头]
Employees [name=小郭, age=14, salary=5000, address=北京]
```





### 4. 内部迭代

### 4.1 forEach

太简单了不写了，就和for循环差不多的

# 📘 Optional

java . util. Optional 是在 Java 8 中引入，用于解决空指针异常的一个工具。

它更像是一个容器类，用于代表一个值存在或者不存在。在 JDK 8 以前，用 null 值来表示一个值不存在。现在使用 Optional 可以更好的表达这个概念。

## 创建 Optional 实例

### Optional.of(T value) 

用于创建一个 Optioal 的类的实例



```java
/**
 * 说明：创建一个 Option 类的实例
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:17:29
 */
@Test
private void test01() {
    //创建
    Optional<String> op = Optional.of("123");
}
```



但是需要注意的是，**Optional 类虽然是 Java 8 中用于避免空指针的工具类，但是它的 of 方法不能接收一个 null**；所以在实际使用中，使用 of 方法构造 Optional 类的实例的时候，需要避免从其他地方传来的用于构造 Optiaonal 实例的值是 null；



```java
/**
 * 说明：使用 null 构造 Optional 类的实例
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:23:24
 */
@Test
public void test02() {
    //创建
    Optional<String> op = Optional.of(null);
    //会产生 java.lang.NullPointerException 异常。
}java
```



但是这仍然极大的提高了 debug 空指针异常的错误。在 JDK 8 以前发生空指针的时候不知道是哪一步发生了空指针；现在如果使用 Optional 类，那么极大可能是在 of 方法中发生空指针。

### Optional.empty()

创建一个空的 Optional 的类的实例，用于补充 of 方法的参数值为 null 的情况。



```java
/**
 * 说明：创建一个空的 Option 类的实例
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:39:21
 */
@Test
public void test03() {
    //创建,此时 op 中是没有值的
    Optional<String> op = Optional.empty();
}
```

### Optional.ofNullable(T value)

of 方法和 empty 方法的结合体，更优雅，代码阅读性也更好，实际使用推荐此方法。

若 value 的值不为 null，则调用 of 方法返回一个值为 value 的 Optional 的类的实例；否则调用 empty 方法返回一个空的 Optional 的类的实例。



```java
/**
 * 说明：创建一个 Option 类的实例
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:39:21
 */
@Test
public void test04() {
    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);
    /*
     * 底层代码就是使用了一个三元表达式，去判断使用 of 方法还是 empty 方法，如下所示
     * public static <T> Optional<T> ofNullable(T value) {
     *      return value == null ? empty() : of(value);
     * }
     */
}
```

## 判断 Optional 实例是否有值/值是否为空

### Optional.isPresent()

用于判断 Optional 实例中包含的值是否null；



```java
/**
 * 说明：判断 Optional 实例是否有值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:55:17
 */
@Test
public void test05() {
    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);

    //判断
    boolean pre_01 = op_01.isPresent();
    boolean pre_02 = op_02.isPresent();

    //输出
    System.out.println(pre_01); //输出 true
    System.out.println(pre_02); //输出 false
}
```

## 处理 Option 实例中的值

### Optional.map(Function mapper)

对 Optional 实例中的值执行指定的操作，操作结束后返回一个包含着新值的新的 Optional 实例。

如果原来的 Optional 实例的值为空，则不进行操作，仍然返回一个 empty 的 Option 实例。

```java
/**
 * 说明：处理 Option 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午3:34:20
 */
@Test
public void test10() {
    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);
    System.out.println(op_01.hashCode());   //1450575459
    System.out.println(op_02.hashCode());   //0

    //处理
    Optional<String> op_03 = op_01.map(str -> str += "abcdefg");
    Optional<String> op_04 = op_02.map(str -> str += "abcdefg");
    System.out.println(op_03.hashCode());   //-1006303071
    System.out.println(op_04.hashCode());   //0

    //获取
    String str_01 = op_03.orElse("xyz");
    String str_02 = op_04.orElse("xyz");

    //输出
    System.out.println(str_01); //123456abcdefg
    System.out.println(str_02); //xyz
}
```

### Optional.flatMap(Function mapper)

与 map 方法类似。

但是 map 方法中，Function 实现类的返回值不会直接被返回，而是会被再包装一层 Optional 类，即“Function 实现类的返回值” ->“Optional<Function 实现类的返回值>”；

而在 flatMap 中返回值保持不变，即直接返回 Function 实现类的返回值，但要求 Function 实现类的返回值必须是 Optional 类型，即“Optional<返回值>” -> “Optional<返回值>”。

这么做的好处是为了避免 map 方法中返回的是一个 Option 类的实例，从而导致调用完 map 方法后得到的返回值是一个 Optional<Optional<真正的值>> 这种嵌套结构的结果。 

```java
/**
 * 说明：处理 Option 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午3:34:20
 */
@Test
public void test11() {
    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);

    //处理
    Optional<Optional<String>> op_03 = op_01.map(str -> Optional.ofNullable(str));
    Optional<String> op_04 = op_02.flatMap(str -> Optional.ofNullable(str));    //与 map方法做比较，能看到map方法返回值是一个Optional<Optional<真正的值>>的嵌套结构的结果

    //获取
    String str_01 = op_03.orElse(Optional.ofNullable("xyz")).orElse("xyz");     //可以看出在这种嵌套结构的结果上，取值更为麻烦
    String str_02 = op_04.orElse("xyz");

    //输出
    System.out.println(str_01); //123456
    System.out.println(str_02); //xyz
}
```

## 获取 Optional 实例中的值

### Optional.get()

直接获取 Optional 实例中的值



```java
/**
 * 说明：直接获取 Optional 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:55:17
 */
@Test
public void test06() {
    //创建
    Optional<String> op = Optional.ofNullable("123456");

    //获取
    String str = op.get();
    System.out.println(str);    //输出 123456
}
```



**但是需要注意的是，如果 Optional 实例中的值为空，get 方法会抛出 java.util.NoSuchElementException 异常。**

```java
/**
 * 说明：直接获取 Optional 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:55:17
 */
@Test
public void test07() {
    //创建
    Optional<String> op = Optional.ofNullable(null);

    //获取
    String str = op.get();
    System.out.println(str);    //抛出异常 java.util.NoSuchElementException
}
```

### Optional.orElse(T other)

get 方法的升级方法，用于避免 get 方法获取 Optional 实例的值时，值为空时抛出异常的情况。

如果 Optional 实例有值时，则返回 Optional 实例的值；否则返回传入的参数值。

```java
/**
 * 说明：获取 Optional 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:55:17
 */
@Test
public void test08() {
    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);

    //获取
    String str_01 = op_01.orElse("abcdefg");
    String str_02 = op_02.orElse("abcdefg");

    //输出
    System.out.println(str_01); //123456
    System.out.println(str_02); //abcdefg
}
```

### Optional.orElseGet(Supplier other)

orElse 方法的升级方法，比起 orElse 方法的固定值，可以更动态的决定当 Optional 实例的值为空时，返回一个什么样的值。

```java
/**
 * 说明：获取 Optional 实例中的值
 * @author DZ_WenYaoHao
 * 2021年3月23日   下午2:55:17
 */
@Test
public void test09() {
    //创建一个随机数
    int i = new Random().nextInt();

    //创建
    Optional<String> op_01 = Optional.ofNullable("123456");
    Optional<String> op_02 = Optional.ofNullable(null);

    //获取
    String str_01 = op_01.orElseGet(() -> i%2 == 0?"abc":"xyz");
    String str_02 = op_02.orElseGet(() -> i%2 == 0?"abc":"xyz");

    //输出
    System.out.println(str_01); //123456
    System.out.println(str_02); //abc 或者 xyz，取决于 i 是奇数还是偶数。
}
```