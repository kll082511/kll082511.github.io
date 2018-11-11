##### 1.���ʸ���ͳ��
map��
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class CountMapper extends Mapper<Object, Text, Text, IntWritable>{	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		// ��ȡ����һ�е����ݽ��зָ�
		String[] val = value.toString().split(" ");
		// ���������context
		for (int i = 0; i < val.length; i++) {
			// Ϊ�˰��ַ�ת��text����
			// ����text����
			Text text = new Text();
			// ��text��set����,���ַ�ת��text����
			text.set(val[i]);
			// ��int����ת��IntWritable����
			context.write(text, new IntWritable(1));
		}
	}
}
```

reduce��
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
	
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
		int sum = 0;
		for (IntWritable val : values) {
			sum += val.get();
		}
		context.write(key, new IntWritable(sum));
	}

}
```

main����
```
package com.lanou.test;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.examples.WordCount;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.Util.CountMapper;
import com.lanou.Util.WordReduce;

public class WordCountTest {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(WordCountTest.class);
		// s����mapper
		job.setMapperClass(CountMapper.class);
		// ����combiner
		job.setCombinerClass(WordReduce.class);
		// ����Reducer
		job.setReducerClass(WordReduce.class);
		// ����reduce��key��valueֵ������
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		// �����һ�����������,ֻ������
		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		// ����д�뵽���λ��.
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));

		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
```
##### 2.Partitioner_______��log��־�л�ȡ�ֻ���,��������,��������,������(��ʵ���������ʽ)
- ����:���ݷ���ֵ,�Ѳ�ͬ������ڲ�ͬ���ļ���
```
/*
 * ʹ�÷����Ĳ���:
 * 1.����job.setPartitionerClass(������.class);
 * ��job.setNumReduceTask(4),�����������,������쳣;
 * 2.����������,�̳�Partitioner��,ʵ��getPartitioner����
 * 3.�����Լ���ҵ���߼�,ͨ��key,value,count�Ĳ���,���÷���ֵ,�õ���ǰmap��Ҫ����ķ���.
 */
/*
 * ��log�л�ȡ�ֻ���,��������,��������,������
 * reduce֮��,������Ҫ�ֻ���,��������(��������,��������,�����ܺ�)
 * Ҫʹ���Զ����������Ϊ���
 * 1.�����������ǵ����ݽ��н�ģ(����ʵ����)
 * 2.mapper��reducer�е��β���ôд,map()����������ô�����ݽ����и�ɸѡ,�����ǵ����ݷ����Զ����bean(ע��javaBean�淶)����,���bean����key����value���
 * 3.map�Ľ������Ҫд�������,����Ҫע�������Զ������һ��Ҫ֧�����л�WritableComparable
 *    Serializable��java�ṩ�����������л�����hadoop�н���ʹ��hadoop�����ṩ��WritableComparable�ӿ���ʵ�����л��ͷ����л���
 *    Ҫע����дwrite()��readFields()���� ����reducer�׶λ�ȡ��������
 *    ע��:readFields()�е��õ�read����һ��Ҫ����д���˳����ж�ȡ�����������˳�����
 * 4.main�е�job.setJarByClass(BeanStreamTest.class);
        job.setMapperClass(BeanStreamMapper.class);
        job.setCombinerClass(BeanStreamReducer.class);
        job.setReducerClass(BeanStreamReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(StreamBean.class);
                    ע������
                    ע��������key,value������
 * 5.ע����дtoString����
 */
```

Partitioner��
```
package com.lanou.Util;

import java.util.HashMap;
import java.util.Map;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

import com.lanou.model.StreamBean;
/*
 * ʹ�÷����Ĳ���:
 * 1.����job.setPartitionerClass(������.class);
 * ��job.setNumReduceTask(4),�����������,������쳣;
 * 2.����������,�̳�Partitioner��,ʵ��getPartitioner����
 * 3.�����Լ���ҵ���߼�,ͨ��key,value,count�Ĳ���,���÷���ֵ,�õ���ǰmap��Ҫ����ķ���.
 */
public class MobilePartitioner extends Partitioner<Text, StreamBean>{

	private static Map map;
	// 1.��һ��������map�����е�context.write()���е�key
	// 2.�ڶ���������map�����е�context.write()���е�value
	// 3.������������reduceTask������
	// �������Ҫ��getPartitioner������ȥ����������map,�ᴴ��̫�Ը�����.
	// ��̫�������ǵ�����,����ֻϣ������һ�ξͺ�.
	static {
		map = new HashMap<>();
		map.put("134", 0);
		map.put("135", 1);
		map.put("136", 2);
	}
	@Override
	public int getPartition(Text paramKEY, StreamBean paramVALUE, int paramInt) {
		// TODO Auto-generated method stub
		// �����
		System.out.println("get 134 Partitioner:" + map.get("134"));
		System.out.println("get 135 Partitioner:" + map.get("135"));
		System.out.println("get 136 Partitioner:" + map.get("136"));
		System.out.println(paramVALUE);
		System.out.println(paramInt);
		String key = paramKEY.toString().substring(0, 3);
		// ����ֵ,�����������������Ľ����Ķ���reduceTask
		Integer result = (Integer) map.get(key);
		if (result == null) {
			result = 3;
		}
		return result;
	}

}

```

ʵ����
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class StreamBean implements WritableComparable<Object> {
	private int uploadStream;
	private int downloadStream;
	private int sumStream;
	public StreamBean() {
		super();
	}
	public StreamBean(String uploadStream, String downloadStream) {
		super();
		this.uploadStream = Integer.parseInt(uploadStream);
		this.downloadStream = Integer.parseInt(downloadStream);
		this.sumStream = this.downloadStream + this.uploadStream;
	}
	
	public StreamBean(int uploadStream, int downloadStream) {
		super();
		this.uploadStream = uploadStream;
		this.downloadStream = downloadStream;
		this.sumStream = this.downloadStream + this.uploadStream;
	}
	
	public int getUploadStream() {
		return uploadStream;
	}
	public void setUploadStream(int uploadStream) {
		this.uploadStream = uploadStream;
	}
	public int getDownloadStream() {
		return downloadStream;
	}
	public void setDownloadStream(int downloadStream) {
		this.downloadStream = downloadStream;
	}
	public int getSumStream() {
		return sumStream;
	}
	public void setSumStream(int sumStream) {
		this.sumStream = sumStream;
	}
	@Override
	public String toString() {
		return  uploadStream + "\t" + downloadStream + "\t" + sumStream;
	}
	@Override
	public void write(DataOutput paramDataOutput) throws IOException {
		paramDataOutput.writeInt(this.uploadStream);
		paramDataOutput.writeInt(this.downloadStream);
		paramDataOutput.writeInt(this.sumStream);
		
	}
	@Override
	public void readFields(DataInput paramDataInput) throws IOException {
		// TODO Auto-generated method stub
		// ȡֵҪ���ϱߵ�д��˳��һ��
		this.uploadStream = paramDataInput.readInt();
		this.downloadStream = paramDataInput.readInt();
		this.sumStream = paramDataInput.readInt();
	}
	@Override
	public int compareTo(Object o) {
		// TODO Auto-generated method stub
		return 0;
	}
		
}

```

map��
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.StreamBean;

public class BeanStreamMapper 
extends Mapper<Object, Text, Text, StreamBean>{
	private Text phone = new Text();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, StreamBean>.Context context)
			throws IOException, InterruptedException {
		// �Խ�����д���Ĺ���
		String[] vals = value.toString().split("\\s+");
		String phone = vals[1];
		String upload = vals[vals.length - 3];
		String download = vals[vals.length - 2];
		StreamBean streamBean = new StreamBean(upload, download);
		this.phone.set(phone);
		// 
		context.write(this.phone, streamBean);
	}

}
```

reduce��
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.StreamBean;

public class BeanStreamReduce 
extends Reducer<Text, StreamBean, Text, StreamBean> {
	
	@Override
	protected void reduce(Text key, Iterable<StreamBean> lists,
			Reducer<Text, StreamBean, Text, StreamBean>.Context context) throws IOException, InterruptedException {
		// ÿ�����������������һ��,����������һ��,��������һ��
		int uploadSum = 0;
		int downloadSum = 0;
		for (StreamBean streamBean : lists) {
			uploadSum += streamBean.getUploadStream();
			downloadSum += streamBean.getDownloadStream();
		}
		StreamBean streamBean = new StreamBean(uploadSum, downloadSum);
		context.write(key, streamBean);
	}

}
```

main����
```
package com.lanou.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.Util.BeanStreamMapper;
import com.lanou.Util.BeanStreamReduce;
import com.lanou.Util.MobilePartitioner;
import com.lanou.model.StreamBean;
/*
 * ��log�л�ȡ�ֻ���,��������,��������,������
 * reduce֮��,������Ҫ�ֻ���,��������(��������,��������,�����ܺ�)
 * Ҫʹ���Զ����������Ϊ���
 * 1.�����������ǵ����ݽ��н�ģ(����ʵ����)
 * 2.mapper��reducer�е��β���ôд,map()����������ô�����ݽ����и�ɸѡ,�����ǵ����ݷ����Զ����bean(ע��javaBean�淶)����,���bean����key����value���
 * 3.map�Ľ������Ҫд�������,����Ҫע�������Զ������һ��Ҫ֧�����л�WritableComparable
 *    Serializable��java�ṩ�����������л�����hadoop�н���ʹ��hadoop�����ṩ��WritableComparable�ӿ���ʵ�����л��ͷ����л���
 *    Ҫע����дwrite()��readFields()���� ����reducer�׶λ�ȡ��������
 *    ע��:readFields()�е��õ�read����һ��Ҫ����д���˳����ж�ȡ�����������˳�����
 * 4.main�е�job.setJarByClass(BeanStreamTest.class);
        job.setMapperClass(BeanStreamMapper.class);
        job.setCombinerClass(BeanStreamReducer.class);
        job.setReducerClass(BeanStreamReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(StreamBean.class);
                    ע������
                    ע��������key,value������
 * 5.ע����дtoString����
 */
public class PartitionerTest {
	/*
	 * ���ݷ�����������뵽��ͬλ��
	 */
	public static void main(String[] args)
		    throws Exception
		  {
		// �����ļ�
		    Configuration conf = new Configuration();
		    System.setProperty("HADOOP_USER_NAME", "hadoop");
		    conf.set("fs.defaultFS",  "hdfs://172.18.24.28:9000");
		 // ��������
		    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2){
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		 // ���� -> mapreduce
		    Job job = Job.getInstance(conf, "word count");
		 // ���ʹ�õ�
		    job.setJarByClass(PartitionerTest.class);
		 // ����ִ��map��������
		    job.setMapperClass(BeanStreamMapper.class);
		 // �ϲ�
		    job.setCombinerClass(BeanStreamReduce.class);
		 // ����ִ��reduce��������
		    job.setReducerClass(BeanStreamReduce.class);
		    // ���÷���,ʹ���Ķ���Patitioner
		    job.setPartitionerClass(MobilePartitioner.class);
		    // ����reduce������,�������������Ч.
		    // ��������С�ڷ���ĸ���,ֻ�ܴ��ڵ���.��������쳣
		    // ����������Զ������,��ʹ��ϵͳĬ�ϵ�HashPartitioner
		    job.setNumReduceTasks(3);
		    // ����reduce������������
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(StreamBean.class);
		    // ����jobҪ��ȡ���ļ����Ķ�Щ
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    // ����job���õĽ�����λ��.
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		    // job.waitForCompletion(true) ���������úõ�job�������ύ��yarn�������Ƿ�����Դ.
		    System.exit(job.waitForCompletion(true) ? 0 : 1);
		  }

}
```



