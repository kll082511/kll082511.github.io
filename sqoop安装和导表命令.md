**��װ:**
ֻ��Ҫ��mysql-connector-java-5.1.34���뵽lib�ļ����¾Ϳ�����

**sqoop����֮import**
�ӹ�ϵ�����ݿ⵼�����ݵ�hdfs��(��Ϊδָ��Ҫ��ŵ�hdfs���Ķ���Ŀ¼��,����Ĭ�ϻ�����/user/�û���/������)

\ �Ǳ�ʾ�س��ĺ���
-m 1�Ǳ�ʾֻ��һ��map���� 
bin/sqoop/ import /
--connect jdbs:mysql://���ݿ����̨����IP��ַ:3306/���ݿ��� \
--username root \
--password kll \
--table ���� --m 1

�������ʹ��Ĭ��·��,��Ҫ�Զ����·��,��Ҫ��������--target-dir ��Ҫ��ŵ�·��(Ҫ�ǲ����ڵ�·��)
./sqoop import \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root --password kll \
--target-dir /sqoopTable/users \
--table users --m 1


./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--target-dir /sqoopTable/users1 --table users --fields-terminated-by '\t' --m 1


./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--target-dir /sqoopTable/users2 --table users --fields-terminated-by '\t' --m 1 --columns name,pwd



hive�е�������(��û��ָ���浽�Ķ������ݿ���Ķ������е�ʱ��,��Ĭ����default���ݿ��д��,�������ǹ�ϵ���ݿ��еı���)
 ./sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll --table users --m 1 \
--hive-import --target-dir /sqoopTable/hive1

hive�д浽ָ�����ݿ����:--hive-table klljdbc.db.klluser1
�����Թ�ϵ�����ݿ��е����ݽ��в�ѯ֮��,�ٵ��뵽hdfs��,ʹ��:--query 'sql��� where [id>3 and $CONDITIONS'
���ʹ����--queryҪ��������ݾ������ǲ�ѯ�����Ľ������,�Ͳ�Ҫ����--table��.����where���ͱ�����$CONDITIONS.
bin/sqoop import --connect jdbc:mysql://172.18.24.28:3306/kllsqoop --username root --password kll \
--m 1 -query 'select id,name from users where $CONDITIONS' --target-dir /user/hadoop/users



��������[����idΪ5�Ժ������]
bin/sqoop import \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root \
--password kll \
--table users --m 1 \
--incremental append \
--check-column id \
--last-value 5

**sqoop����֮export**
bin/sqoop export \
--connect jdbc:mysql://172.18.24.28:3306/kllsqoop \
--username root \
--password kll \
--table lluser \
--export-dir /user/hadoop/users/


