### hive�����Ĵ
1.�Ȱ�hive��ѹ,Ȼ�����������/etc/profile�ļ�������hive��������.���úú�,�ǵ����д��ļ�.
```
export HIVE_HOME=hive��ѹ��·��
��path����������� :$HIVE_HOME/bin
```

2.��hive�ļ����µ�lib�ļ����ﴫ��mysql-connector-java-5.1.34.jar��
3.��hive��ѹ���µ�conf�ļ����´���hive-site.xml(���û�еĻ�����)
4.�ڸ��ļ��������������
```
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://namenode��ip��ַ:3306/hivedb?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>���ݿ������</value>
	</property>
</configuration>
```

6.����hive�鿴�Ƿ�װ����.
### hive��mysql���ӵĲ�������.
###### sql��������������(��˳��ִ��):
```
// ��װwget������
yum -y install wget
// ������ַ
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
sudo yum -y install mysql
sudo yum -y install mysql-server
```

�鿴sql��װ��ɵ���������:
```
// ��������
systemctl start mysqld
// �鿴״̬,��running����ɹ�
systemctl status mysqld
// �ر�����
systemctl stop mysqld
```

###### mysql�������޸�����
```
// �޸����������(��Ҫ��user�û����޸�,������ mysql -u root -p��¼mysql)
UPDATE mysql.user SET authentication_string=PASSWORD('����Ҫ��hive-site.xml�ļ��ڵ�һֱ') where USER='root';
// �޸������Ҫˢ��һ��
flush privileges;
// �˳���,Ҫ����mysql
systemctl restart mysqld
```

- Ϊ�˱�֤����hive��ʹ��mysql,��Ҫ����Щ����֮��,������������,��������
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '���ݿ�����' WITH GRANT OPTION;
ˢ������(����Ҫ���)
FLUSH PRIVILEGES
```

### mysql��������Ĳ���
1.��/etc/my.cnf�ļ������
skip-grant-tables
2.��Ȼ����mysql���������mysql
3.����UPDATE mysql.user SET authentication_string=PASSWORD('6757DUgu') where USER='root'�޸�����Ϊ6757DUgu
4.flush privilegesˢ��
5.����mysql.