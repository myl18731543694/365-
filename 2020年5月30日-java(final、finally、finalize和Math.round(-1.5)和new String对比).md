 

## java(final、finally、finalize和Math.round(-1.5)和new String对比)

#### 1. final、finally、finalize 有什么区别？

**final**

可以修饰变量，表示变量初始化以后，不能再被改写

```java
final String a = null;
// 报错a被初始化为null，不能再被赋值
a = "a";
```

```java
final String a;
// a没有初始化，可以赋值
a = "a";
```

修饰在类上，不可以被子类重写。也可以修饰在static方法上，不过没有意义

```java
class AAA{
    public final void aaa(){
        System.out.println("aaaa");
    }

    public void bbb(){
        System.out.println("bbbb");
    }
}

public class Test extends AAA{
    public static void main(String[] args) {
    }

    @Override
    public void bbb(){
        System.out.println("bbbb chongxie");
    }

    /**
     * 报错，不能重写父类final方法
     */
    @Override
    public void aaa(){
        System.out.println("aaaa chongxie");
    }
}
```

修饰在类上，不能继承final类

```java
final class  AAA{
}

/**
 * 报错，不能继承final类
 */
public class Test extends AAA{
    public static void main(String[] args) {
    }
}
```

**finally**

try、catch、finally，不管代码有没有异常最后都会走finally

**finalize**

Object中的方法，当被垃圾回收掉的时候会执行这个方法

#### 2. String str="i"与 String str=new String(“i”)一样吗？

```java
package com.learn.othercode.learn.jdbcspace;

public class Test {
    public static void main(String[] args) {
        // 是把 "i" 这个常量池的 地址给了 str
        String str = "i";
        // new String("i") 则是把这个 String对象地址给了 str1
        String str1 = new String("i");
        // false
        System.out.println(str == str1);
        // true
        System.out.println(str.equals(str1));

        System.out.println("**************************");

        /*
        Integer bbb = 127会变成 Integer.valueOf(127)
        而new Integer(127)，并不会走缓冲池子，所以 == 是false
        */
        Integer aaa = new Integer(127);
        Integer bbb = 127;
        System.out.println(aaa == bbb);
        System.out.println(aaa.equals(bbb));

        System.out.println("**************************");

        /*
         Integer.valueOf() -128 ~ 127 之间会指向常量池
         所以 == 是 true
         */
        Integer ccc = Integer.valueOf(127);
        Integer ddd = 127;
        System.out.println(ccc == ddd);
        System.out.println(ccc.equals(ddd));
    }

}
```



#### 3. java 中的 Math.round(-1.5) 等于多少？

**round()方法在实现四舍五入的时候实际上是不论正负的,它会先将这个数加上0.5然后在像下取整.**

```java
// -1.5 + 0.5 = -1 向下取整 -1
System.out.println(Math.round(-1.5));
// -1.6 + 0.5 = -1.1 向下取整 -2
System.out.println(Math.round(-1.6));
// 1.1 + 0.5 = 1.6 向下取整 1
System.out.println(Math.round(1.4));
```

