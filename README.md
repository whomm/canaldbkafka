# canaldbkafka
## 简介
canaldbkafka是连接canal和kafka的一个中间件。目的是实现数据库某个表格数据变更转变成消息流的形式，以便后续业务消费kafak的消息流。
## 消息的类型
canal的binlog 会被解析成以下3中类型的消息。其他的类型被过滤掉了。
### insert

```
{
    "data": {
        "need_sub": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "order_description": {
            "type": "varchar(1024)",
            "updated": true,
            "value": ""
        },
        "pay_amount": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "pay_order": {
            "type": "varchar(30)",
            "updated": true,
            "value": ""
        }
    },
    "type": "insert"
}
```

### delete
```
{
    "data": {
        "need_sub": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "order_description": {
            "type": "varchar(1024)",
            "updated": true,
            "value": ""
        },
        "pay_amount": {
            "type": "int(11)",
            "updated": true,
            "value": "0"
        },
        "pay_order": {
            "type": "varchar(30)",
            "updated": true,
            "value": ""
        }
    },
    "type": "delete"
}
```
### update
data对象是各字段类型、是否被更新、值。olddata对象是之前的状态。

```
{
    "data": {
        "Quota": {
            "type": "tinyint(4)",
            "updated": false,
            "value": "0"
        },
        "ReqAmount": {
            "type": "int(11)",
            "updated": true,
            "value": "100"
        }
    },
    "olddata": {
        "Quota": {
            "type": "tinyint(4)",
            "updated": false,
            "value": "0"
        },
        "ReqAmount": {
            "type": "int(11)",
            "updated": false,
            "value": "0"
        }
    },
    "type": "update"
}
```

## 使用说明
### 编译安装

```
mvn compile

mvn package

ll target/canal-dbkafka   #可部署
total 0
drwxr-xr-x   5 xxx  staff   170B 12 21 21:26 bin
drwxr-xr-x   3 xxx  staff   102B 12 21 21:26 conf
drwxr-xr-x  24 xxx  staff   816B 12 21 21:26 lib
drwxr-xr-x   2 xxx  staff    68B 12 21 21:26 logs

ll target/canal-dbkafka/bin  #startmy.sh为启动示例
-rwxr-xr-x  1 xxx  staff   271B 12 21 21:26 startmy.sh
-rwxr-xr-x  1 xxx  staff   2.5K 12 21 21:26 startup.sh
-rwxr-xr-x  1 xxx  staff   1.0K 12 21 21:26 stop.sh

```
### 启动说明

```
#!/bin/bash

current_path=`pwd`
case "`uname`" in
    Linux)
        bin_abs_path=$(readlink -f $(dirname $0))
        ;;
    *)
        bin_abs_path=`cd $(dirname $0); pwd`
        ;;
esac
cd ${bin_abs_path} && ./startup.sh testdb thetable 127.0.0.1:2181 127.0.0.1:9092
```
1. testdb 是canal配置的destination
2. thetable kafka的具体topic
3. 127.0.0.1:2181 是canal配置HA 对应的zookeeper的地址
4. 127.0.0.1:9092  是kafka的地址


### 使用注意事项
1. mysql binlog模式设置为row模式
2. 为了保证数据库消息的顺序性，将消息存储kafka的时候组件采用了同步的方式
3. canal 必须配置zookeeper ha的模式 https://github.com/alibaba/canal/wiki/AdminGuide#ha%E6%A8%A1%E5%BC%8F%E9%85%8D%E7%BD%AE
4. 之前使用针对的是数据库中的一个表在canal配置中已经过滤所以消息中没有表名 可以说是个设计的缺陷。




