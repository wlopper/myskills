[toc]

# 多核并发缓存架构和jmm模型讲解学习

## 多核并发缓存架构图

![image-20210314220345708](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210314220345708.png)

CPU与主内存打交道通过高速缓存，高速缓存可以在任务管理器中查看。

![image-20210314220604253](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210314220604253.png)

## Java线程内存模型

java线程内存模型跟cpu缓存模型类似，是基于cpu缓存模型来建立的，java线程内存模型是标准化的，屏蔽掉了底层不同计算机的区别。

流程中，会先将变量加载到工作内存中形成副本，之后修改后再次刷新到主内存。

![image-20210314221002991](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210314221002991.png)

### 证明模型示例

```java
package com.wlopper.skills.jmm;

/**
 * @author : libowan
 * @date : 2021/3/14 22:14
 * @description :测试内存模型
 */
public class VolatileVisibllityTest {
    private static boolean initFlag = false;

    public static void main(String[] args) throws InterruptedException {
        //死循环
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("waiting data...");
                while (!initFlag){
                }
                System.out.println("==================");
            }
        }).start();
        Thread.sleep(2000);
        //修改死循环条件，尝试结束死循环（结果是失败的，因为新线程修改的只是一个工作内存副本，想要修改成功，需要给initFlag加上volatile关键字）
        new Thread(new Runnable() {
            @Override
            public void run() {
                prepareData();
            }
        }).start();
    }

    private static void prepareData() {
        System.out.println("preparing data...");
        initFlag =true;
        System.out.println("preparing data end...");
    }
}
//结果：
//waiting data...
//preparing data...
//preparing data end...
//没有结束，没有像预期一样直接结束线程。因为修改的只是线程中的工作备份。
```

volatile：保证多线程之间各工作共享内存副本的可见性。

### 上例中的内存流程图

![image-20210315212214720](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210315212214720.png)

注：store的时候已经将数据放到主内存中，但是要write后才会将原来数据更新。

线程一中的use操作还是在操作工作内存中的值，没有去主内存中更新值，所以一直卡死。加了volatile后，

### MESI缓存一致性协议

多个cpu从主内存读取同一个数据到各自的高速缓存，当其中某个cpu修改了缓存中的数据，该数据会马上同步到主内存，其他cpu通过`cpu总线嗅探机制`可以感知到数据的变化，从而将自己缓存里的数据失效。`cpu总线嗅探机制`类似于java中的监听效果。

### volatile底层实现原理

底层实现主要是通过汇编lock前缀指令，它会锁定这块内存区域的缓存(缓存行锁定)并回写到主内存。

lock指令：

1. 会将当前处理器缓存行的数据立即写会到系统内存。
2. 这个写回的内存的操作会引起在其他CPU里缓存了该内存地址的数据无效(MESI协议)

#### 与synchronize的区别

* volatile不会在主内存中read时对对象加锁，而synchronize会。
* volatile在store操作时才会将主内存对象lock住，然后在将数据同步后执行unlock释放锁。
  * 为什么要对主内存lock住？
  * 因为在store时先经过总线(触发MESI协议)已经到达主内存了，但是后还有write操作，这期间可能其他内存会从主内存中读取数据，还是会造成数据不一致。

并发编程三大特性：可见性、原子性、有序性。

volatile只保证了可见性与有序性，但是不保证原子性，保证原子性需要使用synchronize这样的锁机制。

为什么不保证原子性？

比如多个线程对同一个变量循环+1，在a线程要将数据store和write时，线程b的变量a失效了，虽然b已经加过1了，但是失效后需要从主内存中读取，读取的值是a线程的1，然后进行运算时b中添加的操作就丢失了。 



## JMM数据原子操作

* read(读取)：从主内存中读取数据
* load(载入)：将主内存读取到的数据写入工作内存。
* use(使用)：从工作内存读取数据来计算
* assign(赋值)：将计算好的值重新赋值到工作内存中。
* store(存储)：将工作内存数据写入主内存。
* write(写入)：将store过去的变量值赋值给主内存中的变量。
* lock(锁定)：将主内存变量加锁，标识为线程独占状态。
* #### unlock(解锁)：将主内存变量解锁，解锁后其他线程可以锁定该变量。