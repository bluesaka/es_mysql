## 基于Canal同步MySQL数据到ElasticSearch

### MySQL配置

#### 查看binlog是否开启  
 
```
show variables like '%log_bin%'
``` 

#### 查看my.cnf配置

```
mysql --help | grep my.cnf
```

#### 配置my.cnf (如/etc/mysql/my.cnf)，加入以下内容

```
[mysqld]
log-bin = mysql-bin #开启binlog
binlog-format = ROW #选择row模式
server_id = 1 #配置mysql replication需要定义，不能和canal的slaveId重复
```

#### 重启MySQL

#### 查看binlog日志状态

```
# 可以查看master数据库当前正在使用的二进制日志及当前执行二进制日志位置
show master status
```

#### 其他命令

```
* flush logs   #刷新之后会新建一个新的Binlog日志
* reset master    #清空日志文件
* show binlog events in 'mysql-bin.000002' limit 0,100      #查看binlog内容   
* 修改mysql数据文件夹权限  sudo chmod -R a+rwx /usr/local/mysql/data
* 终端查看内容  mysqlbinlog  /usr/local/mysql/data/mysql-bin.000002 | more
```

### MySQL的Canal账号授权

```
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

### 相关问题

如果修改了表名、表结构等，且es服务配置了相关的rule、filter，可能会无法实时同步到es。

可尝试启动多个es服务，不同es服务有不同的配置，修改表后，相关的es服务情况或修改master.info里的内容，然后重启服务。

若要重建索引，重新dump数据，可清除服务中的master.info里的内容，然后重启服务