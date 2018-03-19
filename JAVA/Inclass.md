# java中四种内部类的基本知识
## 1.静态内部类：作为类的静态成员，存在于某个类的内部。
**静态内部类虽然是外部类的成员，但是在未创建外部类的对象的情况下，可以直接创建静态内部类的对象。静态内部类可以引用外部类的静态成员变量和静态方法，但不能引用外部类的普通成员。**
```java

//静态内部类的测试程序
public class Outter {
    static int a=1;
    int b=5;
    static void test(){
        System.out.println("外部类的静态方法");
    }
    static class Inner{
        public void test2(){
            System.out.println("a的值为"+a);;//直接引用外部类的静态成员变量
            test();//直接引用外部类的静态方法
            //b++;试图引用外部类的非静态成员变量，不能通过编译
            System.out.println("静态内部类的方法");
        }
    }
    public static void main(String[] args) {
        Inner in=new Inner();//静态内部类的对象可以直接创建，无需先创建外部类的对象
        in.test2();
    }
}
```

## 2.成员内部类：作为类的成员，存在于某个类的内部。
**成员内部类可以调用外部类的所有成员，但只有在创建了外部类的对象后，才能调用外部的成员。**
```java
public class Outter1 {
    static int a=1;
    int b=0;
    public static void test1(){
        System.out.println("外部类的静态方法");
    }
    public void test2(){
        System.out.println("外部类的非静态方法");
    }

    class Inner{
        public void test(){
            System.out.println("在成员内部类的方法中");
            test1();//调用外部类的静态方法
            test2();//调用外部类的非静态方法
            System.out.println(a+b);//访问外部类的静态成员变量和非静态成员变量
        }
    }
    public static void main(String[] args) {
        //Inner in=new Inner();成员内部类的对象不能直接创建，会报错
        Outter1 out=new Outter1();//先创建外部类的对象
        Inner in=out.new Inner();//注意：！！成员内部类的对象必须通过外部类的对象创建
    }
}
```
## 3.局部内部类：存在于某个方法的内部。
**局部内部类只能在方法内部中使用，一旦方法执行完毕，局部内部类就会从内存中删除。**
**必须注意：如果局部内部类中要使用他所在方法中的局部变量，那么就需要将这个局部变量定义为final的。**
```java
public class Outter2 {
    int a=10;
    public void test(){
        final int c=5;
        System.out.println("在外部类的方法中");
        class Inner{
            int b=20;
            void test1(){
                System.out.println("局部内部类的方法中");
                System.out.println(c);//注意：如果局部内部类中要使用他所在方法中的局部变量，那么就需要将这个局部变量定义为final的。
            }
        }
        Inner inner=new Inner();
        inner.test1();
    }

    public static void main(String[] args) {
        Outter2 outter=new Outter2();
        outter.test();
    }
}
```


## 4.匿名内部类：存在于某个类的内部，但是无类名的类。
**其实访问者设计模式是这种内部类的典型应用**
**匿名内部类的定义与对象的创建合并在一起，匿名内部类一般通过如下形式定义，并且在定义的同时进行对象的实例化。**
new 类或者接口的名字(){

  //匿名内部类的主体，大括号中是匿名内部类的主体，这个主体就是类或者接口的实现，如果是类，那么匿名内部类是该类的子类，如果是接口，匿名内部类需要完成接口的实现。

}
```java
class Person{
    public void show(Message message){
        message.show();
    }
}

class Message{
    public void show(){
        System.out.println("在Message类中");
    }
}

public class Outter3 {
    public static void main(String[] args) {
        Person person=new Person();
        person.show(new Message(){
            public void show(){
                System.out.println("在匿名内部类中");
            }
        });
    }
}

}
```
**java中绝大多数情况下，类的访问修饰符只有public和默认这两种，但也有例外。静态内部类和成员内部类还可以有protected和private两种**
