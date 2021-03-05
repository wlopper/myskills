[toc]

# kettle学习笔记

## kettle是什么？

​	kettle是一个ETL工具。可以完成不同系统间数据库表的迁移操作，对于不同的字段可以通过映射进行一一对应，完成数据库表的无缝迁移。

​	ETL是EXTRACT（抽取）、TRANSFORM（转换）、LOAD（加载）的简称。

​	kettle有两种脚本transformation（转换）和Job（作业）。

​	Kettle有三个主要组件：Spoon、Kitchen、Pan

​     	Spoon：是一个图形化的界面，可以让我们用图形化的方式开发转换和作业。windows选择Spoon.bat；Linux选择Spoon.sh

​    	 Kitchen：利用Kitchen可以使用命令行调用Job

​    	 Pan：利用Pan可以用命令行的形式调用Trans

​     	Carte：Carte是一个轻量级的Web容器，用于建立专用、远程的ETL Server。

## 连接自定义数据库驱动

参考：[链接](https://www.cnblogs.com/dqrcsc/p/12161283.html)

### 1.kettle连接DM7的相关操作

#### 1.1 DM7驱动安装

kettle连接DM7，本质上是通过JDBC连接，因而需要将DM7的JDBC驱动放到${kettle_home}\lib\目录中。DM7的JDBC驱动可以到DM7的安装目录中${dm_home}\drivers\jdbc\中，将以下jar文件复制到kettle的lib目录：

```
DM7JdbcDriver15.jar
DM7JdbcDriver16.jar
DM7JdbcDriver17.jar
```

#### 1.2 配置kettle数据库连接

- 1.在kettle中新建一个数据库连接，连接类型选择：Generic database
- 2.自定义连接URL，填写：jdbc:dm://${ip}:5236/DMSERVER
- 3.自定义驱动类名称：dm.jdbc.driver.DmDriver
- 4.用户名：SYSDBA（自定义）
- 5.密码：对应用户名的密码
- 6.连接方式：保留默认的Native(JDBC)即可。

#### 1.3 测试连接

点击测试，查看是否连接成功。

### 2.可能遇到的问题

#### 2.1 测试连接失败，找不到dm.jdbc.driver.DmDriver类

可能是在安装DM驱动时，没有重启kettle。因为kettle会在启动过程中加载lib中的所有依赖库，所以，如果是在kettle已经启动的情况下复制驱动类，并不会立即加载的。此时，只需要重启下kettle，让其重新加载DM驱动即可。

#### 2.2 表输入中提示找不到表、或者无法解析字段的错误

类似于MySQL中的数据库，DM7数据库中的每张表是归属于某个schema（模式）的。所以在表输入/表输出组件中，一定要通过schema去选定要操作的表。而不要直接从表的子树中选定要操作的表，不然DM无法找到要操作的表。当然，在表输入中，直接手动编写SQL也是可以的，在from子句中指定${schema}.${table_name}，明确指定表的所属schema即可。

## kettle使用

### 1、启动kettle

项目结构如下：

![image-20210305163406989](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305163406989.png)

找到Spoon.bat后双击即可打开，打开后图形界面如下：

![image-20210305163638329](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305163638329.png)

### 2、创建数据库连接

#### 创建

想要转移数据，就必须先创建一个数据库连接。通过在左上`文件`下面的`+`号创建一个数据库连接，具体的链接类型和连接方式视具体情况而定，以DM数据库为例。

![image-20210305164124186](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305164124186.png)

填写信息完成后点击测试按钮，会提示：

![image-20210305164215361](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305164215361.png)

#### 共享

想要在不同转换中同样使用这个数据库，可以在`DB连接`中找到对应的数据库连接，右键-->共享即可。共享比不共享字体更粗。

![image-20210305165020593](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305165020593.png)

### 3、创建转换

通过在左上`文件`下面的`+`号创建一个转换，左侧栏目会自动转成`核心对象`栏。然后在这里就可以进行一系列迁移、转换等操作了。

#### 输入、输出

​	迁移不一定是像数据库迁移，将数据导成excel也可以。所以一个迁移最基础的是输入输出，里面的数据通过流的形式进行迁移。

​	最简单的案例：

![image-20210305165657776](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305165657776.png)

![image-20210305165712361](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305165712361.png)

![image-20210305165734556](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305165734556.png)

#### 查询

在下例中，通过站点查出栏目ids，然后通过栏目ids查询对应栏目下的文章信息，最后输出到目的数据库的文章表中。可以看出，可以通过查询操作进行多次的筛选数据，转换数据，最终形成需要的数据集合后输出即可。

![image-20210305170437207](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210305170437207.png)

### 4、创建作业

​	作业主要是可以创建一个任务，然后设置时间定时执行或者定时轮番执行。一般用来更新数据库的一些内容而不是完全迁移，因为工作中没有使用到这个，所以不是很了解:happy:

