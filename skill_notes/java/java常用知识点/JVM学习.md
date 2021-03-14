[toc]

# java进阶学习

## java执行流程

![image-20210313190636613](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313190636613.png)

## JVM结构

![image-20210313190727203](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313190727203.png)

### 栈

​	存放局部变量(引用)。

特点：先进后出。

#### 多线程下栈的结构

![image-20210313190950516](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313190950516.png)

#### 栈帧

​	方法的区域，存放方法的局部变量。

![image-20210313191246492](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313191246492.png)

#### 栈内存结构

![image-20210313191528043](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313191528043.png)



![image-20210313193043707](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313193043707.png)

#### jvm底层结构及步骤

局部变量表：存放方法局部变量；

操作数栈：变量运算，装载时使用；

动态链接：存储链接各个调用方法的信息(方法区入口地址)、符号引用。

方法出口：方法执行完后，返回的地址信息。

![image-20210313193615984](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313193615984.png)

```java
int a = 1;
int b = 2;
int c = (a+b)*10;
return c;
```

对应步骤：

```html
现在操作栈中放入数据1；
再在变量表中给a赋值1；
现在操作栈中放入数据2；
再在变量表中给b赋值2；
将a的值放入操作栈中；
将b的值放入操作栈中；
在1和2进行int相加；
结果*10
再在变量表中给c赋值30；
返回c的值。
```

对象引用结构：变量表中保存堆中对象的引用，保存局部变量的值。

![image-20210313195944523](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313195944523.png)

### 程序计数器

每个线程都有的一个计数器。存放线程中当前执行到的代码行号等信息。字节码引擎会去动态修改程序计数器。

为什么有程序计数器？

因为在多线程时，可能执行到一半就被其他线程抢走CPU资源，所以需要技术器进行标记当前执行到的地方，以便之后重新继续执行。

![image-20210313193844889](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313193844889.png)

![image-20210313194144354](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313194144354.png)

### 方法区

使用直接内存。存放常量+静态变量+类信息。

![image-20210313200316819](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313200316819.png)

![image-20210313200513740](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313200513740.png)

### 本地方法栈--Native方法

调用C语言中的函数实现(dll文件,类似于jar包)。

### 堆

主要调优的地方。

#### 堆结构

![image-20210313201008010](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313201008010.png)

分为年轻代和老年代。占比1:2；

年轻代中分为Eden，s0，s1三个，占比8:1:1;

Eden区满了，会做minor gc。

gc后对象没有被删掉，就会给这写对象生态标记加一。

老年代中常见对象：

1. 静态对象
2. 容器中的bean对象

进入老年代的条件：

1. 生态数达到15。
2. 对象动态年龄判断。
   1. 一批对象的总大小大于这块Survivor区域内存大小的50%，会直接放到老年代。
3. 老年代空间分配担保机制。
4. minor gc后存活的对象Survivor区放不下。

```java
new 出来的对象放在Eden区中。
    
```

## GC流程

1. 栈中找到所有局部变量(gc root)。
2. 在方法中找到所有局部变量(gc root)。
3. 根据引用找对应的对象。
4. 再找引用对象是否有引用其他对象。直到最后一个。（可达性算法）
5. 标记这些对象。
6. 把这些对象在堆中从Eden区复制到s0区中。
7. 清除Eden中的对象。（非常快，因为是直接清除）
8. gc后对象没有被删掉，就会给这写对象生态标记加一
9. 同理如果Eden区中再次满了，会进行s0、Eden的gc回收，回收后没有被清理就会进入s1，清空s0;生态值加一。
10. Eden再次满了时，对Eden、s1进行gc回收，存活放入s0,清空s2。生态值加一。
11. 一个对象的生态值达到15，就会移入老年代。
12. ![image-20210313202922829](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313202922829.png)
13. 当老年代满了，做full gc；
14. full gc 会对Eden、s0、s1、老年代全做gc。
15. 最终还是满了，就报错 OutOfMemory异常。

## 对象信息结构

分三块：

对象头：包含分代信息。类似于http协议一样。

对象信息：对象类、方法等信息。

![image-20210313202321700](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313202321700.png)

## jdk自带工具

javap：反汇编,只反汇编方法名类名等信息。

​	-c：形成步骤。

jvisualvm：查看jvm内存分配。

​	-安装visual GC插件后可以直观看到jvm的分布。

## JVM调优

最终目的：减少gc；特别是full gc；

原因：gc时会STW(stop the world)，专心gc收集，所以会比较慢。

![image-20210313205121444](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210313205121444.png)

方法：尽量让缓存在年轻代就清除，不要等到老年代再进行清除。

## 一线大厂工资情况

![image-20210314204849721](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210314204849721.png)