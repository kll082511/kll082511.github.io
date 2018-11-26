**安装:**
只需要把mysql-connector-java-5.1.34导入到lib文件夹下就可以了
**sqoop命令之import**
从关系型数据库导入数据到hdfs上(因为未指定要存放到hdfs的哪儿个目录下,所以默认会存放在/user/用户名/表名下)
\ 是表示回车的含义
-m 1是表示只开一个map工作 
bin/sqoop/ import /
--connect jdbs:mysql://数据库的那台电脑IP地址:3306/数据库名 \
--username root \
--password kll \
--table 表名 --m 1
如果不想使用默认路径,想要自定义的路径,就要增加配置--target-dir 想要存放的路径(要是不存在的路径)
```
./sqoop import \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root --password kll \
--target-dir /sqoopTable/users \
--table users --m 1
```

```
./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--target-dir /sqoopTable/users1 --table users --fields-terminated-by '\t' --m 1
```

```
./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--target-dir /sqoopTable/users2 --table users --fields-terminated-by '\t' --m 1 --columns name,pwd
```

hive中导入数据(在没有指定存到哪儿个数据库的哪儿个表中的时候,会默认在default数据库中存放,表名就是关系数据库中的表名)
```
 ./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll --table users --m 1 \
--hive-import --target-dir /sqoopTable/hive1
```

hive中存到指定数据库表中:--hive-table klljdbc.db.klluser1
如果想对关系型数据库中的数据进行查询之后,再导入到hdfs上,使用:--query 'sql语句 where (id>3 and $CONDITIONS)'
如果使用了--query要导入的数据就是我们查询出来的结果集了,就不要设置--table了.且有where语句就必须有$CONDITIONS.
```
bin/sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--m 1 -query 'select id,name from users where $CONDITIONS' --target-dir /user/hadoop/users
```

增量导入(导入id为5以后的数据)
bin/sqoop import \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root \
--password kll \
--table users --m 1 \
--incremental append \
--check-column id \
--last-value 5

**sqoop命令之export**
bin/sqoop export \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root \
--password kll \
--table lluser \
--export-dir /user/hadoop/users/


