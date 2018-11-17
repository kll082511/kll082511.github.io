### OutputFormat
��outputFormat�н��յ��ļ���,����Ҫд��ȥ���ļ�.���Է���Ҫ��Mapper���д���ļ����Ͷ�Ӧ.
��������:
Test��
```
package com.lanou.test;

import java.io.IOException;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.util.MyOutputFormat;
import com.lanou.util.MyoutputMapper;

public class MyOutputTest {
	
	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(MyOutputTest.class);
		job.setMapperClass(MyoutputMapper.class);
		job.setOutputFormatClass(MyOutputFormat.class);
		job.setNumReduceTasks(0);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		MyOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper��
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MyoutputMapper extends Mapper<Object, Text, Text, NullWritable> {
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String[] vals = value.toString().split("\\s+");
		if (vals.length != 3) {
			StringBuffer sb = new StringBuffer(value.toString());
			sb.append("\t");
			sb.append("error");
			value.set(sb.toString());
			context.getCounter("lanou", "error").increment(1);
		}
		context.getCounter("lanou", "all").increment(1);
		context.write(value, NullWritable.get());
	}
}
```

OutputFormat��̳�FileOutputFormat��
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MyOutputFormat extends FileOutputFormat<Text, NullWritable> {
	
	
	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// ��outputFormat�н��յ�����Ҫ���Ķ�Щ�ļ�д
		
		Configuration conf = job.getConfiguration();
		// ��ȡ�ļ�ϵͳ
		FileSystem fileSystem = FileSystem.get(conf);
		// ��ȡ·��
		Path outputPath = this.getOutputPath(job);
		System.out.println("name��:" + outputPath.getName());
		System.out.println("self��:" + outputPath);
		System.out.println("parent��:" + outputPath.getParent());
		// ���������·��,���ص�����
		FSDataOutputStream success = fileSystem.create(new Path(outputPath.toString() + "/success.log"));
		FSDataOutputStream error = fileSystem.create(new Path(outputPath.toString() + "/error.log"));
	
		return new MyRecordWriter(success, error);
	}
}
```

RecordWriter��
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class MyRecordWriter extends RecordWriter<Text, NullWritable> {
	// �����������Ϊ����format�е��������
	private FSDataOutputStream success;
	private FSDataOutputStream error;

	public MyRecordWriter(FSDataOutputStream success, FSDataOutputStream error) {
		super();
		this.success = success;
		this.error = error;
	}

	@Override
	public void close(TaskAttemptContext arg0) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		if (success != null) {
			success.close();
		}else if (error != null) {
			error.close();
		}
	}

	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {
		// TODO Auto-generated method stub  
		if (key.toString().contains("error")) {
			error.write(key.toString().getBytes());
		}else {
			success.write(key.toString().getBytes());
		}
		System.out.println("key:" + key);
		System.out.println("value:" + value);
	}

}
```

### InputFormat
Test��
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.util.MyInputFormat;
import com.lanou.util.MyInputMapper;

public class MyInputFormatTest {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(MyInputFormatTest.class);
		job.setMapperClass(MyInputMapper.class);
		job.setInputFormatClass(MyInputFormat.class);
		job.setNumReduceTasks(0);
		
		// job.setReducerClass(JoinReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
			
		}
		 FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		// yarn
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper��
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MyInputMapper extends Mapper<Text, NullWritable, Text, NullWritable> {
	
	@Override
	protected void map(Text key, NullWritable value, Mapper<Text, NullWritable, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		System.out.println("map is run");
		System.out.println("key:" + key);
		System.out.println("value:" + value);
		context.write(key, value);
	}

}
```

InputFormat��̳�FileInputFormat��
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

public class MyInputFormat extends FileInputFormat<Text, NullWritable> {
	
	// �ж��Ƿ���Ҫ��Ƭ
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public RecordReader<Text, NullWritable> createRecordReader(InputSplit split, TaskAttemptContext context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		MyRecordReader dsMyRecordReader = new MyRecordReader();
		dsMyRecordReader.initialize(split, context);
		return dsMyRecordReader;
	}

}
```

��Ҫʵ��RecordReader��
```
package com.lanou.util;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class MyRecordReader extends RecordReader<Text, NullWritable> {
	private FileSplit fileSplit;
	private Configuration conf;
	private boolean result = false;
	private Text resultText = new Text();
	
	
	@Override
	public void close() throws IOException {
		// TODO Auto-generated method stub
		
	}

	@Override
	public Text getCurrentKey() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return resultText;
	}

	@Override
	public NullWritable getCurrentValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return null;
	}

	// ����
	@Override
	public float getProgress() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return (float) (result ? 1.0 : 0.0);
	}

	// ��ȡ��Ϣ
	@Override
	public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// Ϊ��֪��Ҫ��ȡ�Ķ����ļ�
		fileSplit = (FileSplit)split;
		// hdfs��������Ϣ
		conf = context.getConfiguration();
	}

	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// һ���Զ�ȡ����С�ļ�����������,һ���Դ���map.
//		ֻ�е�һ�ε��õ�ʱ��,����true,֮�󶼷���false.
		if (!result) {
			// ��ȡ �ļ�ϵͳ
			FileSystem fs = FileSystem.get(conf);
			// �����ļ�ϵͳ��ȡ������
			FSDataInputStream open = fs.open(fileSplit.getPath());
			// ���ж�.
			// ����һ���ܴ��������ļ����ݵ�buffer
			byte[] buffer = new byte[(int)fileSplit.getLength()];
			// �������ļ�������д��buffer��
			IOUtils.readFully(open, buffer, 0, buffer.length);
//			BufferedReader br = new BufferedReader(new InputStreamReader(open));
//			br.readLine();
			resultText.set(buffer, 0, buffer.length);
			// ��Ϊһ������,������mapѭ���Ƿ�Ҫ����
			result = true;
			return true;
		}
		return false;
	}

}
```

### InputFormat��OutputFormatһ��ʹ��
Test��
```
package com.lanou.homework;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class InputOutPutFormatTest {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(InputOutPutFormatTest.class);
		job.setMapperClass(SelfMapper.class);
		job.setInputFormatClass(SelfInputFormat.class);
		job.setOutputFormatClass(SelfOutputFormat.class);
		job.setNumReduceTasks(0);
		// job.setReducerClass(JoinReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		// yarn
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper��
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
// ��Ϊ����ʹ�����Զ����InputFormat,����keyin,valuein������object,text
public class SelfMapper extends Mapper<NullWritable, Text, Text, Text> {
	private Text fileName = new Text();
	private Text valueText = new Text();
	
	// ����setup������mapѭ����ִֻ�е�����,����ȡ�ļ���.
	@Override
	protected void setup(Mapper<NullWritable, Text, Text, Text>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		FileSplit fileSplit = (FileSplit)context.getInputSplit();
		fileName.set(fileSplit.getPath().getName());
		
	}
	
	@Override
	protected void map(NullWritable key, Text value, Mapper<NullWritable, Text, Text, Text>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		System.out.println("map is run");
		System.out.println("value:" + value);
		String[] vals = value.toString().split("\n");
		for (String string : vals) {
			System.out.println("result:" + string);
			String[] values = string.split("\\s+");
			// �滻��֮��,�Ż��üӵ�error��������ʾ.
			string = string.replace("\r", "");
			string = string.replace("\n", "");
			if (values.length != 3) {
				StringBuffer sb = new StringBuffer(string);
				sb.append("\t");
				sb.append("error");
				valueText.set(sb.toString());
			}else {
				valueText.set(string);
			}
			context.write(fileName, valueText);

		}
		
	}

}
```

RecordWriter��
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class SelfRecordWriter extends RecordWriter<Text, Text> {
	private FSDataOutputStream success;
	private FSDataOutputStream error;
	
	// �вι���
	

	@Override
	public void write(Text paramK, Text paramV) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String result = paramK.toString() + "\t" + paramV.toString() + "\n";
		if (paramV.toString().contains("error")) {
			this.error.writeUTF(result);
		}else {
			this.success.writeUTF(result);
		}
	}

	public SelfRecordWriter(FSDataOutputStream success, FSDataOutputStream error) {
		super();
		this.success = success;
		this.error = error;
	}

	@Override
	public void close(TaskAttemptContext paramTaskAttemptContext) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		if (success != null) {
			success.close();
		}else if (error != null) {
			
			error.close();
		}
		
	}

}
```

InputFormat��
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

public class SelfInputFormat extends FileInputFormat<NullWritable, Text> {
	
	// �ж��Ƿ���Ҫ��Ƭ
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		// TODO Auto-generated method stub
		// ����Ƭ
		return false;
	}

	@Override
	public RecordReader<NullWritable, Text> createRecordReader(InputSplit arg0, TaskAttemptContext arg1)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// Ҫʹ���Զ����InputFormatʵ���Ͼ���Ҫʹ���Զ����
		// recordReader���������ݵĶ�ȡ,����������Ҫ�����Զ����recordWriter�����ظ�InputFormat����ʹ��
		SelfRecordReader selfRecordReader = new SelfRecordReader();
		// ֻ��������,��ΪrecordReader��������ȡ���ݵ�;����������Ҫͨ��initialize()��������Ƭ��Ϣ��job��Ϣ����
		// recordReader,����recordReader��֪����ȥ�Ķ����ļ�ϵͳ��ȡ�Ķ����ļ�
		selfRecordReader.initialize(arg0, arg1);
		return selfRecordReader;
	}
}
```

RecordReader��
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class SelfRecordReader extends RecordReader<NullWritable, Text> {
	private FileSplit fileSplit;
	private Configuration conf ;
	private boolean res = true;// �Ƿ�Ҫִ�еĿ���
	private Text valueText = new Text();

	@Override
	public void close() throws IOException {
		// TODO Auto-generated method stub
		
	}

	@Override
	public NullWritable getCurrentKey() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return NullWritable.get();
	}

	@Override
	public Text getCurrentValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return valueText;
	}

	@Override
	public float getProgress() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return (float)(res ? 0.0 : 1.0);
	}

	@Override
	public void initialize(InputSplit arg0, TaskAttemptContext arg1) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// ��Ϊ����ͨ��initialize��ȡ����������netKeyValue��������н���ʹ��,������Ҫ�����Ӧ�����Խ��н���
		this.fileSplit = (FileSplit)arg0;
		this.conf = arg1.getConfiguration();
	}

	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// ����ÿ�����ô����
		// ��Ϊ����Ҫһ���Խ������ļ�������
		// ����nexKeyValue()ֻ��Ҫ�ڵ�һ�ε��õ�ʱ��,����true�ͺ���.
		if (this.res) {
			// ��Ϊ���ǵ�conf�����Ѿ����յ���job�е�configuration
			// ����ͨ��conf���Ծ��ܵõ�����Ҫ��ȡ���ļ�ϵͳ,������ֻ��Ҫͨ�������ļ�ϵͳ��api��ȡ�ļ��Ϳ�����.
			FileSystem fileSystem = FileSystem.get(conf);
			// ��ȡ��ǰ�ļ�·��
			Path path = fileSplit.getPath();
			// ͨ��open��ȡ��ǰҪ���ļ���������
			// �������ǵ�fileSplitͨ����Ƭ��ȡ��,����Ҫ�����ļ���������.
			FSDataInputStream open = fileSystem.open(path);
			// ���������ļ�������
			byte[] buff = new byte[(int)fileSplit.getLength()];
			// ������������������д���ֽ�������,ִ������һ��,buff�о���������Ҫ�Ľ����
			IOUtils.readFully(open, buff, 0, buff.length);
			// �����ǵĽ��д�뵽���Ƕ�Ӧ��������,
			// ����Ӧ�ķ������е���
			valueText.set(buff, 0, buff.length);
			
			this.res = false;
			return true;
		}
		return false;
	}

}
```
