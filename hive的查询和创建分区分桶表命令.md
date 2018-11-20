### ����hive
- 1.��hive���ɺ�,ִ��hive��ѹ���µ�bin�ļ����µ�hiveserver2����֮��,���ڵȴ�״̬
- 2.Ȼ��ѡ��������ĻỰ��,�Ҽ�,ѡ�и��ƻỰ,���µĻỰ��,��������beeline
- 3.������������[!connect jdbc:hive2://localhost:10000],�س�.(���mysql�ڱ�̨��������,����localhost,û��,�����ڵ���̨ip)
- 4.��д��ǰ����������¼���û���,�س�.
- 5.���벻��,ֱ�ӻس�.
- 6.����,���Դ���������ݿ�.
#### ���ݿ�Ĵ���: create ����;
### һ.��Ĵ����Ͳ�ѯ
**��ձ�����:**truncate table ����;
**ɾ����:**drop ����;
**ʹ�����ݿ�:**use ����;
- 1.�ڲ�����:
```
create table user(id int,name string) row format delimited fields terminated by "\t";[����������ǰ�ʲô��ʽ�����ݵ��뵽����]
```

- 2.�ⲿ����:
�ⲿ��ɾ��֮��,hdfs�ϴ����ı����ݻ�����,ֻ�Ƕϵ�����ñ������.
```
create external table user(id int,name string) row format delimited fields terminated by "\t";
```

- 3.��������:
```
create table user(id int,name string)
partitioned by(year string,month string)";[�������Բ�ֹһ��,���Ǳ���ʱ�Ĳ������ֶ�]
row format delimited fields terminated by "\t
```

- 4.��Ͱ����:
```
create table order_lin(orderId int,orderNo string,goodsId int,times string)
clustered by (goodsId) into 3 buckets[����������з�Ͱ,�˴��Ǹ�����Ʒid���з���]
row format delimited fields terminated by '\t';
```

- **5.array�������ͽ����contains����**
���������ݵ������:
ս��2,�⾩:���:����,2017-08-16
��������ʮ���һ�,�����:����,2017-08-20
���ߵ���ȭ,����:����:����,2017-12-20
```
create table movies(moive_name string,actors array<string>,first_show date)
row format delimited fields terminated by ','
collection items terminated by ':';
```

����:���������ݵĵ���
**��ѯ����:**
```
select moive_name,actors from t_movie where array_contains(actors,'���');
```

- **6.map���ͽ���Ͳ�ѯ**
����:���������ݷ��ڱ���
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

**��ѯ����:**
```
## ȡmap�ֶε�ָ��key��ֵ
select id,name,members['father'] as father from maps;
## ȡmap�ֶε�����key
select id,name,map_keys(members) as relation from  maps;
## ȡmap�ֶε�����value
select id,name,map_values(members) from t_person;
## ȡmap�ֶ�ָ��value
select id,name,map_values(members)[0] from maps;
```

- 7.�鿴��ǰ�������ݿ��λ��:select current_database();
- 8.�鿴����:show partitions ����;
- **9.������ѯ:**
**if���**
select name,if(array_contains(actors,'�����'),'look','no look') from arrs;

**case���**
```
select name,salary,
case
when salary <= 10000 then 'Ǳ����'
when salary < 100000 then 'С��'
when salary >= 100000 then '666'
end st
from company_emp
```

####��1.�ۺϰ�������:����������Ͱ��,��������
���󣺴���������Ͱ�����������ݵ��뵽������Ͱ����,����Ҫ���Զ�������Ͱ
�����ǰ�������ţ���������,��Ʒ���,ʱ��
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
1.������ͨ��
create table order_lin(orderId int,orderNo string,goodsId int,times string)
 row format delimited fields terminated by '\t';
2.����������Ͱ��
create table partbucket(orderId int,orderNo string,goodsId int,times date)
// ��ӷ���
partitioned by (year string,month string)
// ��ӷ�Ͱ
clustered by (goodsId) into 3 buckets
row format delimited fields terminated by '\t';
// ���ö�̬����
set hive.exec.dynamic.partition.mode=nonstrict;
// ���÷�Ͱ״̬
set hive.enforce.bucketing=true;


3.��ͨ�����ݵ��뵽������Ͱ����
insert into partbucket
partition(year,month)
select *,year(times),month(times) from order_lin;
```

####��2.�ۺϰ�������:չʾ�������������۶�ĵ���
������������:������,ʱ��,����
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

1.���ݵ��̽��з���,����ʱ�������������,�����к�
```
select *,row_number() over(partition by name order by times) as row_num from shops
```

2.���õ�����ʱ���е�times-row_num�õ��Ĳ�ֵ,�����ͬ��ʾ��������
��Ϊ��Ҫ����һ�εĲ�ѯ�������ʱ����ȡ����,������Ҫ���ϴε���ʱ����Ϊ���Ҫ��ѯ�ı���ʹ��
```
select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable
```

3.����ѯ������ʱ������ΪҪ���в�ѯ�ı�,���������,�����Ե������ͼ������Ľ�����з���(group)
�ӷ���Ľ����ȡ���������ͷ���ļ�¼�ĸ���,Ҳ�õ�������������������˼���.
```
select name,count(*) as res_count
from
(select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable) as secondTable
group by diff,name
```

4.��������Ҫ������������������켰���ϵĵ��̽���չʾ,����Ҫ�Է���Ľ����������ɸѡ
```
select name,count(*) as res_count
from
(select name,times,count,date_sub(times,row_num) as diff
from
(select *,row_number() over(partition by name order by times) as row_num from shops) as firstTable) as secondTable
group by diff,name having res_count >= 3;
```

### ��.���ݵĵ��뵼��
- 1.�������ݵ��뵽����:
```
load data local inpath '�ļ��ڵ�ǰ���е�������ϵ�·��' into table ����;
```

- 2.hdfs�����ݵ������:
```
load data inpath '�ļ���hdfs�ϵ�·��' into table ����;
```

- 3.��hive���е����ݵ��뵽����:
```
insert overwrite local directory '/hadoop/shop'
row format delimited fields terminated by ' '
select * from shops limit 100000;
```

##### limit 100000���Բ�Ҫ,�Ƕ�ȡǰ100000�е���˼.
- 4.��hive���е����ݵ��뵽hdfs��:
```
insert overwrite directory '/dbdata/shop'
row format delimited fields terminated by ','[�����ı��Զ��Ÿ���]
select * from shops;
```

### ��.�޸ı���
- 1.�޸ı���:alter table table_name rename to new_table_name;
- 2.�޸ķ�����:alter table t_partition partition(department='xiangsheng',sex='male',howold=20) rename to partition(department='1',sex='1',howold=20);
- 3.��ӷ���:alter table t_partition add partition (department='2',sex='0',howold=40); 
- 4.ɾ������:alter table t_partition drop partition (department='2',sex='2',howold=24); 
- 5.�޸ı���ļ���ʽ����:ALTER TABLE table_name [PARTITION partitionSpec] SET FILEFORMAT file_format;
- 6.�޸ı��ĳ���ļ���ʽ����:alter table t_partition partition(department='2',sex='0',howold=40 ) set fileformat sequencefile;
- 7.�޸���������:ALTER TABLE table_name CHANGE col_old_name col_new_name column_type first;[���޸ĺ���б���������ǰ��һ��,Ҳ����ʹ��after �ֶ���,ʹ����ĳ���ֶκ�]
- 8.������:alter table t_user add columns (sex string,addr string);
- 9.�滻��:alter table t_user replace columns (id string,age int,price float);[ԭ�е�ȫ�����е��滻��]
### ��.����
- 1.����������:select sort_array(array(3,7,5));
- 2.��ȡ���鳤��:select size(array(3,7,5,9));

- 3.��ȡ�����н��õ�һ��:
```
select * from maps where
array_contains(map_keys(relationship),'sister')
```

- 4.count()����:�����������ִ������ݻ������ۼ�1,ȡ��ͳ�Ƶ�����,��NULL,���ͳ��ֱ��Ϊ0.
select count(if(salary > 20000,1,null)) from company_emp 
- 5.���ں���֮һ:row_number() over()
ͨ��partition by������ �������Է���,ͨ��order by��ÿ������������������һ����� ,����TOPN[����ֱ������ͨ���в���]
```
����:select *,row_number() over(partition by gender order by age desc) from topn;
```

**��ѯ�Ա���,������ߵ�������**
```
����:
select * from
(select *,row_number() over(partition by gender order by age desc) as row_num from topn) as res
where row_num<=2;
```

lateral view
explode


