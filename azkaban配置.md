**azkaban配置**

azkaban-executor-server-2.5.0.tar.gz	存放了执行器
azkaban-sql-script-2.5.0.tar.gz		存放了web服务器
azkaban-web-server-2.5.0.tar.gz		存放azkaban运行需要的sql(sql倒完了这个文件就没用了)
create-all-sql-2.5.0.sql	完整的sql文件,只执行它就可以了


ssl访问		


azkaban-executor-2.5.0		启动,关闭执行器  start/shutdown
azkaban.properties		设置时区,设置mysql的信息

azkaban-web2.5.0	启动,关闭web服务器 start/shutdown
azkaban.properties	设置时区,设置mysql的信息,jetty密码等同于sshl的密码
azkaban-user.xml	设置登录按照kaban的用户及其权限


### 一.配置SSL.
- 1.使用:mysql -u root -p命令进入数据库.
- 2.创建数据库:create azkaban;
- 3.进入数据库:use azkaban;
- 4.执行命令:source /home/hadoop/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql
- 5.输入命令:keytool -keystore keystore -alias jetty -genkey -keyalg RSA
先输入密码,其他直接回车,中间输一次y,进行确认,接着直接回车即可.
- 6.把证书keystore(在执行5命令的目录下)拷贝到azkaban-web-2.5.0的文件夹下:cp keystore azkaban-web-2.5.0/
- 7.修改azkaban-web-2.5.0文件.
	- 7.1修改文件:conf/azkaban.properties ,修改如下:
	- 7.2.修改conf/azkaban-users.xml文件,加入:<user username="admin" password="admin" roles="admin,metrics" />
```
文件里的时区修改成如下:
default.timezone.id=Asia/Shanghai
mysql.host=localhost修改成ip[意味着远程连接]
mysql.database=azkaban[写自己新建的数据库名]
把下面的用户名改为root用户,和数据库密码.
在jetty的设置中,把所有的keystore的名字写成之前生成的keystore名,端口不用修改,
所有密码设置成在第五步填的密码.
```

- 8.修该azkaban-executor-2.5.0文件夹
	- 8.1修改conf/azkaban.properties文件,里的时区修改的和上边的一样,还有数据库名和密码用户.
