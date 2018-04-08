### JNI(Java Native Interface)
1. Java平台特性，定义一些JNI函数，通过这些函数实现Java与C/C++代码互调。
2. 一般将C/C++代码编译成Android平台可用的so库。


1. AS 下载NDK 设置NDK路径
2. 在项目gradle.properties文件中加上以下代码，表示我们要使用NDK进行开发。
```java 