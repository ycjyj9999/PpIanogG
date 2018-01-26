# 完全理解JAVA异常处理
## 1.前言

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

在Java语言中，异常可以分为俩类：受检异常（checked exceptions）和非受检异常（unchecked exceptions）。Unchecked异常包括java.lang.RuntimException、java.lang.Error(Error属于特殊情况，因为原理比较相似，所以在这放一起讨论)以及它们的子类，其他异常都是Checked异常。
所有异常最终都继承自Throwable类。以下许多概念解释参考自《Java虚拟机规范第八版》和 [深入理解java异常](http://blog.csdn.net/hguisu/article/details/6155636)。

* **Throwable：** 有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。

* **Error：** 是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM
（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。
* **Exception：** 是程序本身可以处理的异常。

Exception 类有一个重要的子类 RuntimeException。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常NullPointerException、ArithmeticException和 ArrayIndexOutOfBoundException。

注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是**Java编译器**会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

不可查异常(**编译器不要求强制处置的异常**):包括运行时异常（RuntimeException与其子类）和错误（Error）。

Exception 这种异常分两大类运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。

* **运行时异常**：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。

* **非运行时异常 （编译异常）**：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

## 3.JAVA异常的本质
看了这么多概念，你可能还是不知道上面代码的原理和本质，下面是因为你没有明白异常的底层原理，如果对JAVA语言规范和JVM不太感兴趣，可以跳过这段，直接看后面的Java异常总结。
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

**注意：在Java6之前，Oracle的Java编译器使用jsr、jsr_w和ret指令来实现finally字句。从Java6开始，已经不再使用这些指令，所以下文中也不会出现**。

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

异常处理表的每一项都包含三个信息：处理哪部分代码跑出的异常、哪类异常，以及异常处理器的代码在哪里。在table中，catch的from为0，to为8，**即它的监控范围为0-7字节，如果这里出现了编译器判断为异常的语句，代码会跳转至target目标处，这个表中是8，意思是会跳转至第八字节的字节码。**每次捕获到异常会弹出栈顶异常并将新的异常推入栈顶。也就是说新的异常会被写入栈顶，优先被检查到。一般来说catch中是不会出现编译异常的，但是这里我们
人为的抛出了一个异常。它顺利的被监视0-17字节的finally异常处理器捕获，并进行操作。

当有athrow被执行，那么这个异常会查找自己的表去找到对应处理自己的异常，如果在当前方法中没有找到，则会跳到上一层栈中去寻找，直至跳转出程序，代码编译报错，**显示有未处理的异常**。

一般来说，我们不会在catch和finally中抛出异常，所以本文的实例代码是不常见也是不合规的写法。**但是我们为了研究透彻它的原理，我们人为的造出了这样的特例。在文章开始的例子中，testEX2中的catch中throw出的异常，并没有被testEx中的catch捕获，这是为什么呢？**
注意下面的例子：
```java
package jt_test; 
public class test1 {
    double var1;
    double testEx() throws Exception {
        var1 = 0;

        try {
            throw new Exception();
        } catch (Exception var6) {
            var1 = 1;
            throw var6;
        } finally {
            var1 =2;
            return var1;   //在次javap中被注释
        }
    }

    public static void main(String[] var0) {
        return;
    }
}

```

我们对这个代码进行俩次javap -c操作，第二次时，注释掉其中finally的return语句。结果如下：
```C
double testEx() throws java.lang.Exception;
        // 原代码                          //注释return后
    Code:
       0: aload_0                         0: aload_0 
       1: dconst_0                        1: dconst_0
       2: putfield      #2                2: putfield      #2 
       5: new           #3                5: new           #3
       8: dup                             8: dup
       9: invokespecial #4                9: invokespecial #4 
      12: athrow                         12: athrow
      13: astore_1                       13: astore_1
      14: aload_0                        14: aload_0 
      15: dconst_1                       15: dconst_1
      16: putfield      #2               16: putfield      #2
      19: aload_1                        19: aload_1 
      20: athrow                         20: athrow
      21: astore_2                       21: astore_2
      22: aload_0                        22: aload_0
      23: ldc2_w        #5               23: ldc2_w        #5
      26: putfield      #2               26: putfield      #2
      29: aload_2                        29: aload_0 
      30: athrow                         30: getfield      #2
                                         33: dreturn
    Exception table:
       from    to  target type
           5    13    13   Class java/lang/Exception
           5    22    21   any


```

注释return后的table我并没有写上，因为排版不太方便，但它的内容和注释return之前是一模一样的，也就是说**finally里的return操作并不影响它作为异常处理器的功能**。

对比可得，他们的0-29字节都是一样的，区别是第30字节。在我使用的Oracle的Java8编译器规范里，finally是被规定为执行throw或者return的。如果finally区块内没有return，则默认执行athrow指令，将拿到的异常抛出。

而当finally区块内出现了return语句，则athrow被覆盖，**也就是说如果在finally监视范围内的未被处理的异常将会丢失，也就产生了文章开头的现象**。

**在编译期间，代码出现的异常，会被解释为throw语句，被异常表中对应的异常处理器所察觉并处理，若异常层层抛出而最终没能被正确处理，则编译报错**
### 运行异常

运行异常在上面的概念中解释的很准确，我来举个例子更方便大家理解：代码中的很多异常都是不会被编译器发现的，比如若在一份代码中出现int b = 1/0；则会出现除数不能为0的异常，如果这部分代码没有处于异常处理器的监视范围，那么代码将运行报错并把异常打印出来。这类异常，我们称为运行异常。

在Java8虚拟机规范中，对出现运行异常的处理并没有做出明确的规范，所以每个虚拟机有自己的处理异常的设计。还是以上面的int赋值语句为例来进行说明。

它在JVM中经历了这些考验：(以下纯属个人理解，如有错误还望指出)

* ①字节码解释器的pc指向了这条语句所在的位置，读出这条字节码；
* ②匹配到idiv指令(将俩个整数相除)，检查出第二个参数(除数)为0，抛出异常。(**这里的抛出异常与上面编译期间不同，这里已经在JVM运行期间了**)；
* ③获取该异常信息并与当前执行栈帧的异常表进行匹配，若匹配则执行匹配处的操作，**若不匹配则跳出当前栈帧并继续寻找，直至匹配正确的异常处理器**；
* ④若最终没有匹配成功，程序报错，运行结束。

注意:

* 在虚拟机内部，catch和finally里已经完全是一样的东西了没有任何区别。

* **在运行期间用于查找的异常表和上面的表是不一样的，上面的表是写在class文件中的二进制，而这里的表是经过类加载器加载并解析过的数据结构**

* **在虚拟机内部的栈帧结构中，一般会有指向自己代码结束处return的指针和指向上一级调用处的指针，所以在异常查找时指针可以在虚拟机栈中穿梭** 

* **不同的虚拟机都有不同的异常处理机制，这里只是提出了一种方案**

## Java异常总结

对没有看中间的底层分析的同学关于开头处的代码运行结果的原因总结一下，俩句话就是：

* **无论在何处return，当前方法的finally总会被执行。**


* **finally里的return会吃掉它之前捕捉的所有未处理异常。**

### 异常处理语句的语法规则：

* ①必须在 try 之后添加 catch 或 finally 块。try 块后可同时接 catch 和 finally 块，但至少有一个块。
* ②必须遵循块顺序：若代码同时使用 catch 和 finally 块，则必须将 catch 块放在 try 块之后。
* ③catch 块与相应的异常类的类型相关。
* ④一个 try 块可能有多个 catch 块。若如此，则执行第一个匹配块。即Java虚拟机会把实际抛出的异常对象依次和各个catch代码块声明的异常类型匹配，如果异常对象为某个异常类型或其子类的实例，就执行这个catch代码块，不会再执行其他的 catch代码块
* ⑤可嵌套 try-catch-finally 结构。
* ⑥在try-catch-finally 结构中，可重新抛出异常。
* ⑦除了下列情况，总将执行 finally 做为结束：JVM 过早终止（调用 System.exit(int)）；在 finally 块中抛出一个未处理的异常；计算机断电、失火、或遭遇病毒攻击。

### 异常语句的执行顺序：

* ①当try没有捕获到异常时：try语句块中的语句逐一被执行，程序将跳过catch语句块，执行finally语句块和其后的语句；

* ②当try捕获到异常，catch语句块里没有处理此异常的情况：当try语句块里的某条语句出现异常时，而没有处理此异常的catch语句块时，此异常将会抛给JVM处理，finally语句块里的语句还是会被执行，但finally语句块后的语句不会被执行；

* ③当try捕获到异常，catch语句块里有处理此异常的情况：在try语句块中是按照顺序来执行的，当执行到某一条语句出现异常时，程序将跳到catch语句块，并与catch语句块逐一匹配，找到与之对应的处理程序，其他的catch语句块将不会被执行，而try语句块中，出现异常之后的语句也不会被执行，catch语句块执行完后，执行finally语句块里的语句，最后执行finally语句块后的语句；


## 最后
这篇博客前前后后实验，查找资料，总结博客花了3天。对于自己来说，我只是想不断钻研计算机语言的原理和细节，我很满足。

但是时间是有限的，像我这样去钻牛角尖是不可取的，大家遇到类似的问题还是不要太追求细节。我的师兄对我说，对于底层的实现，能找到一种自圆其说的方案就可以了，我也推荐大家这样做。

希望读完这篇文章你能有所收获。

