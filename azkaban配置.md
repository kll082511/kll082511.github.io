**azkaban����**

azkaban-executor-server-2.5.0.tar.gz	�����ִ����
azkaban-sql-script-2.5.0.tar.gz		�����web������
azkaban-web-server-2.5.0.tar.gz		���azkaban������Ҫ��sql(sql����������ļ���û����)
create-all-sql-2.5.0.sql		������sql�ļ�,ִֻ�����Ϳ�����


ssl����				https://��װ��azkaban�ķ�������ip:8443


azkaban-executor-2.5.0 		����,�ر�ִ����  start/shutdown
azkaban.properties		����ʱ��,����mysql����Ϣ

azkaban-web2.5.0		����,�ر�web������ start/shutdown
azkaban.properties		����ʱ��,����mysql����Ϣ,jetty�����ͬ��sshl������
azkaban-user.xml 		���õ�¼����kaban���û�����Ȩ��


### һ.����SSL.
- 1.ʹ��:mysql -u root -p����������ݿ�.
- 2.�������ݿ�:create azkaban;
- 3.�������ݿ�:use azkaban;
- 4.ִ������:source /home/hadoop/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql
- 5.��������:keytool -keystore keystore -alias jetty -genkey -keyalg RSA
����������,����ֱ�ӻس�,�м���һ��y,����ȷ��,����ֱ�ӻس�����.
- 6.��֤��keystore(��ִ��5�����Ŀ¼��)������azkaban-web-2.5.0���ļ�����:cp keystore azkaban-web-2.5.0/
- 7.�޸�azkaban-web-2.5.0�ļ�.
	- 7.1�޸��ļ�:conf/azkaban.properties ,�޸�����:
```
�ļ����ʱ���޸ĳ�����:
default.timezone.id=Asia/Shanghai

mysql.host=localhost�޸ĳ�ip[��ζ��Զ������]
mysql.database=azkaban[д�Լ��½������ݿ���]
��������û�����Ϊroot�û�,�����ݿ�����.
��jetty��������,�����е�keystore������д��֮ǰ���ɵ�keystore��,�˿ڲ����޸�,
�����������ó��ڵ��岽�������.
```

	- 7.2.�޸�conf/azkaban-users.xml�ļ�,����:<user username="admin" password="admin" roles="admin,metrics" />
- 8.�޸�azkaban-executor-2.5.0�ļ���
	- 8.1�޸�conf/azkaban.properties�ļ�,���ʱ���޸ĵĺ��ϱߵ�һ��,�������ݿ����������û�.