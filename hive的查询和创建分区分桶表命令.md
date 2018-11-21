### 连接hive
- 1.在hive搭建完成后,执行hive解压包下的bin文件夹下的hiveserver2命令之后,处于等待状态
- 2.然后选中虚拟机的会话框,右键,选中复制会话,在新的会话中,输入命令beeline
- 3.接着输入命令[!connect jdbc:hive2://localhost:10000],回车.(如果mysql在本台服务器上,就填localhost,没在,就填在的那台ip)
- 4.填写当前服务武器登录的用户名,回车.
- 5.密码不填,直接回车.
- 6.结束,可以创建表和数据库.
#### 数据库的创建: create 库名;
### 一.表的创建和查询
**清空表数据:**truncate table 表名;
**删除表:**drop 表名;
**使用数据库:**use 库名;
- 1.内部表创建:
```
create table user(id int,name string) row format delimited fields terminated by "\t";[引号内填的是按什么格式把数据导入到表中]
```

- 2.外部表创建:
外部表删除之后,hdfs上创建的表数据还存在,只是断掉了与该表的连接.
```
create external table user(id int,name string) row format delimited fields terminated by "\t";
```

- 3.分区表创建:
```
create table user(id int,name string)
partitioned by(year string,month string)";[分区可以不止一层,且是表创建时的不存在字段]
row format delimited fields terminated by "\t
```

- 4.分桶表创建:
```
create table order_lin(orderId int,orderNo string,goodsId int,times string)
clustered by (goodsId) into 3 buckets[根据需求进行分桶,此处是根据商品id进行分区]
row format delimited fields terminated by '\t';
```

- **5.array数组类型建表和contains方法**
将以下数据导入表中:
战狼2,吴京:吴刚:余男,2017-08-16
三生三世十里桃花,刘亦菲:痒痒,2017-08-20
羞羞的铁拳,沈腾:玛丽:艾伦,2017-12-20
```
create table movies(moive_name string,actors array<string>,first_show date)
row format delimited fields terminated by ','
collection items terminated by ':';
```

需求:查出有吴刚演的电视
**查询命令:**
```
select moive_name,actors from t_movie where array_contains(actors,'吴刚');
```

- **6.map类型建表和查询**
需求:把以下数据放在表中
1,zhangsan,father:xiaoming#mother:xiaohuang#brother:xiaoxu,28
2,lisi,father:mayun#mother:huangyi#brother:guanyu,22
3,wangwu,father:wangjianlin#mother:ruhua#sister:jingtian,29
4,mayun,father:mayongzhen#mother:angelababy,26
```
create table maps(id int,name string,members map<string,string>,age int)
row format delimited fields terminated by ','
collection items terminated by '#'
map keys terminated by ':';
```

**查询命令:**
```
## 取map字段的指定key的值
select id,name,members['father'] as father from maps;
## 取map字段的所有key
select id,name,map_keys(members) as relation from  maps;
## 取map字段的所有value
select id,name,map_values(members) from t_person;
## 取map字段指定value
select id,name,map_values(members)[0] from maps;
```

- 7.查看当前所在数据库的位置:select current_database();
- 8.查看分区:show partitions 表名;
- **9.条件查询:**
**if语句**
select name,if(array_contains(actors,'刘亦菲'),'look','no look') from arrs;

**case语句**
```
select name,salary,
case
when salary <= 10000 then '潜力股'
when salary < 100000 then '小子'
when salary >= 100000 then '666'
end st
from company_emp
```

####　1.综合案例分析:创建分区分桶表,导入数据
需求:创建分区分桶,把以下数据导入到分区分桶表中,数据要求自动分区分桶
数据是按订单编号,订单名称,商品编号,时间
101     order123        1       2015-10-01
102     order123        2       2015-10-01
103     order125        5       2016-06-02
104     order126        9       2017-08-03
105     order127        3       2018-09-04
106     order128        4       2016-11-05
107     order129        6       2017-01-06
108     order130        7       2015-03-07
109     order131        8       2916-09-08
```
1.创建普通表
create table order_lin(orderId int,orderNo string,goodsId int,times string)
 row format delimited fields terminated by '\t';
2.创建分区分桶表
create table partbucket(orderId int,orderNo string,goodsId int,times date)
// 添加分区
partitioned by (year string,month string)
// 添加分桶
clustered by (goodsId) into 3 buckets
row format delimited fields terminated by '\t';
// 设置动态分区
set hive.exec.dynamic.partition.mode=nonstrict;
// 设置分桶状态
set hive.enforce.bucketing=true;


3.普通表数据导入到分区分桶表中
insert into partbucket
partition(year,month)
select *,year(times),month(times) from order_lin;
```

####　2.综合案例分析:展示连续三天有销售额的店铺
需求数据如下:店铺名,时间,销量
a,2017-02-10,600
b,2017-02-05,200
b,2017-02-06,300
b,2017-02-08,200
b,2017-02-09,400
b,2017-02-10,600
c,2017-01-31,200
c,2017-02-01,300
c,2017-02-02,200
c,2017-02-03,400
c,2017-02-10,600
a,2017-03-01,200
a,2017-03-02,300
a,2017-03-03,200
a,2017-03-04,400
a,2017-03-05,600

1.根据店铺进行分区,根据时间进行升序排序,生成行号
```
select *,row_number() over(partition by name order by times) as row_num from shops
```

2.将得到的临时表中的times-row_num得到的差值,如果相同表示是连续的
因为是要从上一次的查询结果的临时表中取数据,所以需要把上次的临时表作为这次要查询的表来使用
```
select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable
```

3.将查询到的临时表再作为要进行查询的表,从这个表中,我们以店铺名和减出来的结果进行分组(group)
从分组的结果中取出店铺名和分组的记录的个数,也得到了这个店铺连续销售了几天.
```
select name,count(*) as res_count
from
(select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable) as secondTable
group by diff,name
```

4.根据需求要将结果中连续销售三天及以上的店铺进行展示,所以要对分组的结果进行条件筛选
```
select name,count(*) as res_count
from
(select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable) as secondTable
group by diff,name having res_count >= 3;
```

### 二.数据的导入导出
- 1.本地数据导入到表中:
```
load data local inpath '文件在当前运行的虚拟机上的路径' into table 表名;
```

- 2.hdfs的数据导入表中:
```
load data inpath '文件在hdfs上的路径' into table 表名;
```

- 3.将hive表中的数据导入到本地:
```
insert overwrite local directory '/hadoop/shop'
row format delimited fields terminated by ' '
select * from shops limit 100000;
```

##### limit 100000可以不要,是读取前100000行的意思.
- 4.将hive表中的数据导入到hdfs上:
```
insert overwrite directory '/dbdata/shop'
row format delimited fields terminated by ','[导出的表以逗号隔开]
select * from shops;
```

### 三.修改表定义
- 1.修改表名:alter table table_name rename to new_table_name;
- 2.修改分区名:alter table t_partition partition(department='xiangsheng',sex='male',howold=20) rename to partition(department='1',sex='1',howold=20);
- 3.添加分区:alter table t_partition add partition (department='2',sex='0',howold=40); 
- 4.删除分区:alter table t_partition drop partition (department='2',sex='2',howold=24); 
- 5.修改表的文件格式定义:ALTER TABLE table_name [PARTITION partitionSpec] SET FILEFORMAT file_format;
- 6.修改表的某个文件格式定义:alter table t_partition partition(department='2',sex='0',howold=40 ) set fileformat sequencefile;
- 7.修改列名定义:ALTER TABLE table_name CHANGE col_old_name col_new_name column_type first;[把修改后的列表名放在最前面一列,也可以使用after 字段名,使其在某个字段后]
- 8.增加列:alter table t_user add columns (sex string,addr string);
- 9.替换列:alter table t_user replace columns (id string,age int,price float);[原有的全被现有的替换掉]
### 四.函数
- 1.数组排序函数:select sort_array(array(3,7,5));
- 2.获取数组长度:select size(array(3,7,5,9));

- 3.获取表中有姐妹的一列:
```
select * from maps where
array_contains(map_keys(relationship),'sister')
```

- 4.count()函数:括号内填数字代表数据会依次累加1,取到统计的数据,填NULL,则会统计直接为0.
select count(if(salary > 20000,1,null)) from company_emp 
- 5.窗口函数之一:row_number() over()
通过partition by的属性 对总属性分区,通过order by对每个分区单独排序并生成一列序号 ,分组TOPN[可以直接在普通表中操作]
```
例如:select *,row_number() over(partition by gender order by age desc) from topn;
```

**查询性别中,年龄最高的两个人**
```
例如:
select * from
(select *,row_number() over(partition by gender order by age desc) as row_num from topn) as res
where row_num<=2;
```

lateral view
explode


