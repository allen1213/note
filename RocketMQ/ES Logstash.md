使用 Logstash [官网](https://www.elastic.co/guide/en/logstash/7.8/index.html) 同步 Mysql 数据到 ElasticSearch



### 安装 Logstash 



```shell
docker pull logstash:7.8.1
```



或者可以使用dockerfile 构建logstash镜像

```shell
FROM logstash:7.8.1
#安装input插件
RUN logstash-plugin install logstash-input-jdbc
#安装output插件
RUN logstash-plugin install logstash-output-elasticsearch
#容器启动时执行的命令.(CMD 能够被 docker run 后面跟的命令行参数替换)
CMD ["-f", "/usr/share/logstash/pipeline/logstash.conf"]
```

```shell
docker build -t logstash:7.8.1 .
```





下面通过 Logstash 定时扫描数据库来增量同步数据，首先启动elasticsearch

```shell
docker start es
```







### 准备工作



启动mysql

```shell
docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql
```



新建数据库脚本

```sql
CREATE TABLE user  (
  `id` int(11) NOT NULL,
  `name` varchar(50) NOT NULL,
  `age` int(11) NOT NULL,
  `createtime` datetime(0) NOT NULL,
  `lastupdatetime` datetime(0) NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) 
 
INSERT INTO `user` VALUES(1,"bobo",18,Now(),Now())
INSERT INTO `user` VALUES(2,"allen",18,Now(),Now())
 
SELECT * from `user`
```



创建索引 `user_logstash`

```json
PUT http://192.168.49.133:9200/user_logstash/
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
    "dynamic": true,
    "properties": {
      "id": {
        "type": "integer"
      },
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "createTime": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "lastUpdateTime": {
        "type": "date",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```







### 配置 LogStash

```shell
# 创建挂载目录
mkdir -p /docker/logstash/pipeline
mkdir -p /docker/logstash/config
mkdir -p /docker/logstash/conf.d
mkdir -p /docker/logstash/app
mkdir -p /docker/logstash/myfile
mkdir -p /docker/logstash/logs

# 上传连接mysql的jar包，并移动到 /docker/logstash/myfile 下
rm mysql-connector-java-8.0.15.jar /docker/logstash/myfile
```



在 /docker/logstash/config 目录下新建longstash配置文件 logstash.yml

```yml
path.config: /usr/share/logstash/conf.d/*.conf
path.logs: /var/log/logstash
```



在 /docker/logstash/conf.d 目录下新建文件 user_logstash.conf ，用来配置同步 mysql 数据到 es

```
input {
  jdbc {
    #数据库连接信息
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://192.168.49.133:3306/test"
    jdbc_user => "root"
    jdbc_password => "root"

	#连接mysql jar包
    jdbc_driver_library => "/usr/share/logstash/myfile/mysql-connector-java-8.0.15.jar"

	#开启分页
    jdbc_paging_enabled => true
    jdbc_page_size => 1000

	#执行获取数据库数据的sql文件
    statement_filepath => "/usr/share/logstash/conf.d/user_logstash.sql"
    #statement => "SELECT * FROM user WHERE lastupdatetime >= :sql_last_value"

	#每5分钟执行一次
    schedule => "*/5 * * * *"


    #开启记录上次追加的时间
    use_column_value => true
    
    #以 lastupdatetime 字段作为更新的依据，当mysql中数据变化时会同步到es
    tracking_column => "lastupdatetime"
    tracking_column_type => "timestamp"

    
    #是否清除 last_run_metadata_path 中记录的值
    clean_run => false

    #数据库字段名大小写转换
    lowercase_column_names => false

  }
}


output {
    elasticsearch {

        # ES地址，集群用 ，隔开
        hosts => ["192.168.49.133:9200"]

        # 同步的索引名
        index => "user_logstash"

        # 设置docId和数据库相同
        document_id => "%{id}"       
    }
    stdout {
        # JSON格式输出日志
        codec => json_lines
    }
}
```



在 /docker/logstash/conf.d 目录下新建文件 user_logstash.sql

```sql
select * from user where lastupdatetime >= :sql_last_value
```



启动logstash，会自动加载  /docker/logstash/conf.d 目录下所有 *.conf 文件

```shell
docker run -it --privileged=true -m 512m \
-v /docker/logstash/pipeline/:/usr/share/logstash/pipeline/ \
-v /docker/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /docker/logstash/conf.d:/usr/share/logstash/conf.d/ \
-v /docker/logstash/myfile:/usr/share/logstash/myfile/ \
-v /docker/logstash/logs:/usr/share/logs/ \
--name lg \
-d logstash:7.8.1
```





启动完成后，使用`docker logs -f lg` 或者请求 `PUT http://192.168.49.133:9200/user_logstash/` 查看数据是否同步到es







### 同步配置文件/docker/logstash/conf.d/user_logstash.conf  示例

```
input {
 stdin { }
    jdbc {
        #注意mysql连接地址一定要用ip，不能使用localhost等
        jdbc_connection_string => "jdbc:mysql://10.11.0.100/uckefu?serverTimezone=UTC"
        jdbc_user => "xxx"
        jdbc_password => "xxx"
        #这个jar包的地址是容器内的地址
        jdbc_driver_library => "/opt/kibana/config/mysql-connector-java-6.0.6.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_paging_enabled => "true"
        jdbc_page_size => "50000"
        statement => "SELECT * FROM uk_chat_message where createtime> :sql_last_value order by createtime asc"
        schedule => "*/15 * * * *"
 
	#处理中文乱码问题
      	codec => plain { charset => "UTF-8"}
	#是否记录上次运行的结果
        record_last_run => true
        #记录上次运行结果的文件位置
        last_run_metadata_path => "/opt/kibana/config/station_parameter.txt"
        #是否使用数据库某一列的值，
        use_column_value => true
        tracking_column => "createtime"
        #numeric或者timestamp
        tracking_column_type => timestamp
        
        #如果为true则会清除 last_run_metadata_path 的记录，即重新开始同步数据
        #clean_run => false
 
    }
 }
 
 filter {
    ruby {
        code => "event.timestamp.time.localtime"
    }
}
 
 output {
     stdout {
        codec => json_lines
    }
    elasticsearch {
        #注意mysql连接地址一定要用ip，不能使用localhost等
        hosts => "10.11.1.50:9200"
        index => "uckefu"
        document_type => "message"
        document_id => "%{id}"
    }
}
```





### 同步多张表数据到ES

同步多表和同步单表是一样的，只是在`input {}` 块中多添加一个 `jdbc{}` 块，然后在 `output {}` 中做判断同步到不同的ES索引中即可



`/docker/logstash/conf.d` 目录下新建文件，模板如下

```
input {
	jdbc {
		#第一张表，user
	}
	
	jdbc {
		#第二张表,dept
	}
	
}

filter {
	#...
}

output {

	if [type]=="user" {
		#es...
	}
	
	if [type]=="dept" {
		#...
	}

}
```









### 多文件方式同步ES数据

 logstash 可以借助 **pipelines 机制同步多个表**，只需要在`/docker/logstash/conf.d` 目录下写多个配置文件即可



假设我们有两个表 table1 和 table2，对应两个配置文件 mysql_user.conf 和 mysql_member.conf，只需要修改sql语句，es索引和文档类型 



 `/docker/logstash/conf.d` 下新建 mysql_user.conf

```
input {
  jdbc {
    jdbc_driver_library => "/usr/local/sql/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://192.168.1.103:3306/test_elk"
    jdbc_user => "root"
    jdbc_password => "root"
    schedule => "* * * * *"
    statement => "SELECT * FROM user WHERE update_time >= :sql_last_value"
    use_column_value => true
    tracking_column_type => "timestamp"
    tracking_column => "update_time"
    last_run_metadata_path => "syncpoint_table"
  }
}


output {
    elasticsearch {
        # ES的IP地址及端口
        hosts => ["192.168.128.130:9200", "192.168.128.130:9201"]
        # 索引名称 可自定义
        index => "user"
        # 需要关联的数据库中有有一个id字段，对应类型中的id
        document_id => "%{id}"
        document_type => "user"
    }
    stdout {
        # JSON格式输出
        codec => json_lines
    }
}
```



 `/docker/logstash/conf.d` 下新建 mysql_member.conf

```
input {
  jdbc {
    jdbc_driver_library => "/usr/local/sql/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://192.168.1.103:3306/test_elk"
    jdbc_user => "root"
    jdbc_password => "root"
    schedule => "* * * * *"
    statement => "SELECT * FROM member WHERE update_time >= :sql_last_value"
    use_column_value => true
    tracking_column_type => "timestamp"
    tracking_column => "update_time"
    last_run_metadata_path => "syncpoint_table"
  }
}


output {
    elasticsearch {
        # ES的IP地址及端口
        hosts => ["192.168.128.130:9200", "192.168.128.130:9201"]
        # 索引名称 可自定义
        index => "member"
        # 需要关联的数据库中有有一个id字段，对应类型中的id
        document_id => "%{id}"
        document_type => "member"
    }
    stdout {
        # JSON格式输出
        codec => json_lines
    }
}
```



`/docker/logstash/config` 下新建  pipelines.yml 

```yml
- pipeline.id: table1
  path.config: "/usr/local/sql/mysql_user.conf"
- pipeline.id: table2
  path.config: "/usr/local/sql/mysql_member.conf" 
```





































































































