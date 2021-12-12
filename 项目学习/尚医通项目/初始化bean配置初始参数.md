

```java
aliyun.sms.regionId=default
aliyun.sms.accessKeyId=LTAI4G8eqBXYDCXnrVLsaG
aliyun.sms.secret=rtkoybhDzG5FkC00UYNCoA30APeAcS
```



```java
@Component
public class ConstantPropertiesUtils implements InitializingBean {

    @Value("${aliyun.sms.regionId}")
    private String regionId;
    @Value("${aliyun.sms.accessKeyId}")
    private String accessKeyId;
    @Value("${aliyun.sms.secret}")
    private String secret;

    public static String REGION_Id;
    public static String ACCESS_KEY_ID;
    public static String SECRECT;

    @Override
    public void afterPropertiesSet() throws Exception {
        REGION_Id = regionId;
        ACCESS_KEY_ID = accessKeyId;
        SECRECT = secret;
    }
}
```

