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

## JMM数据原子操作

* read(读取)：从主内存中读取数据
* load(载入)：将主内存读取到的数据写入工作内存。
* use(使用)：从工作内存读取数据来计算
* assign(赋值)：将计算好的值重新赋值到工作内存中。
* store(存储)：将工作内存数据写入主内存。
* write(写入)：将store过去的变量值赋值给主内存中的变量。
* lock(锁定)：将主内存变量加锁，标识为线程独占状态。
* unlock(解锁)：将主内存变量解锁，解锁后其他线程可以锁定该变量。