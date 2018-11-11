### Reduce��join
```
/*
	 * reduce ��join
	 * 1. �ļ���ʽҪע�⣬���������룬�����Լ�����String string = new String(value.getBytes(), "GBK");
	 * 2.����һ������������bean�����ܴ�Ŷ�����Ϣ�����ܴ����Ʒ��Ϣ��
	 * 3.��map�˽��ܲ�ͬ�ļ������ݣ��������Ķ����ļ�����������bean���������϶�Ӧ����Ϣ��û�е�Ҳ��Ҫ��null��Ҫ����Ĭ��ֵ������
	 * ����ɿ�ָ���쳣����ʱ��map�Ĺ���������<thingsid��bean>Ҫע�����л���
	 * 4.��ʱ����Ϊmap�������key��tingsId�����Դ���reduce�������Ѿ������ǽ��кϲ���
	 * ����������ʽ��<thingsId��[orderBean1��orderBean2��thingsBean]>
	 * Ҫ���������е����ݽ��в�ֳ�������ʽ��thingsBan��[orderBean1��orderBean2]������ֻ��Ҫ��orderBean�ļ��Ͻ���ѭ������orderBean
	 * ȱ�ٵ���Ʒ��Ϣͨ��thingsBean��ȫ����͵õ���������Ҫ��������join֮��������ˣ�Ȼ��Ϳ�������ˡ���Ϊ����Ҫʹ�õ�����
	 * ���Ե����������������ó���ʹ�ã�Ҫ���������
 */
```

Bean��
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.Writable;

public class JoinBean implements Writable {
	private int orderId;// ����ID
	private int count;// ������������
	private int stockCount;// ��Ʒ���
	private String thingsId;// ��ƷID
	private String thingsName;// ��Ʒ����
	private int flag;//��ʾ��ǰ������ͨ���Ķ����ļ��õ���
	// ��bean���������ļ�������
	public void setOrder(int orderId, String thingsId, int count) {
		this.thingsId = thingsId;
		this.orderId = orderId;
		this.count = count;
		// û��ȡ��������Ҳ��Ҫ��null ����ʽд�룬
		// �������reduce��ʱ��ͻ��ָ���쳣��
		this.thingsName = "";
		this.stockCount = -1;
		this.flag = 0;
	}
	// ��bean����Ʒ���ļ�������
	public void setThings(String thingsId, String thingsName, int stockCount) {
		this.thingsId = thingsId;
		this.thingsName = thingsName;
		this.stockCount = stockCount;
		// û��ȡ��������Ҳ��Ҫ��null ����ʽд�룬
		// �������reduce��ʱ��ͻ��ָ���쳣��
		this.orderId = 0;
		this.stockCount = 0;
		this.flag = 1;
	}

	public void setThingsByBean(JoinBean thingsBean) {
		this.thingsName = thingsBean.getThingsName();
		this.stockCount = thingsBean.getStockCount();
	}
	
	public int getFlag() {
		return flag;
	}
	public void setFlag(int flag) {
		this.flag = flag;
	}
	public JoinBean() {
		super();
	}
	
	public JoinBean(int orderId, int count, int stockCount, String thingsId, String thingsName) {
		super();
		this.orderId = orderId;
		this.count = count;
		this.stockCount = stockCount;
		this.thingsId = thingsId;
		this.thingsName = thingsName;
	}

	public int getOrderId() {
		return orderId;
	}

	public void setOrderId(int orderId) {
		this.orderId = orderId;
	}

	public int getCount() {
		return count;
	}

	public void setCount(int count) {
		this.count = count;
	}

	public int getStockCount() {
		return stockCount;
	}

	public void setStockCount(int stockCount) {
		this.stockCount = stockCount;
	}

	public String getThingsId() {
		return thingsId;
	}

	public void setThingsId(String thingsId) {
		this.thingsId = thingsId;
	}

	public String getThingsName() {
		return thingsName;
	}

	public void setThingsName(String thingsName) {
		this.thingsName = thingsName;
	}

	@Override
	public void readFields(DataInput input) throws IOException {
		this.orderId = input.readInt();
		this.stockCount = input.readInt();
		this.count = input.readInt();
		this.thingsId = input.readUTF();
		this.thingsName = input.readUTF();
		this.flag = input.readInt();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		output.writeInt(this.orderId);
		output.writeInt(this.stockCount);
		output.writeInt(this.count);
		output.writeUTF(this.thingsId); 
		output.writeUTF(this.thingsName);  
		output.writeInt(this.flag);
	}

	@Override
	public String toString() {
		return orderId + "\t" + count + "\t" + stockCount + "\t"
				+ thingsId + "\t" + thingsName + "\t" + flag;
	}
}
```

map��
```
package com.lanou.until;

import java.io.IOException;
import java.util.Arrays;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import com.lanou.model.JoinBean;

public class ReduceJoinMapper extends Mapper<Object, Text, Text, JoinBean> {
	private JoinBean joinBean = new JoinBean();;
	private Text thingsText = new Text();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, JoinBean>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// ��ȡ��Ƭ(���������,���Ե����䷽��)
		
		FileSplit fileSplit = (FileSplit)context.getInputSplit();
		String fileName = fileSplit.getPath().getName();
		System.out.println(fileName);
		//String string = new String(value.getBytes(), "GBK");
		String[] vals = value.toString().split("\\s+");
		System.out.println(Arrays.toString(vals));
		//JoinBean joinBean = new JoinBean();
		if ("orders.txt".equals(fileName)) {
			joinBean.setOrder(Integer.parseInt(vals[0]), vals[2], Integer.parseInt(vals[3]));
		}else {
			joinBean.setThings(vals[0], vals[1], Integer.parseInt(vals[2]));
		}
		thingsText.set(joinBean.getThingsId());
		context.write(thingsText, joinBean);
	}
}
```

reduce��
```
package com.lanou.until;

import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.util.ArrayList;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.JoinBean;

public class ReduceJoinReduce extends Reducer<Text, JoinBean, Text, JoinBean> {
	private JoinBean thingBean = new JoinBean();
	//private JoinBean orderBean = new JoinBean();
	
	@Override
	protected void reduce(Text key, Iterable<JoinBean> value, Reducer<Text, JoinBean, Text, JoinBean>.Context context)
			throws IOException, InterruptedException {
		//
		// key ��Ʒid
		// ��Ʒ value [����1,����2,]
		ArrayList<JoinBean> ordersList = new ArrayList<>();
		for (JoinBean joinBean : value) {
			System.out.println("i am run");
			if (joinBean.getFlag() == 1) {
				thingBean.setCount(joinBean.getCount());
				thingBean.setThingsName(joinBean.getThingsName());
				thingBean.setThingsId(joinBean.getThingsId());
				thingBean .setCount(joinBean.getCount());
				thingBean.setStockCount(joinBean.getStockCount());
			}else {
				JoinBean joinBean2 = new JoinBean();
				try {
					BeanUtils.copyProperties(joinBean2, joinBean);
				} catch (IllegalAccessException | InvocationTargetException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				ordersList.add(joinBean2);
			}
		}
		// ��Ʒ1      ����1������2
		// ���������У����еĶ���������Ʒ��Ϣ
		for (JoinBean joinBean : ordersList) {
			joinBean.setThingsByBean(thingBean);
			context.write(key, joinBean);
		}
	}

}
```

main��
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.JoinBean;
import com.lanou.until.ReduceJoinMapper;
import com.lanou.until.ReduceJoinReduce;


public class ReduceJoinTest {
	
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		// 
		/*
		 * reduce ��join
		 * 1. �ļ���ʽҪע�⣬���������룬�����Լ�����String string = new String(value.getBytes(), "GBK");
		 * 2.����һ������������bean�����ܴ�Ŷ�����Ϣ�����ܴ����Ʒ��Ϣ��
		 * 3.��map�˽��ܲ�ͬ�ļ������ݣ��������Ķ����ļ�����������bean���������϶�Ӧ����Ϣ��û�е�Ҳ��Ҫ��null��Ҫ����Ĭ��ֵ������
		 * ����ɿ�ָ���쳣����ʱ��map�Ĺ���������<thingsid��bean>Ҫע�����л���
		 * 4.��ʱ����Ϊmap�������key��tingsId�����Դ���reduce�������Ѿ������ǽ��кϲ���
		 * ����������ʽ��<thingsId��[orderBean1��orderBean2��thingsBean]>
		 * Ҫ���������е����ݽ��в�ֳ�������ʽ��thingsBan��[orderBean1��orderBean2]������ֻ��Ҫ��orderBean�ļ��Ͻ���ѭ������orderBean
		 * ȱ�ٵ���Ʒ��Ϣͨ��thingsBean��ȫ����͵õ���������Ҫ��������join֮��������ˣ�Ȼ��Ϳ�������ˡ���Ϊ����Ҫʹ�õ�����
		 * ���Ե����������������ó���ʹ�ã�Ҫ���������
		 */
		  Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(ReduceJoinTest.class);
		    job.setMapperClass(ReduceJoinMapper.class);
		    job.setReducerClass(ReduceJoinReduce.class);
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(JoinBean.class);
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

###  map��join
���õ�mapper����setup����ִֻ��һ�ε�����.

```
/*
* map��join
* ����orders.txt �ӱ�things.txt
* ��������map�����ӱ��Ի����ļ�����ʽ���룬���뵽job��
* ����������map�д���֮�󣬾���ȱ�ٴӱ����ݵ������ԡ�
* ����Ҫ�Դ�����������ݵ���������������ȱʧ�Ĵӱ�����
* �ӱ����ݴ�-��setup()�������ѭ������mapǰִֻ��һ�εķ���
* �������ǵĻ����ļ����м��ء����������һ��һ����������Ʒ��Ϣ�Ķ�����<��ƷID����Ʒ����>��ʽ����map�У���������ʹ�á�
* ������map�У�����ͨ��map.get(key)�͵���ȡ����ǰ�����������Ӧ����Ʒ�ˣ�����ֵ��д��Ϳ����ˡ�
* 
*/
```

Bean��
```
package com.lanou.model;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;
public class OrderInfo implements WritableComparable<OrderInfo> {
	private String orderId;
	private String goodsId;
	private double price;
	public void setOrder(String orderId, String goodsId, double price) {
		this.goodsId = goodsId;
		this.orderId = orderId;
		this.price = price;
	}
	@Override
	public String toString() {
		return "OrderInfo [orderId=" + orderId + ", goodsId=" + goodsId + ", price=" + price + "]";
	}
	public String getOrderId() {
		return orderId;
	}
	public void setOrderId(String orderId) {
		this.orderId = orderId;
	}
	public String getGoodsId() {
		return goodsId;
	}
	public void setGoodsId(String goodsId) {
		this.goodsId = goodsId;
	}
	public double getPrice() {
		return price;
	}
	public void setPrice(double price) {
		this.price = price;
	}
	public OrderInfo() {
		super();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		output.writeUTF(this.orderId);
		output.writeUTF(this.goodsId);
		output.writeDouble(this.price);
	}
	@Override
	public void readFields(DataInput input) throws IOException {
		this.orderId = input.readUTF();
		this.goodsId = input.readUTF();
		this.price = input.readDouble();
		
	}
	@Override
	public int compareTo(OrderInfo arg0) {
		int relP = (int) (this.price - arg0.getPrice());
		int relI = arg0.getGoodsId().compareTo(this.orderId);
		return relI == 0 ? relP : -relI;
	}
}
```

map��
```
package com.lanou.until;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.MapJoinBean;

public class MapJoinMapper extends Mapper<Object, Text, Text, MapJoinBean> {
	private Text keyText = new Text();
	private MapJoinBean mapJoinBean = new MapJoinBean();
	private Map<String, MapJoinBean> containMap = new HashMap<>();
	
	
	// map��ѭ��ִ��
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, MapJoinBean>.Context context)
			throws IOException, InterruptedException {
	//FileInputStream fileInputStream	= new FileInputStream("things.txt");
		String[] vals = value.toString().split("\\s+");
		// ֻ�ж�����Ϣ
		mapJoinBean.setOrder(vals[0], vals[2], vals[3]);
		// ���ݶ����е���Ʒ��Ϣ�����ҵ������ж�Ӧ����Ʒ���󣬽���������䡣
		MapJoinBean thingsBean = containMap.get(mapJoinBean.getThingsId());
		
		// �����Ƕ�������ȱ�ٵ���Ʒ���Խ��и�ֵ
		mapJoinBean.setThings(thingsBean.getThingsName(), thingsBean.getStockCount());
		
		keyText.set(mapJoinBean.getThingsId());
		context.write(keyText, mapJoinBean);
	}
	
	// ��map֮ǰֻ����һ��
	@Override
	protected void setup(Mapper<Object, Text, Text, MapJoinBean>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		//FileReader fileReader = new FileReader("things.txt");
		FileInputStream fileInputStream	= new FileInputStream("things.txt");
		BufferedReader br = new BufferedReader(new InputStreamReader(fileInputStream, "UTF-8"));
		String line = "";
		// ����һ�����������洢���н�����������Ʒ ���� -��map
		while (!StringUtils.isEmpty(line = br.readLine())) {
			// ��ȡ����Ʒ��Ϣ
			System.out.println("line:" + line);
			String[] things = line.split("\\s+");
			// ������Ʒ����
			MapJoinBean thingsBean = new MapJoinBean();
			// ������ֵ
			thingsBean.setThings(things[0], things[1], Integer.parseInt(things[2]));
			// ��Ʒid����Ʒ�������ʽ����map��
			containMap.put(thingsBean.getThingsId(), thingsBean);

		}
	}

}
```

reduce��
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.OrderInfo;

public class MaxOrderReduce extends Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable> {

	@Override
	protected void reduce(OrderInfo orderInfo, Iterable<NullWritable> value,
			Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		for (NullWritable nullWritable : value) {
			System.out.println(nullWritable);
		}
		arg2.write(orderInfo, NullWritable.get());
	}
}
```

main����
```
package com.lanou.test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.until.FileWordReduce;
import com.lanou.until.MapJoinMapper;

public class MapJoinTest {
	/*
	 * map��join
	 * ����orders.txt �ӱ�things.txt
	 * ��������map�����ӱ��Ի����ļ�����ʽ���룬���뵽job��
	 * ����������map�д���֮�󣬾���ȱ�ٴӱ����ݵ������ԡ�
	 * ����Ҫ�Դ�����������ݵ���������������ȱʧ�Ĵӱ�����
	 * �ӱ����ݴ�-��setup()�������ѭ������mapǰִֻ��һ�εķ���
	 * �������ǵĻ����ļ����м��ء����������һ��һ����������Ʒ��Ϣ�Ķ�����<��ƷID����Ʒ����>��ʽ����map�У���������ʹ�á�
	 * ������map�У�����ͨ��map.get(key)�͵���ȡ����ǰ�����������Ӧ����Ʒ�ˣ�����ֵ��д��Ϳ����ˡ�
	 * 
	 */
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MapJoinTest.class);
		    job.setMapperClass(MapJoinMapper.class);
		    job.setNumReduceTasks(0);
		    job.setReducerClass(FileWordReduce.class);
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(Text.class);
		    
		    // ��ӻ����ļ�
		    job.addCacheFile(new URI("hdfs://172.18.24.28:9000/list/things.txt"));
		    
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```
### ͨ������GroupingComparator��ȡ��ͬ�������е�������
Bean�����һ��һ��
```
/*
 * ���� - GroupingComparator
 * 1.Ҫ�������Զ���ķ����߼���
 * 1.1����һ�����캯�����ڲ�����super��Ҫ���з���Ƚϵ��ࣩ
 * 1.2ע������Ҫ���бȽϵ������ȥʵ��Comparable�ӿ�
 * ��Ϊ���Ƿ���Ƚ���sort����֮�󣬶���ֻ�ܶ����ڵĶ�����бȽϡ�����Ҫ��ʵ�����compareTo�����У�
 * ��Ҫ���з����������Ϊ��������ĵ�һҪ�ء�ֻҪ�������ܱ�֤��ͬ���������ڡ������Ҫ���򣬴ӵڶ�Ҫ�ؿ�ʼ��
 * 1.3��������Ҫ���з�����߼���ʵ�ֳ��󷽷�
 * public int compare(WritableComparable a, WritableComparable b)
 * �����ڵ�����������бȽϣ��������0���ͻ���Ϊ�Ǹý�һ��reduce
 * ���ǲ��᣻��������reduce�������������һ�ֱȽϡ����磺1.getOrderId = 2.getOrderId����2.getOrderId = 3.getOrderId
 * ֱ���ȽϽ����λ0�ˣ�֪������һ��reduce�ˣ���ʱ��������ͬ�ķ���һ��reduce�У��������reduce���л�ȡ����Щ����Ϊ
 * �ý���һ���bean��ֻҪѭ�����������ڵ������е�key����ÿһ��bean��
 * ��һ��reduceִ���꣬�����ִ�����ǵķ���Ƚϣ�������ͬ�ģ���ִ��reduce���Ӷ��γ�һ��ѭ����һֱ������reduce���н�����
 * 
 */
```

mapper��
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.OrderInfo;

public class MaxOrderMapper extends Mapper<Object, Text, OrderInfo, NullWritable> {
	private OrderInfo orderinfo = new OrderInfo();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, OrderInfo, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String[] vals = value.toString().split("\\s+");
		orderinfo.setOrder(vals[0], vals[1], Double.parseDouble(vals[2]));
		context.write(orderinfo, NullWritable.get());
	}

}
```

Reduce��
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.OrderInfo;

public class MaxOrderReduce extends Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable> {

	@Override
	protected void reduce(OrderInfo orderInfo, Iterable<NullWritable> value,
			Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		for (NullWritable nullWritable : value) {
			System.out.println(nullWritable);
		}
		arg2.write(orderInfo, NullWritable.get());
	}
}
```

Test��
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.OrderInfo;
import com.lanou.until.GroupByOrderCompartor;
import com.lanou.until.MaxOrderMapper;
import com.lanou.until.MaxOrderReduce;
/*
 * ���� - GroupingComparator
 * 1.Ҫ�������Զ���ķ����߼���
 * 1.1����һ�����캯�����ڲ�����super��Ҫ���з���Ƚϵ��ࣩ
 * 1.2ע������Ҫ���бȽϵ������ȥʵ��Comparable�ӿ�
 * ��Ϊ���Ƿ���Ƚ���sort����֮�󣬶���ֻ�ܶ����ڵĶ�����бȽϡ�����Ҫ��ʵ�����compareTo�����У�
 * ��Ҫ���з����������Ϊ��������ĵ�һҪ�ء�ֻҪ�������ܱ�֤��ͬ���������ڡ������Ҫ���򣬴ӵڶ�Ҫ�ؿ�ʼ��
 * 1.3��������Ҫ���з�����߼���ʵ�ֳ��󷽷�
 * public int compare(WritableComparable a, WritableComparable b)
 * �����ڵ�����������бȽϣ��������0���ͻ���Ϊ�Ǹý�һ��reduce
 * ���ǲ��᣻��������reduce�������������һ�ֱȽϡ����磺1.getOrderId = 2.getOrderId����2.getOrderId = 3.getOrderId
 * ֱ���ȽϽ����λ0�ˣ�֪������һ��reduce�ˣ���ʱ��������ͬ�ķ���һ��reduce�У��������reduce���л�ȡ����Щ����Ϊ
 * �ý���һ���bean��ֻҪѭ�����������ڵ������е�key����ÿһ��bean��
 * ��һ��reduceִ���꣬�����ִ�����ǵķ���Ƚϣ�������ͬ�ģ���ִ��reduce���Ӷ��γ�һ��ѭ����һֱ������reduce���н�����
 * 
 */
public class MaxOrderTest {
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MaxOrderTest.class);
		    job.setMapperClass(MaxOrderMapper.class);
		    //job.setNumReduceTasks(0);
		    job.setReducerClass(MaxOrderReduce.class);
		    job.setOutputKeyClass(OrderInfo.class);
		    job.setOutputValueClass(NullWritable.class);
		    
		    job.setGroupingComparatorClass(GroupByOrderCompartor.class);
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

### ���û����ļ�ʵ��Reduce_Join
ʵ����
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class MapJoinBean implements WritableComparable<MapJoinBean> {
	private int orderId;// ����ID
	private int count;// ������������
	private int stockCount;// ��Ʒ���
	private String thingsId;// ��ƷID
	private String thingsName;// ��Ʒ����
	
	public void setOrder(String orderId, String thingsId, String count) {
		this.thingsId = thingsId;
		this.orderId = Integer.parseInt(orderId);
		this.count = Integer.parseInt(count);
		
	}
	
	public void setThings(String thingsName, int stockCount) {
		this.thingsName = thingsName;
		this.stockCount = stockCount;
	}
	public void setThings(String thingsId, String thingsName, int stockCount) {
		this.thingsId = thingsId;
		this.thingsName = thingsName;
		this.stockCount = stockCount;
		// û��ȡ��������Ҳ��Ҫ��null ����ʽд�룬
//		// �������reduce��ʱ��ͻ��ָ���쳣��
//		this.orderId = 0;
//		this.stockCount = 0;
	}
	public MapJoinBean() {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public MapJoinBean(int orderId, int count, int stockCount, String thingsId, String thingsName) {
		super();
		this.orderId = orderId;
		this.count = count;
		this.stockCount = stockCount;
		this.thingsId = thingsId;
		this.thingsName = thingsName;
	}
	public int getOrderId() {
		return orderId;
	}
	public void setOrderId(int orderId) {
		this.orderId = orderId;
	}
	public int getCount() {
		return count;
	}
	public void setCount(int count) {
		this.count = count;
	}
	public int getStockCount() {
		return stockCount;
	}
	public void setStockCount(int stockCount) {
		this.stockCount = stockCount;
	}
	public String getThingsId() {
		return thingsId;
	}
	public void setThingsId(String thingsId) {
		this.thingsId = thingsId;
	}
	public String getThingsName() {
		return thingsName;
	}
	public void setThingsName(String thingsName) {
		this.thingsName = thingsName;
	}
	@Override
	public String toString() {
		return "MapJoinBean [orderId=" + orderId + ", count=" + count + ", stockCount=" + stockCount + ", thingsId="
				+ thingsId + ", thingsName=" + thingsName + "]";
	}
	@Override
	public void readFields(DataInput input) throws IOException {
		// TODO Auto-generated method stub
		this.orderId = input.readInt();
		this.stockCount = input.readInt();
		this.count = input.readInt();
		this.thingsId = input.readUTF();
		this.thingsName = input.readUTF();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		// TODO Auto-generated method stub
		output.writeInt(this.orderId);
		output.writeInt(this.stockCount);
		output.writeInt(this.count);
		output.writeUTF(this.thingsId); 
		output.writeUTF(this.thingsName);
	}

	@Override
	public int compareTo(MapJoinBean o) {
		// TODO Auto-generated method stub
		int id = this.orderId - o.getOrderId();
		int countrel = this.thingsId.compareTo(o.getThingsId());
		
		return id != 0 ? (- id) : (countrel == 0 ? -1 : countrel);
	}
	
	

}
```

mapper��
```
package com.lanou.until;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.MapJoinBean;

public class JoinMapper extends Mapper<Object, Text, MapJoinBean, NullWritable> {
	private Text keyText = new Text();
	private MapJoinBean mapJoinBean = new MapJoinBean();
	private Map<String, MapJoinBean> containMap = new HashMap<>();

	// map��ѭ��ִ��
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, MapJoinBean, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// FileInputStream fileInputStream = new FileInputStream("things.txt");
		String[] vals = value.toString().split("\\s+");
		// ֻ�ж�����Ϣ
		mapJoinBean.setOrder(vals[0], vals[2], vals[3]);
		// ���ݶ����е���Ʒ��Ϣ�����ҵ������ж�Ӧ����Ʒ���󣬽���������䡣
		MapJoinBean thingsBean = containMap.get(mapJoinBean.getThingsId());

		// �����Ƕ�������ȱ�ٵ���Ʒ���Խ��и�ֵ
		mapJoinBean.setThings(thingsBean.getThingsName(), thingsBean.getStockCount());

		keyText.set(mapJoinBean.getThingsId());
		context.write(mapJoinBean, NullWritable.get());
	}

	// ��map֮ǰֻ����һ��
	@Override
	protected void setup(Mapper<Object, Text, MapJoinBean, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// FileReader fileReader = new FileReader("things.txt");
		FileInputStream fileInputStream = new FileInputStream("things.txt");
		BufferedReader br = new BufferedReader(new InputStreamReader(fileInputStream, "UTF-8"));
		String line = "";
		// ����һ�����������洢���н�����������Ʒ ���� -��map
		while (!StringUtils.isEmpty(line = br.readLine())) {
			// ��ȡ����Ʒ��Ϣ
			System.out.println("line:" + line);
			String[] things = line.split("\\s+");
			// ������Ʒ����
			MapJoinBean thingsBean = new MapJoinBean();
			// ������ֵ
			thingsBean.setThings(things[0], things[1], Integer.parseInt(things[2]));
			// ��Ʒid����Ʒ�������ʽ����map��
			containMap.put(thingsBean.getThingsId(), thingsBean);

		}
	}

}
```

Reduce��
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.MapJoinBean;

public class JoinReduce extends Reducer<MapJoinBean, NullWritable, MapJoinBean, NullWritable> {
	
	
	@Override
	protected void reduce(MapJoinBean arg0, Iterable<NullWritable> arg1,
			Reducer<MapJoinBean, NullWritable, MapJoinBean, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		arg2.write(arg0, NullWritable.get());
	}

}
```

Test��
```
package com.lanou.test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.MapJoinBean;
import com.lanou.until.JoinMapper;
import com.lanou.until.JoinReduce;

public class JinTest {
	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MapJoinTest.class);
		    job.setMapperClass(JoinMapper.class);
		    //job.setNumReduceTasks(0);
		    job.setReducerClass(JoinReduce.class);
		    job.setOutputKeyClass(MapJoinBean.class);
		    job.setOutputValueClass(NullWritable.class);
		    
		    // ��ӻ����ļ�
		    job.addCacheFile(new URI("hdfs://172.18.24.28:9000/list/things.txt"));
		    
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```