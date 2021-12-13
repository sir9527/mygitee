

mysql中with聚合数据



1.新建表

```sql
表1：test01
CREATE TABLE `test01` (
  `id` int NOT NULL,
  `username` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  `valuetest1` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci

INSERT INTO `test02`(`id`, `username`, `valuetest1`) VALUES (1, 'xujiaqi', '001');
```

```sql
表2：test02
CREATE TABLE `test01` (
  `id` int NOT NULL,
  `username` varchar(255) COLLATE utf8mb4_general_ci DEFAULT NULL,
  `valuetest2` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci

INSERT INTO `test02`(`id`, `username`, `valuetest2`) VALUES (1, 'xujiaqi', '002');
```



2.with聚合数据

```sql
WITH 
value001 as (SELECT username,`valuetest1` FROM `test01` where id = 1),
value002 as (SELECT username,`testvalue2` FROM `test02` where id = 1)
SELECT valuetest1,testvalue2 
FROM value001 A,value002 B 
where A.username = B.username
```

```sql
查询结果：valuetest1=001，testvalue2=002
```

