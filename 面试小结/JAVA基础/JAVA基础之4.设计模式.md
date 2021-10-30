

# 设计模式

## 1.单例

```java
/**
 * 懒汉式：线程安全（推荐使用）
 */
public class Singleton06 {
    public static void main(String[] args) {
        Singleton instance1 = Singleton.getInstance();
        Singleton instance2 = Singleton.getInstance();
        //true
        System.out.println(instance1==instance2);
    }
}

class Singleton{
    //构造器  new
    private Singleton() { }

    //final修饰必须要有初始值
    //volatile防止构造器的指令重排
    private static volatile Singleton instance;


    /**
     * 1.如果第一次检查instance不为null，那就不需要执行下面的加锁和初始化操作。
     *   因此，可以大幅降低synchronized带来的性能开销
     * 2.volatile防止构造器的指令重排
     * 3.线程1进入if判断还没继续；线程2也进入if判断
     *   如果指令重排，线程2会判断空然后进入拿锁初始化对象；
     *   如果禁止指令重排，线程2不会判断空保证单例
     * 4.时序图解：https://www.jianshu.com/p/a9ecac4bd753
     */
    public static Singleton getInstance() {
        //提高效率
        if (instance==null){
            synchronized (Singleton.class){  //锁Class。锁：锁对象、锁Class
                //禁止指令重排
                if (instance==null){
                    /**
                     * 1.创建对象的3行伪代码
                     *      memory=allocate();        //1:分配对象的内存空间
                     *      ctorInstance(memory);     //2:执行构造方法，初始化对象
                     *      instance = memory;        //3:设置instance指向刚分配的内存地址
                     *
                     * 2.创建对象步骤2和步骤3可能会重排序
                     *      memory=allocate();        //1:分配对象的内存空间
                     *      instance = memory;        //3:设置instance指向刚分配的内存地址
                     *                                //注意，此时对象还没有被初始化！
                     *                                //插入线程2判断对象是否为空
                     *      ctorInstance(memory);     //2:初始化对象
                     */
                    //不是原子操作（伪代码有3步  没有禁止指令重排，A线程还在"初始化对象",B线程判空为false置接返回空对象）
                    instance=new Singleton();
                }
            }
        }
        return instance;
    }
}
```





## 2.设计模式（解决多if判断）

"模板 + 工厂 + 策略"   -- > 替换多if判断

实际使用：https://gitee.com/domineering_red_tide/design_patterns_23/tree/master/mydesign/src/main/java/com/baidu/mydesign/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E4%BD%BF%E7%94%A8/%E5%B7%A5%E5%8E%82_%E6%A8%A1%E6%9D%BF_%E7%AD%96%E7%95%A5/demo

