# 完全理解JAVA异常处理
##1.前言

**先来个热身,想想看下面代码的正确结果是什么**
```java
package jt_test;  
public class ExceptionDemo {
    double testEx() throws Exception {
        try {
            throw new Exception();
        } catch (Exception v2) {
            System.out.println(1);
            return 1;
        } finally {
            System.out.println(2);
        }
    }

    public static void main(String[] var0) {
        ExceptionDemo a = new ExceptionDemo();
        try{
            a.testEx();
        }catch (Exception v) {
        }
        return;
    }
}
```
**正确输出结果是1 2，继续往下看**

```java
package jt_test; 
public class ExceptionDemo {
    double v1 = 0;
    double testEx() throws Exception {
        try {
            System.out.println(1);
            testEx2();
        } catch (Exception v2) {
            System.out.println(5);
            return v1;
        } finally {
            System.out.println(6);
            return v1;
        }
    }
    double testEx2() throws Exception {
        try {
            System.out.println(2);
            throw new Exception();
        } catch (Exception v2) {
            System.out.println(3);
            throw new Exception();
        } finally {
            System.out.println(4);
            return v1;  //这里可以选择注释，比对结果
        }
    }

    public static void main(String[] var0) {
        ExceptionDemo a = new ExceptionDemo();
        try{
            a.testEx();
        }catch (Exception v) {
        }
        return;
    }
}

```
**可能你会得到1 2 3 4 5 6的结果，想到得到这个结果，只需要把testEX2的finally中的return注释掉即可。**

**正确输出结果是1 2 3 4 6，如果你答对并且知道它出现的原理，你就没有必要继续看这篇文章了**

## 2.JAVA异常

在Java语言中，异常可以分为俩类：可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）。Unchecked异常包括java.lang.RuntimException、java.lang.Error以及它们的子类，其他异常都是Checked异常。
所有异常最终都继承自Throwable类。以下许多概念解释参考自《Java虚拟机规范第八版》和 [深入理解java异常](http://blog.csdn.net/hguisu/article/details/6155636)。

* **Throwable：** 有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。

* **Error（错误）:  **是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM
（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。
* **Exception（异常）: **是程序本身可以处理的异常。

Exception 类有一个重要的子类 RuntimeException。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常（NullPointerException、ArithmeticException）和 ArrayIndexOutOfBoundException。

注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是**Java编译器**会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

不可查异常(**编译器不要求强制处置的异常**):包括运行时异常（RuntimeException与其子类）和错误（Error）。

Exception 这种异常分两大类运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。

* **运行时异常**：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。

* **非运行时异常 （编译异常）**：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

## 3.JAVA异常的本质
看了这么多概念，你可能还是不知道上面代码的原理和本质，下面是因为你没有明白异常的底层原理，如果对JAVA语言规范和JVM不太感兴趣，可以跳过这段，直接看后面的代码解释
。

在我看来，抛异常的本质实际上是程序控制权的一种全局转换，说白了，就是偏移量的跳转，从异常抛出的地方转换至处理异常的地方。最近和师兄一起用C手撸了一个throw-catch-finally的接口，所以对异常处理这里的理解颇有心得。
### 编译异常

简单来说，在编译期被javac解释为throw关键字的异常即为编译异常，程序猿必须给它们设计对应异常类型的异常处理器catch或者finally，如果没有编译则会报错。

手撸一个异常的demo用来测试，代码如下
```java
package jt_test; 
public class test2 {
    double var1;
    double testEx() throws Exception {
        try {
            throw new Exception();
        } catch (Exception var6) {
            var1 = 1;
            throw var6;
        } finally {
            return var1;
        }
    }

    public static void main(String[] var0) {
        return;
    }
}
```

javap -c以下，可以清晰的看见原理(字节码注释纯属个人理解，如有错误还望指出)

```C

    double testEx() throws java.lang.Exception;
    Code:
       0: new           #2                  // 创建Exc实例
       3: dup                               // 复制创建Exc的实例引用
       4: invokespecial #3                  // Method java/lang/Exception."<init>":()V 调用Exc构建函数
       7: athrow                            // 抛出异常，由于这里是强制跑出异常，所以显示了athrow命令
       8: astore_1                          // 将栈顶引用对象弹出，并放入局部变量表1的槽位
       9: aload_0                           // 将异常对象(引用类对象)推入栈顶
      10: dconst_1
      11: putfield      #4                  // Field var1:D
      14: aload_1                           // 将栈顶1号槽位的引用对象弹出(查找对应的异常)
      15: athrow                            // 抛出异常
      16: astore_2                          // 存储异常引用
      17: aload_0                           // 将异常对象(引用类对象)推入栈顶
      18: getfield      #4                  // Field var1:D
      21: dreturn
    Exception table:
       from    to  target type
           0     8     8   Class java/lang/Exception
           0    17    16   any


```
这段简单的字节码，可以看见0-7字节为try的执行，8-15为catch的执行，16-21为finally的执行。

这里最下方的Exception table就是我们在代码中撸出的catch和finally。可以发现，对于编译器来说，catch和finally并没有区别，他们是一样的异常处理器。只不过catch的监视范围要远小于finally，并且finally对异常类型的匹配是any。

在table中，catch的from为0，to为8，**即它的监控范围为0-7字节，如果这里出现了编译器判断为异常的语句，代码会跳转至target目标处，这个表中是8，意思是会跳转至第八字节的字节码。**每次捕获到异常会弹出栈顶异常并将新的异常推入栈顶。也就是说新的异常会被写入栈顶，优先被检查到。一般来说catch中是不会出现编译异常的，但是这里我们
人为的抛出了一个异常。它顺利的被监视0-17字节的finally异常处理器捕获，并进行操作。

当有athrow被执行，那么这个异常会查找自己的表去找到对应处理自己的异常，如果在这个栈帧区的方
### 运行异常
