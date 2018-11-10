#### ����

```
Configuration conf = new Configuration();
// �����û�,����Ҫ�õ��û���˭
//System.setProperty("HADOOP_USER_NAME", "hadoop");
// ����Ҫʹ�õ��ļ�ϵͳ��hdfs ->��ַ�� 172.18.24.28:9000
//conf.set("fs.defaultFS",  "hdfs://172.18.24.28:9000");
//fileSystem = FileSystem.get(conf);
fileSystem = FileSystem.get(new URI("hdfs://172.18.24.28:9000"), conf, "hadoop");
```

#### ����

- 1.�ϴ�����

```
// 1.�����ļ�·���ǵø���б��
// 2.hdfs�ϵ��ļ�·��,ע���ļ����Ƿ����,����/kll,����ļ��д���,�ͻ��ϴ������ļ�����;���������,�ͻ��Ը��ļ����ԭ�ļ�����µĴ���.
fileSystem.copyFromLocalFile(new Path("G:/MapReduce/input/things.txt"), new Path("/list"));
```

- 2.ɾ������

```
// ɾ��������һ����ʽ�Ѿ�����
// �����еĵڶ���������ʾ�Ƿ�Ҫʹ�õݹ����,������ɾ��һ���ļ�ûʲôӰ��,�������Ҫɾ�������ļ���,ֻҪ���ǿ��ļ���,�ͱ������ó�true,����ݹ������������ɾ��.
fileSystem.delete(new Path("/ccc/ll.md"), true);
```

- 3.�����ϴ�

```
// input �����ݶ���
FileInputStream fis = new FileInputStream("C:/Users/Administrator/Desktop/kk/stream.txt");
// �����ݶ���
FSDataOutputStream fsDataOutputStream = fileSystem.create(new Path("/list/in.txt"));
IOUtils.copyBytes(fis, fsDataOutputStream, 1024);
```

- 4.��������

```
FSDataInputStream fsDataInputStream = fileSystem.open(new Path("/list/in.txt"));
FileOutputStream foStream = new FileOutputStream("C:/Users/Administrator/Desktop/ll/ll.txt");
org.apache.commons.compress.utils.IOUtils.copy(fsDataInputStream, foStream);
```