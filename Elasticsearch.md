# Elasticsearch

## 1. 基础概念

集群和结点

![img](file:///C:\Users\LENOVO\Documents\Tencent Files\2211068912\Image\C2C\MKBSR_MO[H3RO8%AT4Z{06O.png)

1.索引(类似于数据库里面的database)

索引：含有相同属性的文档集合



2.类型(相当于sql中的table)

类型：索引可以定义一个或多个类型，文档必须属于一个类型



3.文档(相当于sql中的一行记录)

文档：文档是可以被索引的基本数据单位



4.分片：每个索引都有多个分片，每个分片是一个Lucene索引

分片的好处: 分摊索引的搜索压力, 分片还支持水平的拓展和拆分以及分布式的操作, 可以提高搜索和其他处理的效率



5.备份：拷贝一份分片就完成了分片的备份

备份的好处: 当主分片失败或者挂掉, 备份就可以代替分片进行操作, 进而提高了es的可用性, 备份的分片还可以进行搜索操作, 以分摊搜索的压力.

## 2.基本用法

ES以RESTFul风格来命名API的, 其API的基本格式如下

http://<ip>:<port>/<索引>/<类型>/<文档id>

ES的动作是以http方法来决定的: 常用的http方法: GET/PUT/POST/DELETE



1.开启head插件

![](C:\Users\LENOVO\Desktop\新图\31.png)

2.在D:\elasticsearch-7.2.0\config\elasticsearch.yml中进行配置，创建master集群

![](C:\Users\LENOVO\Desktop\新图\33.png)

启动elasticsearch

![](C:\Users\LENOVO\Desktop\新图\34.png)

**当看到started时说明启动成功**



![](C:\Users\LENOVO\Desktop\新图\2.png)

3.类似地进行slave1和slave2的创建（需要修改默认端口）

![](C:\Users\LENOVO\Desktop\新图\3.png)





### 2.1 创建索引

创建索引分为: **结构化创建与非结构化创建**

查看索引是否是结构化的方法

**Mappings**是结构化的一个关键词, 其后内容是空的, 说明这个索引是一个非结构化的索引

![1581268582718](C:\Users\LENOVO\AppData\Roaming\Typora\typora-user-images\1581268582718.png)

```json
请求数据的json格式

{

    "settings": {

        "number_of_shards": 3,

        "number_of_replicas": 1

    },

    "mappings": {

            "properties": {

                "name": {

                    "type": "text"

                },

                "country": {

                    "type": "keyword"

                },

                "age": {

                    "type": "integer"

                },

                "date": {

                    "type": "date",

                    "format": "yyyy-MM-dd HH:mm:ss||yyyy:MM:dd||epoch_millis"

                }

            }
        
        }
    
}
```

![](C:\Users\LENOVO\Desktop\新图\5.png)



### 2.2 数据插入

#### 2.2.1 指定文档id插入

使用http中的**put**方法,

插入时输入的ip地址, 如127.0.0.1:9200/people/_doc/1

请求参数依次为:索引名称/类型名称/文档id

请求参数

```json
{
	"name": "陈浩楠",
	"country": "China",
	"age":20,
	"date": "2000-03-27"
}
```

![img](file:///C:\Users\LENOVO\AppData\Roaming\Tencent\Users\2211068912\QQ\WinTemp\RichOle\D[}T$I7OK6X~K`[{VG6K5OT.png)

#### 2.2.2 自动产生文档id插入

使用http中的**post**方法,

插入时输入的ip地址, 如127.0.0.1:9200/people/_doc/

请求参数

```json
{
	"name": "陈浩楠2",
	"country": "China",
	"age":25,
	"date": "1995-03-27"
}
```

![img](file:///C:\Users\LENOVO\AppData\Roaming\Tencent\Users\2211068912\QQ\WinTemp\RichOle\O$LMMIQF5N8R0Y1}A}EG2IH.png)



### 2.3 修改数据

#### 2.3.1 直接修改文档

http方法: **post**方法

请求地址:  127.0.0.1:9200/people/_doc/1/_update

关键词: _update, doc

请求参数

```json
{
	"doc": {
		"name": "陈浩楠3"
	}
}
```

“doc”为关键字, 要修改的文档放在doc中, 实例修改了type为people索引下_doc中id为1 的name属性

![](C:\Users\LENOVO\Desktop\新图\6.png)

![1581269815147](C:\Users\LENOVO\AppData\Roaming\Typora\typora-user-images\1581269815147.png)



#### 2.3.2 脚本修改文档

通过脚本修改的api格式与直接修改的是一致的

http方法: **post**方法

请求地址:  127.0.0.1:9200/people/_doc/1/_update

请求参数

```json
{
	"script": {
		"lang": "painless",
		"inline": "ctx._source.age += 10"
	}
}
```

![](C:\Users\LENOVO\Desktop\新图\7.png)

关键字”script”: 标志以脚本的方式修改文档

“lang”: 表示以何种脚本语言进行修改, “painless”表示以es内置的脚本语言进行修改. 此外es还支持多种脚本语言, 如Python, js等等

“inline”:指定脚本内容 “ctx”代表es上下文, _source 代表文档



此外还有其他的参数设置方式, 例如将参数放到外面

```json
{
	"script": {
		"lang": "painless",
		"inline": "ctx._source.age = params.age; ctx._source.name = params.name",
		"params": {
			"age": 50,
			"name": "jigexio"
		}
	}
}
```

![](C:\Users\LENOVO\Desktop\新图\8.png)



### 2.4 删除文档

#### 2.4.1 删除数据

http方法: **delete**

请求路径127.0.0.1:9200/people/_doc/1

![](C:\Users\LENOVO\Desktop\新图\9.png)

#### 2.4.2 删除索引

1. 可以通过head插件进行删除

2. 通过api删除

http方法: **delete**

链接地址: 127.0.0.1:9200/people