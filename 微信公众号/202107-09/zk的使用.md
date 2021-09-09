

* 1.依赖
* 2.配置文件
* 3.常量类
* 4.zk序列化
* 5.zk工具类
* 6.Controller层
* 7.Service层



### 1.依赖

```java
<!--zkclient-->
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
<!-- 引入zookeeper -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.12.0</version>
</dependency>
<!--json-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.73</version>
</dependency>
<!--常用工具类-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<optional>true</optional>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



### 2.配置文件

```
server.port=8088
        
zookeeper.connect_str=127.0.0.1:2181
zookeeper.timeout=2000
```


### 3.常量类

```java
public class ZkStringConstant {

    public static final String ROOT_PATH = "/aiEngine";
    public static final String BACK_SLASH = "/";
}
```



### 4.zk序列化

```java
import com.alibaba.fastjson.JSONObject;
import org.I0Itec.zkclient.exception.ZkMarshallingError;
import org.I0Itec.zkclient.serialize.ZkSerializer;
import java.io.*;

public class ZkSerializerImpl implements ZkSerializer {

    String charset = "UTF-8";

    /**
     * 序列化：将对象转成json字符串后在转成bytes传到zk节点上
     */
    @Override
    public byte[] serialize(Object o) throws ZkMarshallingError{
        try {
            return JSONObject.toJSONString(o).getBytes(charset);
        } catch (UnsupportedEncodingException e) {
            throw new ZkMarshallingError(e);
        }
    }


    /**
     * 反序列化：将zk节点上的bytes数据转成json字符串，在转成json对象
     */
    @Override
    public Object deserialize(byte[] bytes) throws ZkMarshallingError {
        try {
            return JSONObject.parse(new String(bytes, charset));
        } catch (UnsupportedEncodingException e) {
            throw new ZkMarshallingError(e);
        }
    }
}

```



### 5.zk工具类

```java
import org.I0Itec.zkclient.ZkClient;
import org.apache.zookeeper.CreateMode;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;
import javax.annotation.PostConstruct;
import java.util.List;
import static com.example.demo8088.utils.ZkStringConstant.BACK_SLASH;
import static com.example.demo8088.utils.ZkStringConstant.ROOT_PATH;


@Component
public class ZkUtils {

    @Value("${zookeeper.connect_str}")
    private String zkServer;

    @Value("${zookeeper.timeout}")
    private int timeout;

    private static ZkClient zkClient;

    private ZkUtils() {
    }

    /**
     * @PostConstruct：Bean创建完成的时候，会后置执行@PostConstruct修饰的方法
     */
    @PostConstruct
    private void init(){
        ZkUtils.zkClient = new ZkClient(zkServer, timeout);
        // 设置zk序列化
        zkClient.setZkSerializer(new ZkSerializerImpl());
        if (!zkClient.exists(ROOT_PATH)){
            zkClient.createPersistent(ROOT_PATH);
        }
    }


    /**
     * 列出指定path下所有子节点
     */
    public static void getAllChildPath(String path, List<String> pathList){
        List<String> childrenList = zkClient.getChildren(path);
        if(CollectionUtils.isEmpty(childrenList)){
            return;
        }
        for(String childStr : childrenList){
            StringBuilder sBuilder = new StringBuilder();
            String tmpPath = sBuilder.append(path)
                    .append(BACK_SLASH)
                    .append(childStr)
                    .toString();
            List<String> tmpChildren = zkClient.getChildren(tmpPath);
            if (CollectionUtils.isEmpty(tmpChildren)){
                pathList.add(tmpPath);
            }else {
                getAllChildPath(tmpPath, pathList);
            }
        }
    }

    /**
     * 创建节点
     */
    public static String create(String servName, String version, Object content) {
        StringBuilder path = new StringBuilder();
        path.append(ROOT_PATH).append(BACK_SLASH).append(servName);
        if (!zkClient.exists(path.toString())) {
            zkClient.createPersistent(path.toString());
        }
        path.append(BACK_SLASH).append(version);
        if (zkClient.exists(path.toString())) {
            zkClient.delete(path.toString());
        }
        return zkClient.create(path.toString(), content, CreateMode.PERSISTENT);
    }


    /**
     * 传入一个path，返回一个对象
     */
    public static Object getObjectByPath(String path){
        return zkClient.readData(path);
    }
}
```



### 6.Controller层

```java
@RestController
public class Test {

    @Autowired
    private TestService testService;

    @GetMapping("/demo")
    public void demo(){
        testService.addNode();
    }
}
```


### 7.Service层

```java
import com.alibaba.fastjson.JSONObject;
import lombok.Data;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;
import static com.example.demo8088.utils.ZkStringConstant.ROOT_PATH;


@Service
public class TestService {

    public void addNode(){
        // 增加节点
        Manager manager = new Manager();
        manager.setSerName("product001");
        manager.setSerVersion("v1.0");
        ZkUtils.create("product001","v1.0", manager);

        // 获取节点中的对象数据
        List<String> path = new ArrayList<>();
        ZkUtils.getAllChildPath(ROOT_PATH, path);
        for (int i = 0; i < path.size(); i++) {
            JSONObject jsonData = (JSONObject)ZkUtils.getObjectByPath(path.get(i));
            Manager man = jsonData.toJavaObject(Manager.class);
        }
    }


    @Data
    public static class Student{
        private String name;
    }

    @Data
    public static class Manager{
        private String serName;
        private String serVersion;
    }


}
```











