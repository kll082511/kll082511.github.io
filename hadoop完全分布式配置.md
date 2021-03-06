﻿### hadoop完全分布式的创建
#### 1.xshell上的发送键输入的所有会话的功能的使用
- ①创建三个新的虚拟机,全部用xsheel打开,三个虚拟机都要创建hadoop用户,且在其根目录下,放入jdk和hadoop压缩包.
- ②右键,选择发送键输入的所有会话,在其中一个虚拟机中解压jdkhadop.其他两个虚拟机会自动同步.
- ③配置好jdk和hadoop环境

##### ④使用scp命令把之前配好的伪分布式的5个.xml文件的内容传送到现在的三个新建虚拟机中.

```
格式:scp 要传输的内容 要传到的电脑的用户名@要传到的电脑的IP:要传到的这台电脑的位置
例如:
scp -r /home/tars/hadoop-2.7.3/etc/hadoop hadoop@172.18.24.190:/home/tars/hadoop-2.7.3/etc/hadoop
要传的内容是之前伪分布式配置好的hadoop解压包下的etc文件夹下的hadoop文件夹
要传到的是hadoop用户,ip是172.18.24.190.
要注意权限,如果权限都是root,使用chown -R hadoop:hadoop /home/tars/hadoop-2.7.3/etc/hadoop命令把两个hadoop文件权限全部改成hadoop用户,或者使用su权限
重复命令,注意ip地址,把剩余两台虚拟机的hadoop的文件替换掉.
```

###### 注意:新建的三个虚拟机是当做datanode使用的,之前创建的伪分布式当做namedata,以上操作其实主要是jdk和hadoop环境配置,以及把namenode同步配置在datanode上.这里就不需要进行格式化了,格式化是针对namenode进行的
#### 2.在hadoop解压包的etc下的hadoop文件夹下的slaves文件里加入新建的三个虚拟机的IP地址(是为把datanode写进去进行注册)
#### 3.免密操作(都是在namenode虚拟机上操作)
- ①注册命令是:ssh-keygen -t dsa(生成公钥和私钥文件,遇到写密码的地方直接回车,是为了在启动关闭时不需要密码)
- ②执行命令:ssh-copy-id hadoop@172.18.24.190 (是把namenode公钥拷贝给datanode,使namenode可以免密操作datanode,此语句是为了和datanode建立关系,匹配秘钥.语句执行三次,每次的IP地址写新建的三个虚拟机的地址)
- ③执行名:cat .ssh/id_dsa.pub >> ~/.ssh/authorized_keys(将公钥追加到自己的 authorized_keys文件上,namenode也可以免密)
- ④查看正常加入的datanode的命令:hdfs dfsadmin -report

#### 3.在网页上登录namenode,发现datanode没有正常加入到namenode上的解决方法
- 找到datanode虚拟机的hadoop的解压包下的logs文件夹下的namenode的log文件打开,会发现hostname的错误.这时,在namenode的/etc/hosts文件中添加三个datanode和namenode虚拟机的ip和对应给他取个名字即可.
	- 例如: 172.18.24.190  datanode1
	- 172.18.24.28 namenode

### 以下是设置yarn-site.xml文件的代码:是为了让namenode可以识别ResourceManager.
**注意:下面的namenodes是namenode的主机名,所以记得要把namenode和datanode的主机名改了.**
```
<property>

    <name>yarn.resourcemanager.address</name>
    <value>namenodes:8032</value>

  </property>

  <property>

    <name>yarn.resourcemanager.scheduler.address</name>

    <value>namenodes:8030</value>

  </property>

  <property>

    <name>yarn.resourcemanager.resource-tracker.address</name>

    <value>namenodes:8031</value>

  </property>
```
