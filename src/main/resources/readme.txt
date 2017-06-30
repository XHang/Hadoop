安装成功并启动Hbase
即可通过一下地址访问
192.168.21.253:16010

好了，开发环境配置成功，接下来首先用自带的命令来操作。
PS：所以表名什么名之类的，都要用双引号括起来
创建表
create '表名', 'ColumnFamily名'
  list '表名'  列出表的信息
 插入数据
  推入数据  put  'test', 'row1', 'cf:a', 'value1'
  意思是插入在test表的row1，列是cf:a，值是value1
  注意：Hbase的列包括ColumnFamily的前缀，所以推入数据的列名为cf:a,随便写列名可是要受惩罚的
 查询数据
 	查询所有数据    scan 'test'
 	只获取一行数据  get 'test' ,'row1'
  警用表
  		想修改表，删除表或者更改某些设置需要警用表
  		disable 'test'
  	启用表
  		enable 'test'
  	删除表
  		drop 'test'
  	quit退出shell命令
  	./bin/stop-hbase.sh停止hbase数据库运行
  	
  	连接psftp
  	psftp -l 用户名 192.168.21.253
  	获取远程文件到主机
  	 get /home/Hbase/hbase-1.3.0/lib/metrics-core-2.2.0.jar
  	
  