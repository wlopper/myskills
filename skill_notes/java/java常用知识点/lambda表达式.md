[toc]

# java8 lambda表达式学习

## 简单案例

1. 首先准备两个类，实现学生按成绩从高到底排序

```java
package com.wlopper.skills.lambda;

import org.junit.Test;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

/**
 * @author : libowan
 * @date : 2021/2/28 11:00
 * @description :学生类
 */
public class Student {
    private String name;
    private Double score;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }

    public Student() {
    }

    public Student(String name, Double score) {
        this.name = name;
        this.score = score;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getScore() {
        return score;
    }

    public void setScore(Double score) {
        this.score = score;
    }
}
```

```java
package com.wlopper.skills.lambda;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

/**
 * @author : libowan
 * @date : 2021/2/28 11:13
 * @description :测试
 */
public class Test {
    //测试
    @org.junit.Test
    public void test(){
        //准备数据
        List<Student> studentList = new ArrayList<Student>();
        studentList.add(new Student("stu1",100.0));
        studentList.add(new Student("stu2",97.0));
        studentList.add(new Student("stu3",96.0));
        studentList.add(new Student("stu4",95.0));
        studentList.add(new Student("stu5",98.0));
        System.out.println(studentList);
        //数据排序，根据分数从高到底排序
        Collections.sort(studentList, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return Double.compare(o2.getScore(), o1.getScore());
            }
        });
        System.out.println(studentList);
    }
}

```

代码中用到了匿名内部类，使用sort方法排序。不过整体为了实现排序操作传递了整个Comparable类，显得比较笨拙，有大量的代码冗余。lambda表达式可以解决这种问题！使用lambda后：

```java
@org.junit.Test
public void test(){
    //准备数据
    List<Student> studentList = new ArrayList<Student>();
    studentList.add(new Student("stu1",100.0));
    studentList.add(new Student("stu2",97.0));
    studentList.add(new Student("stu3",96.0));
    studentList.add(new Student("stu4",95.0));
    studentList.add(new Student("stu5",98.0));
    System.out.println(studentList);
    //数据排序，根据分数从高到底排序
    Collections.sort(studentList, (o1, o2) -> Double.compare(o2.getScore(), o1.getScore()));
    System.out.println(studentList);
}
```

## lambda语法

lambda表达式主要是面向函数式接口（只包含一个抽象方法的接口），可以简化在使用函数式接口时代码冗余杂乱的现象。

### 多参数（最常见是Comparator接口比较）

1. lambda的基本格式为`(o1,o2) ->{表达式...};`；
2. 如果表达式中带有两个参数的话，参数类型可以省略，但两边的大括号不可以省略；
3. 如果表达式只有一行，那么大括号也可以省略；

### 无参数（最常见是Runnable接口创建线程）

1. 参数的括号不能省略；
2. 其他语法同多参数；

```java
//原写法
@org.junit.Test
public void test1(){
    System.out.println("主线程：" + Thread.currentThread().getName());
    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("开启的新线程：" + Thread.currentThread().getName());
        }
    });
    thread.start();
}
//lambda写法
@org.junit.Test
public void test1(){
    System.out.println("主线程：" + Thread.currentThread().getName());
    Thread thread = new Thread(() -> System.out.println("开启的新线程：" + Thread.currentThread().getName()));
    thread.start();
}
```

### 单参数

1. 可以省略参数的括号和类型
2. 其他语法同多参数

### jdk常用函数式接口

1. Function：提供任意一种类型的参数，返回另外一个任意类型返回值。 R apply(T t);

2. Consumer：提供任意一种类型的参数，返回空值。 void accept(T t);

3. Supplier：参数为空，得到任意一种类型的返回值。T get();

4. Predicate：提供任意一种类型的参数，返回boolean返回值。boolean test(T t);

## 方法引用

lambda表达式用于替换函数式接口，方法引用也是如此，方法引用可以使代码更加简单和便捷。

```java
public static void test1_() {
    List<String> strLst = new ArrayList<String>() {
        {
            add("adfkjsdkfjdskjfkds");
            add("asdfasdfafgfgf");
            add("public static void main");
        }
    };
    Collections.sort(strLst, String::compareToIgnoreCase);
    System.out.println(strLst);
}
```

### 使用方式

1. 方法引用主要有如下三种使用情况

   （1）. 类：：实例方法

   （2）. 类：：静态方法

   （3）. 对象：：实例方法

## 构造器引用（使用较少）

```java
public class LambdaTest3 {
 
    @Test
    public void test1_(){
        List<Integer> list = this.asList(ArrayList::new ,1,2,3,4,5);
        list.forEach(System.out::println);
    }
 
    public  <T> List<T> asList(MyCrator<List<T>> creator,T... a){
        List<T> list =  creator.create();
        for (T t : a)
            list.add(t);
        return list;
    }
}
interface MyCrator<T extends List<?>>{
    T create();
}
```



