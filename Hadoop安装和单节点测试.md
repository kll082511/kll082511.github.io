#### hadoop�İ�װ(ǰ����밲װ��JDK,�ļ�·����ʹ�õľ���·��,���Ը����Լ�����λ���ʵ�����)
- 1.��hadoop��װ�������������,��ѹ.�ҵ�����/home/hadoop/tars�ļ����·���,���»������Ϊ��.
- 2.��/etc/profile����  ~/.bash_profile�������hadoop��·��
```
export HADOOP_HOME=/home/hadoop/tars/hadoop-2.7.3 #hadoop��ѹ������·��
#��PATH�������������������
:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

	- ����֮��Ҫִ������: . ~/.bash_profile����. /etc/profile,ʹ֮��Ч,���õ��Ķ����ļ�,�����Ķ�������.
	- ������Ч��������:hadoop
- 3.����һ���ļ���,������:mkdir kllinput.
- 4.��kllinput�ļ����´���һ��kll.txt�ļ�,������:touch kll.txt
- 5.ʹ������:vi /home/hadoop/kllinput/kll.txt,�����ı��༭����,������������.

- 6.ִ������:hadoop jar /home/hadoop/tars/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /home/hadoop/kll/kll.txt /home/hadoop/output
	- �������������jar��hadoop-mapreduce-examples-2.7.3.jar���wordcount����ͳ��kll�ļ����µĵ��ʵĳ��ִ���.
	- �ǵùرշ���ǽ:systemctl disable firewalld ���ùرշ���ǽ;systemctl stop firewalld ��ʱ�رշ���ǽ,�´�����,��Ҫ�ٴιر�.
- 7.ִ������:cat output/*