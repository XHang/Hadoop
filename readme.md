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
先决条件：
1. java  ssh软件安装完毕  
2. 看你系统的位数和hadoop安装的位数，两者一定要一致。什么，安装了hadoop后怎么看hadoop位数？  
  	假设你已经安装好hadoop，如图所示  
  	![吔屎啦，图片显示不出来]  
(https://github.com/XHang/Hadoop/blob/Hadoop/hadoop%E4%BD%8D%E6%95%B0.png)   
  	**惨痛的教训。。。**   
 **从解压完毕进入到解压目录讲起**   
 
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
    		<property>
                <name>hadoop.tmp.dir</name>
                <value>file:/home/hadoop-2.8.1/tempFile</value>
                <description>Abase for other temporary directories.</description>
        </property>
	</configuration>
	
解释：配置NameNode结点的URI,包括协议，url，端口  
			集群里面的每一台机器都需要知道NameNode的地址。  
			DataNode结点会先在NameNode上注册，这样它们的数据才可以被使用。  
			独立的客户端程序通过这个URI跟DataNode交互，以取得文件的块列表。  
			可以认为NameNode是中央节点吧，DataNode要注册在NameNode上面。  
			客户端通过和NameNode交互，就可以知道有什么DataNode  
			另：hadoop.tmp.dir是hadoop的临时目录，默认为/tmp/hadoo-hadoop.这个默认目录系统启动会随之删除。  
			建议自己建一个
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
` bin/hdfs dfs -put etc/hadoop/*.xml /input`  把linux系统中的一个文件复制到分布式系统文件根目录input下   
运行一个提供的示例  
`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha3.jar grep input output 'dfs[a-z.]+'`  
什么意思呢？   
把分布式的一个文件拷贝到本地linux系统中  
`bin/hdfs dfs -get output 本地目录`   
直接在分布式文件系统中查看某文件   
`bin/hdfs dfs -cat output/*`  
停止NameNode和DataNode的守护进程  
 `sbin/stop-dfs.sh`  
### 第八步：在伪分布式模式下运行YARN上的MapReduce作业。
YARN是什么？YARN是为了替换老套的，原先的MapReduce job的调度管理。
YARN将 ResourceManagement 资源管理和JobScheduling/JobMonitoring 任务调度监控分开。
用 ResourceManger和 ApplicationMaster这两个进程来管理。
要启用这个的话，需要配置一下内容
PS：可能你会找不到这个文件，只有mapred-site.xml.template这个文件
把它复制到当前目录，并改名为mapred-site.xml
再PS：官网坑爹啊
etc/hadoop/mapred-site.xml:

	<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
	</configuration> 

etc/hadoop/yarn-site.xml:、

	<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        	<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME
        	</value>
    </property>
	</configuration>
	
配置完毕，启动yarn
  $ sbin/start-yarn.sh
  当然关闭就要使用
  $ sbin/stop-yarn.sh
  完成了这个任务以后启动hdfs后还要启动yarn
  注意看下日志启动过程有没有error，反正我是没有。。
  不过差点被几个小时的log骗了。。。
 
 
 
## 3 hadoop命令指南
hadoop命令基本都是这种形式  
`shellcommand [SHELL_OPTIONS] [COMMAND] [GENERIC_OPTIONS] [COMMAND_OPTIONS]`
关键点：
1. shellcommand   这个是指定调用的哪个项目的命令。   
	比如，Hadoop的hadoop   ，HDFS的hdfs ,  YARN 的yarn  
2. SHELL_OPTIONS  执行java命令之前shell命令的选项，如：  
	--buildpaths  启用开发版的jar包    
	--config confdir  覆盖默认的配置目录，默认是：$HADOOP_HOME/etc/hadoop    
	--daemon mode  守护线程模式  如果这个命令支持守护线程，则以适当形式运行，不支持守护线程的命令，你写了也没用  
	--debug  启用shell级别的日志记录  
	--help  帮助信息。  
	--hosts  当workers使用时，使用主机名列表覆盖workers的文件，如果workers没有，则选项忽略。  
	--loglevel loglevel  覆盖的日志级别，可以用的日志级别有 FATAL, ERROR, WARN, INFO, DEBUG, and TRACE. Default is INFO.   
	--workers  如果可能，在workers文件中所有主机执行该命令。  
	  
3. COMMAND  要执行的命令  
4. GENERIC_OPTIONS  这个是很多命令都支持的通用选项,不是所有命令都有这个。。    
	-archives <逗号分隔的档案列表>   在计算机上指定要解除归档的档案列表，只适用于 job.   
	-conf <configuration file>  指定应用的配置文件  
	-D <property>=<value>  使用给定属性的值  
	-files <逗号分隔的文件列表>指定要复制到map reduce cluster的逗号分隔文件。 仅适用于 job. 。  
	-fs <file:///> or <hdfs://namenode:port>  指定要使用的默认文件系统URL。 覆盖配置中的“fs.defaultFS”属性。  
	-jt <local> or <resourcemanager:port> 指定ResourceManager。 仅适用于 job.   
	-libjars <comma seperated list of jars> 指定在类路径中的jar文件列表，用逗号分隔。 仅适用于jod。  

### hadoop通用的命令
介绍：所有这些命令都是从hadoop shell命令执行的。 它们被分解成用户命令和管理命令  
1. archive ，详细看档案那一章
2.  checknative
	`hadoop checknative [-a] [-h]`
	-a  检查所以库文件是否可用
	
	
   5.COMMAND_OPTIONS  命令选项，其他不解释   
### hadoop 档案详解  
hadoop的档案是一个特殊的档案，用于将所有小文件整合一个文件，整合后的文件还可以通过访问每一个小文件，并且作为mapreduce任务的输入，其实类似于压缩，不过hadoop创建归档文件会消耗和源文件一样的硬盘空间，也就是说，这，丫，没，有，压，缩，功，能！
	特性如下：
1. hadoop将归档文件映射到文件系统目录。  
2. hadoop的归档文件有har的后缀名  
3. hadoo归档目录包含许多metadata（就是形如_index 和_masterindex）和数据文件（part-*）  
4. _index文件包含了归档文件的名称和数据文件（part-*）的位置  
接下来讲命令了  
1. 怎么创建一个档案?  
用法：`Usage: hadoop archive -archiveName name -p <parent> [-r <replication factor>] <src>* <dest>`  
解释：-archiveName是要归档的名称，举个栗子，比如说：`勇者斗恶龙nds.har`。看到没，这个名称必须带有har扩展名
			parent是文件应放到哪个位置的相等路径，比如说:`-p /勇者斗恶龙nds/bar a/b/c e/f/g`,这里的`/勇者斗恶龙nds/bar`是父路径，后面的是相对路径
			划重点，这个命令是一个Map / Reduce作业，你需要一个map reduce集群来运行这个
			还有-r表示复制的要素，默认不写的话使用要素3
			如果你只想归档一个目录，使用这个命令足矣：`hadoop archive -archiveName zoo.har -p /foo/bar -r 3 /outputdir`

## hadoop的伪分布式和完全分布式区别
独立式直接对象文件系统进行操作，而且科学的情况下不可能用到这个形式，故此不讲
伪分布式嘛，就是namenode和datanode在一台机器的不同线程里运行。  
完全分布式，那就是一台机器做namenode，下辖若干个机器装载datanode
此外从分布式应用的角度来说，集群中的结点由一个JobTracker和若干个TaskTracker组成，JobTracker负责任务的调度，TaskTracker负责并行执行任务。  
TaskTracker必须运行在DataNode上，这样便于数据的本地计算。JobTracker和NameNode则无须在同一台机器上。

## hadoop的问题
1. 无论执行什么命令，总有Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	大多是因为你下载和安装的hadoop是编译好的，而且还是32位，但是你的机子是64位的。所以你需要自己去下载源码到本机编译。
	`http://dl.bintray.com/sequenceiq/sequenceiq-bin/`这个网站已经有64位的，编译好的了。
	还有可能是。。。

2：
理论上，格式化后开启hdfs后这样就可以用ip+端口访问hadoop的NameNode网页  
默认情况下是本机ip+9870.  
如果不行的话，看下输出日志，别被控制台的out日志骗了，日志文件应该是log格式的
我那个之所以不能启动，是因为我只配了并且创建了hadoop.tmp.dir目录。但是里面深层的一个目录没创建，结果hadoop启动时找不到这个目录，就狗带了
启动了这个目录后，还是不能运行hadoop，看了一下日志又是告诉我namenode没有格式化，好吧，我再格式化一次，又来报错，说什么
`URI has an authority component`
搞事情啊搞事情。最后给hdfs-site.xml配了

		<property>    
        <name>dfs.namenode.name.dir</name>    
        <value>${hadoop.tmp.dir}/dfs/name</value>  
    </property>    
    <property>    
        <name>dfs.datanode.data.dir</name>    
        <value>${hadoop.tmp.dir}/dfs/data</value>  
    </property>
  指定一个文件路径。当然hadoop.tmp.dir要替换为你的临时文件夹路径。别用这种高端大气的变量名。
  嗯，这样启动配会有一个警告，大概是说你只配一个路径，要小心数据的事balabala的。不管
  最后访问http://localhost:50070。。ok，显示出来了！
  PS:活用下JPS命令，看下那些服务未启动
  







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
				







 