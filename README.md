# springboot 学习笔记

## 1、环境准备：

> 开发工具 idea
>
> jdk1.8
>
> mysql 8.0

## 2、创建一个springboot项目

在idea新建一个project，选择Spring Initializr，这样我们就可以直接联网创建一个springboot项目

![idea创建项目](study/1.PNG)

![idea创建项目](study/2.PNG)

这里我们都选择默认的，项目基于maven管理，并在打包的时候打包为jar包，项目完成之后可以直接丢到服务器，执行命令 java -jar xxxx.jar就可以把我们的项目跑起来。

点击下一步，springboot会让我们选择我们所需要的依赖，这里我们先简单选择几个

> Developer Tools -> Lombok: 这是一个可以省去getter、setter、构造方法、tostring方法等的一个依赖，使用注解就可以实现。
>
> Web -> Spring Web: 可以提供Rest服务  
>
> SQL -> MySQL Driver: 我们使用mysql，所以我们这个依赖帮我们提供mysql驱动
>
> SQL -> MyBatis Framework: 数据库持久层框架

这样，我们就完成了springboot项目的创建。





