



### 4. Spring的@Transactional事务的其他问题

#### 4.1 嵌套事务回滚多了

```java
@Service
public class RoleService {
    @Transactional(propagation = Propagation.NESTED)
    public void doOtherThing() {
        System.out.println("保存role表数据");
    }
}
```

```java
// 原本是希望调用roleService.doOtherThing方法时，如果出现了异常，只回滚doOtherThing方法里的内容，不回滚 userMapper.insertUser里的内容，即回滚保存点
// 因为doOtherThing方法出现了异常，没有手动捕获，会继续往上抛，到外层add方法的代理方法中捕获了异常。所以，这种情况是直接回滚了整个事务，不只回滚单个保存点
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        userMapper.insertUser(userModel);
        roleService.doOtherThing();
    }
}
```

```java
// 正确的只回滚保存点的写法
@Slf4j
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        userMapper.insertUser(userModel);
        try {
            roleService.doOtherThing();
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}
```



#### 4.2 大事务问题及解决

![大事务引发的问题](.\image\大事务引发的问题.png)

```java
// 需要的事务只是3行，却因为其他查询过多而会导致大事务问题
@Service
public class UserService {
    
    @Autowired 
    private RoleService roleService;
    
    @Transactional
    public void add(UserModel userModel) throws Exception {
       query1();
       query2();
       query3();
       // 实际想用的事务就是下面这2行
       roleService.save(userModel);
       update(userModel);
    }
}

@Service
public class RoleService {
    
    @Autowired 
    private RoleService roleService;
    
    @Transactional
    public void save(UserModel userModel) throws Exception {
       query4();
       query5();
       query6();
       // 实际想用的事务就是下面这1行
       saveData(userModel);
    }
}
```

### 5.大事务的解决方法

#### 5.1 将查询select方法放到事务外

```java
// 待解决的大事务
@Service
public class UserService {
    
    @Autowired 
    private RoleService roleService;
    
    @Transactional
    public void add(UserModel userModel) throws Exception {
       query1();
       query2();
       query3();
       // 实际想用的事务就是下面这2行
       roleService.save(userModel);
       update(userModel);
    }
}
```

```java
// 解决大事务
@Servcie
  publicclass ServiceA {
     // 这里采用注入自己，也可以新建service调用doSave方法
     @Autowired
     prvate ServiceA serviceA;
  
     public void save(User user) {
           queryData1();
           queryData2();
           serviceA.doSave(user);
     }
     
     @Transactional(rollbackFor=Exception.class)
     public void doSave(User user) {
         addData1();
         updateData2();
      }
}
```



#### 5.2 用编程式事务替代声明式事务@Transactional

```java

@Service
public class TestService {
    
   @Autowired
   private TransactionTemplate transactionTemplate;
    
   // 声明式事务 
   @Transactional(rollbackFor=Exception.class)
   public void save(User user) {
         queryData1();
         queryData2();
         // 实际想用的事务就是下面这2行
         addData1();
         updateData2();
   }
   
   // 编程式事务 
   public void save(final User user) {
         queryData1();
         queryData2();
         transactionTemplate.execute((status) => {
            addData1();
            updateData2();
            return Boolean.TRUE;
         })
   }
}

```


#### 5.3 事务中避免远程调用

远程调用包括：调用接口，发MQ消息，或者连接redis、mongodb保存数据等

```java
@Service
public class TestService {
    
   @Autowired
   private TransactionTemplate transactionTemplate;
    
   // 声明式事务会导致大事务 
   @Transactional(rollbackFor=Exception.class)
   public void save(User user) {
         callRemoteApi(); // 远程调用 可能很费时间
         addData1();
   }
   
    
   // 编程式事务解决大事务
   public void save(final User user) {
         // 远程调用不在事务中，无法保证数据一致性，需要"重试+补偿机制"达到最终一致性
         callRemoteApi();
       
         transactionTemplate.execute((status) => {
            addData1();
            return Boolean.TRUE;
         })
   }
}
```


####  5.4 事务中避免一次性处理太多数据

```markdown
问题：批量更新1000条数据
解决：分页处理，1000条数据，分50页，一次只处理20条数据，这样可以大大减少大事务的出现
```



####  5.5 非事务执行

```java
   @Autowired
   private TransactionTemplate transactionTemplate;
   
   ...
   
   // 待改进
   public void save(final User user) {
         transactionTemplate.execute((status) => {
            addData(); // 增加数据
            addLog(); // 新增日志
            updateCount(); // 更新数量
            return Boolean.TRUE;
         })
   }

   // 改进后：addLog增加操作日志方法 和 updateCount更新统计数量方法，是可以不在事务中执行的，因为操作日志和统计数量这种业务允许少量数据不一致的情况
   public void save(final User user) {
         transactionTemplate.execute((status) => {
            addData();           
            return Boolean.TRUE;
         })
         addLog();
         updateCount();
   }
```



#### 5.6 异步执行

```java
   @Autowired
   private TransactionTemplate transactionTemplate;
   
   // 待改进
   public void save(final User user) {
         transactionTemplate.execute((status) => {
            order(); // 下单
            delivery(); // 发货
            return Boolean.TRUE;
         })
   }

   // 改进后：下单后不需要立刻发货，可以借助mq发货（mq可以实现最终一致性）
   public void save(final User user) {
         transactionTemplate.execute((status) => {
            order();
            return Boolean.TRUE;
         })
         sendMq();
   }
```