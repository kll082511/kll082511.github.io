**azkaban 工作流调度器**
- 1.通过command命令来执行,以键值对的形式存在:
type=command
comman=真正要执行 的命令
- 2.在执行的时候经常需要执行多个job,需要有先后顺序,用[dependencies=job.名称]来指定我当前要执行的job依赖于哪儿个job,在依赖的这个job执行完后再来执行我
- 3.除了可以执行我们的liunx基础命令,hdfs的命令也可以执行
	- 帮我们执行mapreduce:job中设置要执行的mapreduce的命令,命令中用到的要执行的jar包,我们需要在上传job的时候一并压缩,上传到azkaban.
- 4.hive命令也能执行
	- 4.1 hive -e;
	- 4.2 hive -f '文件名'.这时候不仅要上传job本身,还要上传对应的文件,将job和文件一起压缩打包进行上传.
