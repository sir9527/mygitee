
```java
// 1.增
db.User.save({name:'zhangsan',age:21,sex:true})
    
// 2.删
db.User.remove(id)
db.User.remove({})
    
// 3.修改    
db.User.update({name:"zhangsan"}, {$set:{age:100, sex:0}}) 
    
// 4.查询    
db.User.find()
db.User.find({name:"zhangsan"})    
db.User.find({age:21}, {'name':1, 'age':1})
db.User.find().sort({age:1})
db.User.find().skip(0).limit(3)
db.User.find({age:{$in:[21,26,32]}})
db.User.find({age:{$gt:20}}).count()
db.User.find({$or:[{age:21}, {age:28}]})
```



[TOC]

参考文档：https://mp.weixin.qq.com/s/_0FSYm7VwYQxjHchWXiivw



数据库、集合、文档



高可用架构
1.Master-Slave 模式（不推荐）
2.Replica Set 副本集模式（推荐）
3.Sharding 模式（推荐）



## 1.配置文件

```java
spring:
  data:
    mongodb:
      host: 127.0.0.1
      port: 27017
      database: test_database
```



## 2.实体类

```java
@Document(collection="persons")
@Data
@ToString
@Accessors(chain = true)
public class Person implements Serializable{

    private static final long serialVersionUID = -3258839839160856613L;

    @Id
    private Long id;

    private String userName;

    private String passWord;

    private Integer age;

    // mongo数据库存时间会慢8个小时 存string可以避免这种情况
    // 索引字段
    @Indexed 
    private String createTime;

}
```



## 3.测试类（增删改查）

```java
@SpringBootTest
class MongoApplicationTests {

    @Autowired
    private MongoTemplate mongoTemplate;


    // 查询集合中的全部文档数据
    @Test
    public void findAll() throws Exception{
        List<Person> result = mongoTemplate.findAll(Person.class);
        System.out.println("查询数量：" + result.size());
    }

    // 查询
    @Test
    public void findByAndCondition() {
        // 条件
        Criteria criteriaUserName = Criteria.where("userName").is("张三");
        Criteria criteriaPassWord = Criteria.where("passWord").is("123456");
        Criteria criteria = new Criteria().andOperator(criteriaUserName, criteriaPassWord);
        // 从第一行开始，查询2条数据返回, 并排序
        Query query = new Query(criteria)
                .with(Sort.by("createTime"))
                .limit(2)
                .skip(1);
        List<Person> result = mongoTemplate.find(query, Person.class);
        System.out.println("查询数量：" + result.size());
    }


    // 插入
    @Test
	public void insert() throws Exception{
        Person person =new Person();
        person.setId(20l)
                .setUserName("张三")
                .setPassWord("123456")
                .setCreateTime(LocalDateTime.now().toString());
        // mongoTemplate.insert(person);
        mongoTemplate.insert(person, "persons");
    }


    // 更新
    @Test
    public void updateMany() throws Exception{
        // 查询条件
        Criteria criteriaId = Criteria.where("id").is(4l);
        Query query= new Query(criteriaId);
        // 更新条件
        Update update= new Update();
        update.set("userName", "张三").set("passWord", "123456");;
        // 更新查询满足条件的文档数据（全部）
        UpdateResult result = mongoTemplate.updateMulti(query, update, Person.class);
        if(result!=null){
            System.out.println("更新条数：" + result.getMatchedCount());
        }
    }


    // 删除
    @Test
    public void remove() throws Exception{
        // 删除条件
        Criteria criteriaName = Criteria.where("userName").is("张三");
        Query query = new Query(criteriaName);
        DeleteResult result = mongoTemplate.remove(query, Person.class);
        System.out.println("删除条数：" + result.getDeletedCount());
    }
    
}
```











