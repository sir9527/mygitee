

```

-- 原文链接
https://www.jianshu.com/p/20a630706354

```



Tomcat启动调试模式

```

./catalina.sh jpda start

注：启动调试模式,默认端口8000
```



修改默认端口

```

-- 修改 tomcat /bin/catalina.sh文件中的8000端口,修改为自己想要的.
-- 修改完后需要重启调试模式
#   JPDA_ADDRESS    (Optional) Java runtime options used when the "jpda start"
#                   command is executed. The default is 8000.
if [ -z "$JPDA_ADDRESS" ]; then
    JPDA_ADDRESS="8000"
fi

```



IDEA连接调试模式

```

选择菜单:
Run >>> Edit Configurations...

添加远程调试:
第一步:点击"+"号
第二步:点击Remote
第三步:输入 Host 和 Post
第四步: 点击 OK
第五步:启动Remote.完成调试模式连接

```





![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210528180451.png)


