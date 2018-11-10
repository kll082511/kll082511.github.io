- 1.����copy��ȡ�ڶ���block�������

```
	@Test
	public void downloadBySeek() throws IllegalArgumentException, IOException {
		// ��ȡhadoop��ѹ�����ļ�
		RemoteIterator<LocatedFileStatus> listFiles = 
				fileSystem.listFiles(new Path("/list/hadoop-2.7.3.tar.gz"), false);
		// ����������,����hadoopѹ����.
		FSDataInputStream fsDataInputStream =
				fileSystem.open(new Path("/list/hadoop-2.7.3.tar.gz"));
		// ��ȡ���ļ������п�.
		while (listFiles.hasNext()) {
			// 
			LocatedFileStatus next = listFiles.next();
			// ��ȡ ���ļ�������block����Ϣ
			BlockLocation[] blockLocations = next.getBlockLocations();
			// ѭ����
			for (int i = 0; i < blockLocations.length; i++) {
				// �ж��ǵڶ�����,���ж�ȡ������.
				if (i == 1) {
					fsDataInputStream.seek(blockLocations[i].getOffset());
					break;
				}
			}
			
		}
		// ���������,ָ��λ��.
		FileOutputStream foStream = 
				new FileOutputStream("C:/Users/Administrator/Desktop/ll/oo.tar.gz");
		// �������.
		org.apache.commons.compress.utils.IOUtils.copy(fsDataInputStream, foStream);
		
	}
```

- 2.��ȡ����block����(ͨ������ƫ����ʵ��)

```
	@Test
	// ������block��������������
	public void getBlocks() throws FileNotFoundException, IllegalArgumentException, IOException {
		// ͨ��fileSystem��listFiles���������Զ�ʵ�ֵݹ�(�Դ��ݹ�)�г��ļ����ͣ����ص���һ��Զ�̿ɵ�������,��Ҫ����������������һ�������Ƿ�����·�����ڶ��������Ƿ�ݹ�
		RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(new Path("/list/had.zip"), false);
		// ������Ҫ��ȡ���ļ�
		FSDataInputStream open = 
				fileSystem.open(new Path("/list/had.zip"));
		while (listFiles.hasNext()) {
			LocatedFileStatus next = listFiles.next();
			BlockLocation[] locations = 
					next.getBlockLocations();
			for (int i = 0; i < locations.length; i++) {
				// Ϊ�˲���ÿ�ζ�ȡһ��������,ֱ����������ƫ����.
				// seek�Ƕ����������ļ�ָ����������
				open.seek(0);
				FileOutputStream fos = 
						new FileOutputStream("C:/Users/Administrator/Desktop/ll/had1" + i + ".zip");
				// �﷨��
				org.apache.commons.io.IOUtils.copyLarge(open, fos, locations[i].getOffset(), locations[i].getLength());
			}
			
		}
	}
```

- 3.��ȡ����block����(��org.apache.commons.io.IOUtils.copyLarge()���Ĳη���)
```
// ������block��������������
	public void getBlocksSec() throws FileNotFoundException, IllegalArgumentException, IOException {
		RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(new Path("/list/had.zip"), false);
		FSDataInputStream open = 
				fileSystem.open(new Path("/list/had.zip"));
		while (listFiles.hasNext()) {
			LocatedFileStatus next = listFiles.next();
			BlockLocation[] locations = 
					next.getBlockLocations();
			Long sum = 0L;
			for (int i = 0; i < locations.length; i++) {
				
				FileOutputStream fos = 
						new FileOutputStream("C:/Users/Administrator/Desktop/ll/had1" + i + ".zip");
				long length = locations[i].getLength();
				long offset = locations[i].getOffset();
				// ÿ�εĶ�ȡ��һ���µ�block��
				// ����һ:������
				// ������:д����
				// ������:�����ƫ����
				// ������:���ڶ�ȡ��block��ĳ���.
				org.apache.commons.io.IOUtils.copyLarge
				(open, fos, sum, locations[i].getLength());
				// ��Ϊ��ʼ��ƫ����Ϊ0,����sumҪ�������м�.
				sum = length + offset;
			}
			
		}
	}
```

- 4.��ȡ�ļ�����,ͨ��ʵ����ʵ��

```
public void testSeek() throws IllegalArgumentException, IOException {
		BlockInfo blockInfo1 = new BlockInfo(0, 6);
		BlockInfo blockInfo2 = new BlockInfo(6, 6);
		BlockInfo blockInfo3 = new BlockInfo(12, 3);
		List<BlockInfo> blockInfos = new ArrayList<>();
		blockInfos.add(blockInfo1);
		blockInfos.add(blockInfo2);
		blockInfos.add(blockInfo3);
		FSDataInputStream inputStream = fileSystem.open(new Path("/list/in.md"));
		int sum = 0;
		for (int i = 0; i < blockInfos.size(); i++) {
			int offset = blockInfos.get(i).getOffset();
			int length = blockInfos.get(i).getLength();
			FileOutputStream fileOutputStream = new FileOutputStream("C:/Users/Administrator/Desktop/hdfs/output/seek"+i+".txt");
			// ƫ��������Ϊ��,ָ���Ƶ�ƫ����Ϊ0��λ��.
			org.apache.commons.io.IOUtils.copyLarge(inputStream, fileOutputStream, 0, length);
			sum = offset+length;
			
		}
	}
```
