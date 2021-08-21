


[JAVA8视频传送门](https://www.bilibili.com/video/BV1ut411g7E9?from=search&seid=7238829590391996939)


* 前言
    * JAVA8的2大改变
    * Stream API
* 1.员工实体类
* 2.员工数据库初始化
* 3.Stream API 操作集合
    * 3.1 map
    * 3.2 sorted
    * 3.3 allMatch、anyMatch、noneMatch
    * 3.4 findFirst、findAny
    * 3.5 collect（结束语句用法1）
    * 3.6 reduce（结束语句用法2）
    * 3.7其他（功能已经被collect包含）




### 前言

**JAVA8的2大改变**

- Lambda表达式（箭头函数）：lambda 是一个匿名函数

- Stream API 操作集合

注：函数式接口：只包含一个抽象方法的接口。函数式接口可以用Lambda表达式。



**Stream API** 

- stream顺序流 和 parallelStream()并行流
- filter  
- distinct
- skip
- map
- sorted
- allMatch
- findFirst
- count
- max
- min
- forEach
- reduce (map + reduce)
- collect (Collectors.toList...)
- ...





### 1.员工实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {

    private int id;
    private String name;
    private int age;
    private double salary;
    private Status status;

    public enum Status {
        FREE, BUSY, VOCATION;
    }

    // 不带状态的构造方法
    public Employee(int id, String name, int age, double salary) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}
```



### 2.员工数据库初始化

```java
    List<Employee> emps = Arrays.asList(
            new Employee(102, "李四", 59, 6666.66, Status.BUSY),
            new Employee(101, "张三", 18, 9999.99, Status.FREE),
            new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
            new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(105, "田七", 38, 5555.55, Status.BUSY)
    );
```



### 3.Stream API 操作集合

#### 3.1 map

```java
    @Test
    void test01() {
        List<String> strList = Arrays.asList("aaa", "bbb", "ccc");
        
        List<String> updateList = strList.stream()
                .map(String::toUpperCase)
                .collect(Collectors.toList());
    }
```




#### 3.2 sorted

```java
    @Test
    void test02() {
        List<String> empList = emps.stream()
                .sorted((e1, e2) -> {
                    if (e1.getAge() == e2.getAge()) {
                        return e1.getName().compareTo(e2.getName());
                    } else {
                        return Integer.compare(e1.getAge(), e2.getAge());
                    }
                })
                .map(Employee::getName)
                .collect(Collectors.toList());
    }
```


#### 3.3  allMatch、anyMatch、noneMatch

```java
    /**
     * 1.allMatch——检查是否匹配所有元素
     * 2.anyMatch——检查是否至少匹配一个元素
     * 3.noneMatch——检查是否没有匹配的元素
     */
    @Test
    void test03() {
        // false
        boolean bl = emps.stream()
                .allMatch((e) -> e.getStatus().equals(Status.BUSY));

        // true
        boolean bl1 = emps.stream()
                .anyMatch((e) -> e.getStatus().equals(Status.BUSY));

        // false
        boolean bl2 = emps.stream()
                .noneMatch((e) -> e.getStatus().equals(Status.BUSY));
    }

```



#### 3.4 findFirst、findAny


```java
    /**
     * 1.findFirst —— 返回第一个元素
     * 2.findAny —— 返回当前流中的任意元素
     */
    @Test
    void test051() {
        Optional<Employee> first = emps.stream()
                .sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
                .findFirst();
        
        Optional<Employee> op2 = emps.parallelStream()
                .filter((e) -> e.getStatus().equals(Status.FREE))
                .findAny();
    }
```


#### 3.5 collect（结束语句用法1）

```java
    // 拼接字符串
	@Test
    void test04() {
        // ----李四,张三,王五,赵六,赵六,赵六,田七----
        String str = emps.stream()
                .map(Employee::getName)
                .collect(Collectors.joining("," , "----", "----"));
    }
```



```java
    // 计算工资总数
    @Test
    void test05() {
        // 48888.84000000001
        Optional<Double> sum = emps.stream()
                .map(Employee::getSalary)
                .collect(Collectors.reducing(Double::sum));
        sum.get()
    }
```



```java
    @Test
    void test06() {
        // 9999.99  获取最大工资数
        Optional<Double> max = emps.stream()
                .map(Employee::getSalary)
                .collect(Collectors.maxBy(Double::compare));
        System.out.println(max.get());

        // Employee(id=103, name=王五, age=28, salary=3333.33, status=VOCATION) 
        // 获取最小工资数人员的信息
        Optional<Employee> op = emps.stream()
                .collect(Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));

        // 48888.840000000004
        // 获取人员工资总数
        Double sum = emps.stream()
                .collect(Collectors.summingDouble(Employee::getSalary));
        
        // 6984.120000000001
        // 获取平均工资
        Double avg = emps.stream()
                .collect(Collectors.averagingDouble(Employee::getSalary));
        
        // 7
        // 获取人员数
        Long count = emps.stream()
                .collect(Collectors.counting());
        
        // 获取平均值、最小值、最大值等
        DoubleSummaryStatistics dss = emps.stream()
                .collect(Collectors.summarizingDouble(Employee::getSalary));
        dss.getMax();
        dss.getMin();
        dss.getAverage();
        // ...
    }
```



```java
    @Test
    void test07() {
        // 分组
        Map<Status, List<Employee>> map = emps.stream()
                .collect(Collectors.groupingBy(Employee::getStatus));
    }
```



```java
    // 多级分组
    @Test
    public void test8(){
        Map<Status, Map<String, List<Employee>>> map = emps.stream()
                .collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
                    if(e.getAge() >= 60)
                        return "老年";
                    else if(e.getAge() >= 35)
                        return "中年";
                    else
                        return "成年";
                })));
    }
```



```java
   // 分区
    @Test
    public void test9(){
        Map<Boolean, List<Employee>> map = emps.stream()
                .collect(Collectors.partitioningBy((e) -> e.getSalary() >= 5000));
    }
```


#### 3.6 reduce（结束语句用法2）

```java
    /**
     * reduce：归约  可以将流中元素反复结合起来，得到一个值。
     * 经典用法：map + reduce
     */
    @Test
    void test10() {
        // 55
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        Integer sum = list.stream()
                .reduce(0, (x, y) -> x + y);

        // 48888.84000000001
        Optional<Double> op33 = emps.stream()
                .map(Employee::getSalary)
                .reduce(Double::sum);
    }
```



```java
    /** 需求：搜索名字中 “六” 出现的次数
     *  - flatMap —— 把所有流连接成一个流
     *  - 注：Java8Test是个类 这个类中有方法filterCharacter()
     */
    @Test
    void test011() {
        // 3
        Optional<Integer> sum11 = emps.stream()
                .map(Employee::getName)
                .flatMap(Java8Test::filterCharacter)
                .map(e -> {
                    if(e.equals('六'))
                        return 1;
                    else
                        return 0;
                }).reduce(Integer::sum);
    }
    

    public static Stream<Character> filterCharacter(String str){
        List<Character> list = new ArrayList<>();

        for (Character ch : str.toCharArray()) {
            list.add(ch);
        }

        return list.stream();
    }
```






#### 3.7其他（功能已经被collect包含）

```java
    /**
     * 1.count——返回流中元素的总个数
     * 2.max——返回流中最大值
     * 3.min——返回流中最小值
     */
    @Test
    void test051() {
        // 3
        long count = emps.stream()
                .filter((e) -> e.getStatus().equals(Status.FREE))
                .count();

        // 9999.99
        Optional<Double> op11 = emps.stream()
                .map(Employee::getSalary)
                .max(Double::compare);

        // Employee(id=103, name=王五, age=28, salary=3333.33, status=VOCATION)
        Optional<Employee> op21 = emps.stream()
                .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
    }
```













