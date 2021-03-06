# Cluster

logkit cluster

## 架构

1. master <-> slave 结构，slave向所有master定时发送心跳(注册)
2. master可以有多个，master无状态
3. 通过tag标签来区分不同的slave，以执行不同的任务，以便从逻辑上对所有slave进行区分

## 主要功能

* 访问master可以查看所有logkit的运行状态
* 通过master可以部署所有logkit（或指定某个logkit）执行日志收集任务（增、删、改）

## API

### Master API -- 注册slave

```
POST /logkit/cluster/register
{
  "url":"slave_url",
  "tag":"first"
}
```

###  Master API -- 获取slave列表

```
GET /logkit/cluster/slaves?tag=tagvalue
```

返回

```
[{"url":"http://10.10.0.1:1222","tag":"tag1","status":"ok","last_touch":<rfc3339 string>}]
```

* status 状态有三种， ok，表示正常，30s内有联系；bad，表示1分钟内有联系；lost，表示超过1分钟无心跳
* 可以通过tag url参数获取一类tag的列表


###  Master API -- 获取runner状态列表

```
GET /logkit/cluster/status?tag=tagvalue
```

返回

```
Content-Type: application/json

{
 "url1":{
    "status":{
       "runner1": {
          "name":"runner1",
          "logpath":"/your/log/path1",
          "readDataSize": <读取数据的bytes大小>.
          "readDataCount":<读取数据条数>,
          "elaspedtime":<总用时>,
          ... //此处省略，与get status
       }
    },
    "tag":<tag>,
    "error":<error info>
  }
  ...
}

```
如果 tag 为空，则对所有的 slave 进行操作

###  Master API -- 获取runner 配置文件

```
GET /logkit/cluster/configs?tag=tagvalue
```

返回

```
Content-Type: application/json

{
 "url1":{
    "configs":{
        config_file_path: {
            "name": xxx,
            "batch_interval": xx
            ....
            以下略过，此处为 logkit runner 的配置文件
        },
        ....
    },
    "tag":<tag>,
    "error":<error info>
  }
  ...
}

```
如果 tag 为空，则对所有的 slave 进行操作

### Slave API -- 注册（修改）标签

```
POST /logkit/cluster/tag
{
  "tag":"first"
}
```

修改slave 的标签，slave向master发送心跳时通过register接口顺便做修改

### Master API -- 为 tag=tagValue 的 Slave 添加 runner

```
POST /logkit/cluster/configs/<runnerName>?tag=tagValue
{
    "name": "runner-xxx",
    ....
    以下略过，request body 为 logkit runner 的配置文件
}
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```

### Master API -- 为 tag=tagValue 的 Slave 更新 runner

```
PUT /logkit/cluster/configs/<runnerName>?tag=tagValue
{
    "name": "runner-xxx",
    ....
    以下忽略，request body 为 logkit runner 的配置文件
}
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```

### Master API -- 为 tag=tagValue 的 Slave 删除 runner

```
DELETE /logkit/cluster/configs/<runnerName>?tag=tagValue
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```

### Master API -- 为 tag=tagValue 的 Slave 停止 runner

```
POST /logkit/cluster/configs/<runnerName>/stop?tag=tagValue
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```

### Master API -- 为 tag=tagValue 的 Slave 启动 runner

```
POST /logkit/cluster/configs/<runnerName>/start?tag=tagValue
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```

### Master API -- 为 tag=tagValue 的 Slave 重置 runner

```
POST /logkit/cluster/configs/<runnerName>/reset?tag=tagValue
```

如果 tag 为空，则对所有的 slave 进行操作

如果有错误，response body 中为错误信息:

若返回码为 400, 则 response body 为一条错误信息
若返回码为 400, 则错误信息格式为:
```
{"error": <error message>}
```
若返回码为 503, 则错误信息格式为:
```
{
    <slave_url>: {
        "url": <slave_url>,
        "tag": <slave_tag>,
        "mgrType": <操作类型('add', 'update', 'delete', 'start', 'stop', 'reset')>,
        "error": <错误信息>
    }
    ...
}
```
