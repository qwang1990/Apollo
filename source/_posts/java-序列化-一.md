---
title: java 序列化(一)
date: 2017-07-23 19:37:19
categories: 
- 后端 
tags: 
- java 
---
# java序列化
## 废话
序列化就是为了保存在内存中的各种对象的状态，并且可以把保存的对象状态再读出来。虽然你可以用你自己的各种各样的方法来保存Object States，
但是Java给你提供一种应该比你自己好的保存对象状态的机制,那就是序列化。在java中实现Serializable接口就说明该类可以被序列化。
一般用于:
 - 把内存中对象保存在文件或数据库中
 - 网络传输

## 几个问题
1.静态成员变量可以被序列化么？
 不行的。
 ```java
    public class Foo implements Serializable {
        private static final long serialVersionUID = -3450064362986273896L;
           public static int a = 1;
    }
    
    public class SerialMain {
    
        public static void main(String[] arg) throws IOException, ClassNotFoundException {
            Foo myFoo = new Foo(); 
            Foo.a = 100;
    
            FileOutputStream fs = new FileOutputStream("foo.ser");
            ObjectOutputStream os = new ObjectOutputStream(fs);
            os.writeObject(myFoo);
            os.close();
    
            Foo.setA(10000);
    
            FileInputStream fi=new FileInputStream("foo.ser");
            ObjectInputStream oi=new ObjectInputStream(fi);
            Foo box=(Foo)oi.readObject();
            oi.close();
            System.out.println(box);
    
        }
    }
```
结果为 a=10000
因为序列化是记录对象的状态，并不记录类的状态。

2.serialVersionUID的作用
serialVersionUID可以理解为可序列化对象的一个运行时版本号，当反序列化时jvm会对比目标对象和源数据之间的版本号是否一致，如果不一致会
报InvalidClassException。导致serialVersionUID不一致的可能有很多，比如对象属性的变化，环境的不同等，所以为了兼容性，建议Serializable对象
都定义一个serialVersionUID成员变量，格式如上代码所示。这样既能避免环境问题，又能当对象版本变化时不报错（此时只序列化能识别的对象，但是我感觉这
并不是一个好的现象，这样会不能及时发现升级，有可能造成更大的错误。）

3.enum序列化
enum序列化其实被序列化的是名字，反序列化时用valueof方法来匹配，这个比较坑，特别是系统升级时，所以使用时一定要谨慎。

# protobuff
## 什么是protobuf
protobuff是google开源的语言无关，平台无关的可扩展的序列化结构（就像xml一样，不过更简单，更高效）。用户可以通过一次定义如何构建数据，然后可以使用
不同的语言方便的读写这些数据。






