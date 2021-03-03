[toc]

## 记一次在linux上利用docker部署项目

​	20201028，公司急着快速项目演示，所以立马开始了项目部署，这次部署不仅是我小组的项目，其他小组项目也要统一部署，于是一场轰轰烈烈的项目部署就开始了。我要部署的项目的整体结构最为复杂，遇到的坑也就最多，可谓是一步一坑！！！

​	先说总体流程，公司打算迁移最近部署的191项目到新的220环境。191之前是使用tomcat部署的，这次要求使用docker部署到k8s上。之前没用过docker和k8s的我只能按照要求一步步来了。然后项目部署在203服务器上，统一用180服务器的nginx进行域名代理。缓存使用28,29两台服务器。

​	从上面的总体环境就能看出有几个坑我一定会遇到：

* 项目和nginx部署不在同一台服务器上，生成的文件以及其他静态资源是访问不到的，这个需要利用到nginx的mount命令进行挂载。
* 缓存使用两台服务器，配置文件需要进行相应的更改。
* base、单点服务集成是部署在不同的docker镜像中的，想要集成访问地址需要配置正确
* ......

坑实在太多，我也没能了解到所有的细节，有些坑也只能说一下大概错误原理是什么，具体解决办法也得进行搜索才行。

## 具体流程

​	公司首先发了[演示环境部署应用-规划表](C:\Users\19387\Documents\WeChat Files\wxid_u5e3m4hke28822\FileStorage\File\2020-10\海淀B1机房-演示环境部署应用-规划表-2020-10-27.xlsx)文档，涵盖了需要部署的环境的详细信息。

然后我准备从191环境拷贝服务。准备好后达成docker镜像。docker的整个结构如下：![image-20201029142728055](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20201029142728055.png)

```shell
src:目录存放服务
images：打包后的镜像保存路径，不过这次部署没有保存到这个目录
debug-build：里面只有一个文件：01-rebuild-images.sh，为打包脚本，内容如下：
    #/bin/bash
    docker build -t pkpai-csscms:1.0.202010294 ./
    docker images ls pkpai-csscms:1.0.202010294
    docker images | grep pkpai-csscms
	下面进行解释脚本意思：
        使用shell，
        docker构建，-t代表标签，：之前为名称，：之后为标签版本号，./表示在本目录下
        查询刚打包的镜像信息，包括镜像的ID等。

k8s：里面只有一个文件：deploy-csscms.yaml，为部署配置内容为：
    apiVersion: apps/v1
        kind: Deployment
        metadata:
         name: csscms
         namespace: csszt
         labels:
           app: csscms
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: csscms      
          template:
            metadata:
              labels:
                app: csscms
            spec:
              containers:
                - name: csscms
                  image: pkpai-csscms:1.0.202010287
                  ports:
                    - containerPort: 8080
                  imagePullPolicy: IfNotPresent
	大致意思是容器内部向外暴露端口为8080，然后各个名称改为具体的服务名称。

```

这时第一个坑来了，打docker镜像需要在有Dockerfile的路径下执行名命令：`sh debug-build/01-rebuild-images.sh`进行打包。

这里的`Dockerfile`也有个坑，Dockerfile内容如下：

```dockerfile
FROM ky-tomcat-mem:v1.1
MAINTAINER images-csscms

ADD ./src/csscms  /usr/tomcat/webapps/csscms
RUN  mkdir -p  /data/02ShareNFSfile/07csscms/www/nfsgxbcms/
RUN chmod 777 /data/02ShareNFSfile/07csscms/www/nfsgxbcms

RUN  chmod -R 777 /data/02ShareNFSfile/07csscms/www/nfsgxbcms
RUN  chmod -R 777 /usr/tomcat/webapps/csscms
RUN  mkdir -p /data/02ShareNFSfile/07csscms/HanLP

ADD ./src/2cmsothers/HanLP /data/02ShareNFSfile/07csscms/HanLP
ENV LANG C.UTF-8

EXPOSE 8080
```

之前的`Dockerfile`没有赋权限的操作，导致镜像打包后的静态资源路径不能插入，访问的操作。

打完镜像后要部署到k8s上，这时候就要执行·` kubectl -n csszt create -f deploy-csscms.yaml`命令。

执行完后需要到麒麟云环境上创建对应的服务，依据文档[麒麟容器云v10-部署发布服务用户参考手册](C:\Users\19387\Documents\WeChat Files\wxid_u5e3m4hke28822\FileStorage\File\2020-10\麒麟容器云v10-部署发布服务用户参考手册.docx),创建完成后返回服务端口，拿到这个端口到180服务器上的统一nginx配置代理。

以上只是理想操作，实际上遇到了很多其他坑。

#### 附件上传成功，但是访问站点nginx报错403

这是因为启动用户没有权限访问代理的路径，最好的解决办法就是使用root用户启动nginx，并且需要在`nginx.conf`文件中配置`user root`。

#### 访问不到域名？

访问不到域名需要在本地hosts文件中添加域名映射，并且需要配置DNS域名解析服务器地址，确保能够访问到这个域名。

#### 生成站点但是页面没有重新生成，但是附件能看的到？

这个是因为服务所在服务器和nginx所在服务器不在同一个服务器下，生成只会服务器地址生成，也就是说生成的文件在服务所在的服务器，而访问的是nginx所在的服务器，至于图片能访问到，是因为附件存储的是相对路径，利用相对路径也就能访问到nginx服务器下的路径了。

## 文件配置路径

内容管理文件路径：/data/02ShareNFSfile/07csscms/www/nfsgxbcms

全文检索文件路径：/data/02ShareNFSfile/fulltext/cssoa/webapps/cssoa/upload/indexdir/