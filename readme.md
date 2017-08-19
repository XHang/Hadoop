# 简介：hadoop的学习
## 1：简介
hadoop是一个分布式应用系统。看起来好像是专门处理大数据的存储和取出的一套系统  
 它不是一个软件。包含两个核心软件  
 *HDFS*和*MapReduce*
### HDFS  
 是一个分布式文件系统，高度容错性，一般的数据崩溃吓不了它
 适合部署在廉价的机器上，提供高吞吐量的数据访问，大数据集的应用特别适合它
 它的设计特点是：
 1. 文件分块存储，一个文件可能分为几块，存在不同的节点上。
 2. 大数据文件，至少是T级别的文件，G级别的没意思
 3. 流式数据访问，倾向于一次写入，多次访问，希望不用动态改变文件内容,就算是改变只能在文件末尾添加
4. 廉价硬件，普通PC就可以搞定
5. 硬件故障，会将文件的一部分创建副本保存起来，这样即使原文件块损坏了，也能取到文件  
*HDFS*的相关名词  
*Block*：将一个文件进行分块，通常是64M。   
*NameNode*:保存整个文件系统的目录信息、文件信息及分块信息，这是由唯一一台主机专门保存,当然这台主机如果出错，*NameNode*就失效了。  

	ps:在Hadoop2.开始支持activity-standy模式----如果主 NameNode失效，启动备用主机运行NameNode。   
*DataNode*：分布在廉价的计算机上，用于存储*Block块*文件。  
### MapReduce 
 
那，数据存的介质和策略都有HDFS一手承包了，取出数据谁负责呢？MapReduce  
诸如取出数据的最大值，就是MapReduce的本事了  
它的原理是这样的：  
将大的数据分析分成小块逐个分析，最后再将提取出来的数据汇总分析，最终获得我们想要的  


## 2:安装
先决条件：java  ssh软件安装完毕  
 ** 从解压完毕进入到解压目录讲起**
### 第一步
编辑这个文件 `etc/hadoop/hadoop-env.sh`
 设置java环境变量如下所示
 
    # set to the root of your Java installation
    export JAVA_HOME=/home/jdk/jdk1.8.0_11
    
###  第二步
 尝试运行该命令bin/hadoop  
 放心，这个命令只是列出命令列表罢了，不会开启运行的
 
###  第三步
选择hadoop的运行模式有，以下三种：
1. 单机模式  
2. 伪分布模式   
3. 集群模式  
本教程使用伪分布式

### 第四步：  
编辑：`etc/hadoop/core-site.xml:`    
 加上代码
 
    <configuration>
    		<property>
       		 	<name>fs.defaultFS</name>
        		<value>hdfs://localhost:9000</value>
    		</property>
	</configuration>
	
解释：配置NameNode结点的URI,包括协议，url，端口  
			集群里面的每一台机器都需要知道NameNode的地址。  
			DataNode结点会先在NameNode上注册，这样它们的数据才可以被使用。  
			独立的客户端程序通过这个URI跟DataNode交互，以取得文件的块列表。  
			可以认为NameNode是中央节点吧，DataNode要注册在NameNode上面。
			客户端通过和NameNode交互，就可以知道有什么DataNode
### 第五步  
编辑：`etc/hadoop/hdfs-site.xml:`  
加上

    <configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>	
	
解释：
这个是文件块的数据备份个数，一般来说一个设置为3，数字无上限，多的可能无用，且占用空间，少之可能影响数据可靠性。
	
### 第六步：ssh
嗯，在伪分布式下，你要设置ssh的无密码登陆，别以为你安装了ssh就万事大吉了。  
因为在伪分布式下运行时必须启动守护进程，而启动守护进程的前提是已经成功安装SSH。  
NameNode将使用SSH协议启动DataNode进程，伪分布模式下DataNode和NameNode均是本身，所以必须配置SSH localhost的无密码验证。  
	通过ssh localhost测试需要不需要密码登陆？如果需要，执行此两条命令  
	$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa  
    $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  
    $ chmod 0600 ~/.ssh/authorized_keys  
    然后再测试ssh localhost。没问题就继续吧  
    吃惊，配了这个呢，以后运行start-dfs.sh就不用输入密码确认了，作用估计也就是这个吧。。  
    
	
### 第七步：测试可用性
接下来在本地运行下MapReduce作业。关于YARN作业，请参考官网的另一篇教程  
接下来创建分布式文件系统吧  
`bin/hdfs namenode -format`  文件系统创建完毕  
`sbin/start-dfs.sh` 开启NameNode和DataNode  
接下来利用 MapReduce 来创建HDFS文件目录了  
`bin/hdfs dfs -mkdir /user` 创建一个目录成功  
`bin/hdfs dfs -mkdir /user/<username>`再来一个  
` bin/hdfs dfs -put etc/hadoop/*.xml input`  把linux系统中的一个文件复制到分布式系统文件目录下   
运行一个提供的示例  
`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha3.jar grep input output 'dfs[a-z.]+'`  
什么意思呢？   
把分布式的一个文件拷贝到本地linux系统中  
`bin/hdfs dfs -get output 本地目录`   
直接在分布式文件系统中查看某文件   
`bin/hdfs dfs -cat output/*`  
停止NameNode和DataNode的守护进程  
 `sbin/stop-dfs.sh`  
 
## 3脚本
hadoop的脚本全部都要在`bin/hadoop`里面调用,运行没有参数的hadoop会打印所有命令
基本命令格式：`hadoop [--config confdir] [--loglevel loglevel] [COMMAND] [GENERIC_OPTIONS] [COMMAND_OPTIONS]`  
解释：
1. --config confdir  这个参数是使用的配置文件目录，默认不写就是`$ {HADOOP_HOME} / conf`    
2. --loglevel loglevel  这个参数是覆盖的日志等级，可用参数有：
FATAL, ERROR, WARN, INFO, DEBUG, and TRACE. 默认是INFO   
3. COMMAND 命令符，
4. GENERIC_OPTIONS   常规选项
5. COMMAND_OPTIONS  命令选项  
因此，一个完整的命令格式应该是这样的

###　常规选项　GENERIC_OPTIONS
-archives<path1,path2>　　　注,里面是以逗号分割的归档文件列表

	何谓归档：包括归档文件和归档目录：归档文件是特殊的文件档案，这种文件通常具有har的后缀名
				归档目录通常包含元数据（以_index和_masterindex的形式）和数据（part- *）文件







 