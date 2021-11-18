



参考文章：https://mp.weixin.qq.com/s/798wedANtsalz2Nd8zYBPA



```java
public class Test {
    public static void main(String[] args) {
        // 会在方法区的常量池创建常量"abc"
        String st1 = "abc";
        String st2 = "abc";
        // 创建2个对象
        // 1.会先在方法区常量池找"abc"，没有就创建
        // 2.常量池有"abc"后，new 会在堆上创建一个"abc"的拷贝副本
        // st3内存地址指向堆
        String st3 = new String("abc");
        // 因为"a"、"b"、"c"本来就在常量池中，+会直接创建常量池的"abc"，所以st4指向常量池
        String st4 = "a" + "b" + "c";
        // 在方法区的常量池创建常量"ab"
        String st5 = "ab";
        // +运算符的原理：
        //   由StringBuilder或者StringBuffer类和里面的append方法实现拼接，
        //   然后调用 toString() 把拼接的对象转换成字符串对象，
        //   最后把得到字符串对象的地址赋值给变量
        // 小结：先是在堆中创建StringBuffer类的内存"abc"，再调用toString在堆中创建"abc"，st6指向堆中toString后的地址
        String st6 = st5 + "c";

        System.out.println(st1 == st2); // true
        System.out.println(st1 == st3); // false
        System.out.println(st1 == st4); // true
        System.out.println(st1 == st6); // false
    }
}
```