<!--
 * @Autor: cc
 * @Description: 
 * @Version: 1.0
 * @Date: 2020-07-06 10:45:11
 * @LastEditors: cc
 * @LastEditTime: 2020-07-07 15:37:08
--> 
# 安装步骤(win10 jdk1.8)
## 1、安装nodejs
使用node -V
## 2、安装grunt-cli

<img src="./nodejs.jpg">

## 修改elasticSearch配置文件

进入elasticSearch/config目录下修改elasticsearch.yml

````
cluster.name: elasticsearch
http.port: 9200
http.cors.enabled: true 
http.cors.allow-origin: "*"
node.master: true
node.data: true
````
修改完成后进入bin目录双击运行elasticSearch.bat
访问http://localhost:9200
<img src="./9200.jpg">
## 安装Elasticsearch-head
下载解压放在elasticsearch目录下
### 修改Gruntfile.js
````
connect: {
			server: {
				options: {
					hostname: '*',
					port: 9100,
					base: '.',
					keepalive: true
				}
			}
		}
````
### 修改_site/app.js若是本地访问则不需要修改
````
app.App = ui.AbstractWidget.extend({
		defaults: {
			base_uri: null
		},
		init: function(parent) {
			this._super();
			this.prefs = services.Preferences.instance();
			this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";
			if( this.base_uri.charAt( this.base_uri.length - 1 ) !== "/" ) {
				// XHR request fails if the URL is not ending with a "/"
				this.base_uri += "/";
			}
````
### elasticsearch-head-master目录下cmd npm run start 
访问http://localhost:9100
<img src="./9100.jpg">
# 使用postman操作elasticSearch
## 索引操作
### 创建索引
````
PUT http://localhost:9200/suibo
{
    "mappings": {
        "properties": {
            "imageId": {
                "type": "keyword"
            },
            "tagIds": {
                "type": "keyword"
            }
        }
    }
}

````
关于elasticsearch字段类型的介绍参考
https://www.jianshu.com/p/bfef6a890b42
这里我们的需求是根据tagid去做一个相似度匹配，所以keyword更加合适
<h3 style="color:red">text用于全文搜索的, 而keyword用于关键词搜索.</h3>

### 获取索引
GET http://localhost:9200/suibo
### 删除索引
DELETE http://localhost:9200/suibo
## 文档操作
### 新增文档指定id
````
PUT http://elasticsearch-1:9200/suibo/_doc/10
{
    "imageId": "1",
    "tagIds": [1,2,3],
}
返回
{
    "_index": "suibo2",
    "_type": "_doc",
    "_id": "10",
    "found": false
}
````
### 新增文档不指定id
````
POST http://localhost:9200/suibo/_doc
{
    "imageId": "1",
    "tagIds": [1,2,4]
}
返回
{
    "_index": "suibo",
    "_type": "_doc",
    "_id": "I5x_I3MBlJK3llVVb-vO",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 7,
    "_primary_term": 1
}
这里会自动生成id
````
### 新增文档-指定操作类型
````
PUT http://localhost:9200/suibo/_doc/1?op_type=create
{
    "imageId": "1",
    "tagIds": [1,2,4]
}
````
生产过程当中根据业务去添加指定类型，防止数据重复，保证数据唯一性


### 获取文档byId
GET http://localhost:9200/suibo/_doc/1

### 查询文档 terms
````
POST http://localhost:9200/suibo/_search
{
	"query": {
		"terms": {
			"tagIds": [1, 2, 3]
		}
	},
	"sort": [{
		"_score": {
			"order": "desc"
		}
	}]
}
````
### 修改文档 整条数据更新
````
POST http://localhost:9200/suibo/_update/1
{
    "doc": {
        "imageId": "1",
        "tagIds": [1,2,5]
    }
}
````
### 更新指定文档字段
````
POST http://localhost:9200/suibo/_update/1
{
    "script": {
        "source": "ctx._source.tagIds = params.tagIds",//更新表达式
        "params": {
            "tagIds": [5]
        }
    }
}
````
# 业务分析
根据图片的tag标签去做相似推荐
方式一、tagIds类型设置为keyword并且数据存储的格式为数组
<img src="./tagIds_keyword.png">
查询的时候使用terms的方式
````
POST http://localhost:9200/suibo/_search
{
	"query": {
		"terms": {
			"tagIds": [1, 2, 3]
		}
	},
	"sort": [{
		"_score": {
			"order": "desc"
		}
	}]
}
````
但是使用terms的话会自动屏蔽分词器，得出的结果不是很好
方式二：采用分词器，whitespace（es内置的分词器）并且tagIds设置类型为text
1、创建索引的时候指定采用分词器
````
http://localhost:9200/suibo
{
    "mappings":{
        "properties":{
            "imageId":{
                "type":"keyword"
            },
            "tagIds":{
                "type":"text",
                "analyzer":"whitespace"
            }
        }
    }
}
````
2、使用java端将数据库数据清洗同步到es
3、查询查看结果
````
http://localhost:9200/suibo/_search
{
	"query": {
		"match": {
			"tagIds":"2343 351 637 2095 589 2431 12"
		}
	},
	"sort": [{
		"_score": {
			"order": "desc"
		}
	}]
}
````
得到结果
2343 351 637 2095 589 2431 12                           [传入数据]
2343 2408 351 637 1696 1623 2431 2095 125 2324 571      [2343,351,637,2095,2431]
2343 351 1868 2095                                      [2343,351,2095,]
2343 1160 79 637 2356 732 2431 2095 571 1729            [2343,637,2095,2431]
406 2343 2254 1523 1369 168 1790 1171 2431 2095         [2343,2095,2431,]
2343 351 2310 637 1571 2095 906 1498 871 1729           [2343,351,637,2095]
589 637 12 732 1669 381                                 [637,589,12]
864 589 732 2431 2459                                   [589,2431,]
2343 2254 351 589                                       [2343,351,589]
https://blog.csdn.net/pyq666/article/details/99639810
https://blog.csdn.net/zhangkang65/article/details/79163005
https://blog.csdn.net/const_qiu/article/details/89429111


https://www.cnblogs.com/royfans/p/9786429.html