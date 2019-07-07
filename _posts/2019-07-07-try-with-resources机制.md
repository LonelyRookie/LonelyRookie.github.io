---
layout:     post                    # 使用的布局（不需要改）
title:      try-with-resources机制               # 标题 
subtitle:   try-with-resources机制（try后面跟小括号()）          #副标题
date:       2019-07-07              # 时间
author:     HuangCanCan             # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - java
---

Java7提供了try-with-resources机制，其类似Python中的with语句，`将实现了`**java.lang.AutoCloseable 接口**`的资源定义在 try 后面的小括号中，不管 try 块是正常结束还是异常结束，这个资源都会被自动关闭`。 try 小括号里面的部分称为 try-with-resources 块。

Java 7 的编译器和运行环境支持新的 try-with-resources 语句，称为 ARM 块(Automatic Resource Management) ，自动资源管理。

使用try-with-resources机制的代码如下所示：

    public static String readFirstLineFromFile(String path) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
            return reader.readLine();
        }
    }

在Java7之前，只能使用下面的语句关闭资源：

    public static String readFirstLineFromFileWithFinallyBlock (String path) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader(path));
        try {
            return reader.readLine();
        } finally {
            if (reader != null) {
                reader.close();
            }
        }
    }

readFirstLineFromFile 方法中，如果 try 块和 try-with-resources 块都抛出了异常，则抛出 try 块中的异常， try-with-resources 块中的异常被忽略；
readFirstLineFromFileWithFinallyBlock 方法中，如果方法 readLine 和 close 都抛出了异常，则抛出 finally 块中的异常， try 块抛出的异常被忽略。

参考：
[官网 try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)