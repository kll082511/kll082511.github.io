# Hadoopα�ֲ�ʽ
#### 1.�����û�
#### 2.ʹ���´������û����ӵ������
#### 3.����ǽ�ر� 
- (1)systemctl disable firewalld 
- (2)systemctl status firewalld
#### 4.JDK
- (1)����
- (2)�ϴ�
- (3)��ѹ������
- (4)���û�������
#### 5.hadoop����
- (1)����hadoop��
- (2)�ϴ�
- (3)��ѹ,����
- (4)���û�������(bin\sbin)
#### 6.����hadoop�м����ļ�
- (1)hadoop-env.sh 
	- ��($JAVA_HOME·�����ó�JDK��ѹ����·��)
- (2)core-site.xml
	- ��fs.defaultFS��
		- 1)Ĭ���Ǳ����ļ�ϵͳfile:///
		- 2)ʹ��URLͳһ��Դ��λ���ĸ�ʽ���ó�hdfs://������ip��ַ(ʵ����namenode��ip):9000
			- ��ȣ�jdbc:mysql://localhost:3306
	- ��hadoop.tmp.dir:���hadoop��ʱ�ļ����ļ�Ŀ¼
	- �۴�������(�Ƿ���<configuration>��ǩ�ڵ�):
	<property>??
????????<name>hadoop.tmp.dir</name>?//�����ʱ�ļ�?
????????<value>/home/hadoop/tmp</value>// ���·��
????</property>??
????<property>??
????????<name>fs.defaultFS</name>??
????????<value>hdfs://172.18.24.28:9000</value>??
????</property>? 
- (3)hdfs-site.xml:
	- ��dfs.namenode.name.dir��namenode����·��(��·����Ҫ����ǰ��¼���û�����д��Ȩ��)
	- ��dfs.datanode.data.dir��datanode����·��(��·����Ҫ����ǰ��¼���û�����д��Ȩ��)
	- �۴�������:
	<property>????
????????<name>dfs.replication</name>//���ñ���(����ʱɾ��)????
????????<value>1</value>????
????</property>????
????<property>????
????????<name>hdfs.namenode.name.dir</name>????
????????<value>file:/home/hadoop/dfs/name</value>????
????</property>????
????<property>????
????????<name>dfs.datanode.data.dir</name>????
????????<value>file:/home/hadoop/dfs/data</value>????
????</property>
- (4)yarn-site.xml
	- ��yarn.nodemanager.aux-services��ʹ��mapreduce_shuffle����
	- �ڴ�������:
	<property>??
		<name>mapreduce.framework.name</name>??
		<value>yarn</value>??
	</property>??
	<property>??
		<name>yarn.nodemanager.aux-services</name>??
		<value>mapreduce_shuffle</value>??
	</property>
- (5)Mapred-site.xml��cp Mapred-site.xml.templete��
	- ��Mapreduce.framework.name��yarn
	- �ڴ�������:
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property> 
#### 7.��ʽ��HDFS
- (1)Hadoop  namenode  -format
	- ��ʽ���ɹ��ı�־:Storage:.....successfully 
#### 8.����HADOOP����
- (1)start-all.sh����start-dfs.sh and start-yarn.sh
	- �ɹ��ı�־��:����jps������ֵ�����ͼ:

#### 9.�ر�hadoop����
- (1)Stop-all.sh���� stop-yarn.sh and stop-dfs.sh
