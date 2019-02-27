---
title: Java-注解
categories: Java
tags: [Java, 注解, Spring Annotation]
---

##### 元注解

要自定义注解，首先要知道元注解。元注解的作用就是注解其他注解，java.lang.annotation包中定义了四个注解：@Target, @Retention, @Documented, @Inherited。

- @Target

@Target用于描述注解的使用范围（即：被描述的注解可以用在什么地方）

@Target的取值有：

   1) ElementType.CONSTRUCTOR:用于描述构造器
   2) ElementType.FIELD:用于描述域
   3) ElementType.LOCAL_VARIABLE:用于描述局部变量
   4) ElementType.METHOD:用于描述方法
   5) ElementType.PACKAGE:用于描述包
   6) ElementType.PARAMETER:用于描述参数
   7) ElementType.TYPE:用于描述类、接口(包括注解类型) 或enum声明
   8) ElementType.ANNOTATION_TYPE 
   9) ElementType.TYPE_PARAMETER
   10) ElementType.TYPE_USE

- @Retention

@Retention表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）

@Retention的取值有：

   1) RetentionPoicy.SOURCE:在源文件中有效（即源文件保留）
   2) RetentionPoicy.CLASS:在class文件中有效（即class保留）
   3) RetentionPoicy.RUNTIME:在运行时有效（即运行时保留）

- @Documented

@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

- @Inherited

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。

