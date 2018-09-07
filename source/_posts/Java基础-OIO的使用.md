---
title: Java 网络编程基础（一）- IO
date: 2018-09-04 16:16:32
tags: Java SE
---

# Java学习（一）- IO

## Java IO(java.io)包的内容

Java IO包主要包含了输入和输出到文件、网络流、内存缓冲区等，却不包含开启网络链接的类，但只要打开了一个网络连接，我们就可以用Java IO的InputStream和OutputStream来进行读写操作；

## Java IO类总结

![Java IO类梗概](https://s1.ax1x.com/2018/09/04/iSoE7V.md.jpg)

如上图所示，Java IO流主要分为字符流和字节流，Reader和Writer等读写是**基于字符**的，InputStream和OutputStream是**基于字节流**的。
除此之外，还有一个特殊的类java.io.RandomAccessFile，RandomAccessFile 虽然属于java.io下的类，但它不是InputStream或者OutputStream的子类；它也不同于FileInputStream和FileOutputStream。 FileIsnputStream 只能对文件进行读操作，而FileOutputStream 只能对文件进行写操作；RandomAccessFile与输入流和输出流不同之处就是RandomAccessFile可以**访问文件的任意地方**同时支持文件的读和写，并且它支持**随机访问**。RandomAccessFile包含InputStream的三个read方法，也包含OutputStream的三个write方法。同时RandomAccessFile还包含一系列的readXxx和writeXxx方法完成输入输出。 

## Java IO异常处理

### Java7之前的Try-Catch-Finally处理风格

    private static void printFile() throws IOException {
        InputStream input = null;

        try {
            input = new FileInputStream("file.txt");

            int data = input.read();
            while(data != -1){
                System.out.print((char) data);
                data = input.read();
            }
        } finally {
            if(input != null){
                input.close();
            }
        }
    }

看起来非常不简洁，很有可能导致许多批判Java人口中的高度问题。

### Java7后的Try-with-resources

    private static void printFileJava7() throws IOException {

        try(  FileInputStream     input         = new FileInputStream("file.txt");
            BufferedInputStream bufferedInput = new BufferedInputStream(input)
        ) {

            int data = bufferedInput.read();
            while(data != -1){
                System.out.print((char) data);
        data = bufferedInput.read();
            }
        }
    } 

创建的资源会按照与创建/排列相反的顺去去自动关闭，只要实现了AutoClosable接口的类（需要实现close()方法）都可以使用try-with-resources结构。

## OIO的并发问题

对于同一个同一个InputStream/OutputStream或者Reader/Writer，不应当有多个线程进行操作,因为无法保证每个线程会读/写到哪里。


