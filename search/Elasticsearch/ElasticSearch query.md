`https://www.cnblogs.com/qdhxhz/p/11493677.html`



基础url：
```
http://localhost:9200/poetry/poem/_search
```

### term

```js
{
	"query": {
		"term": {
			"content": "洛阳" 
		}
	}
}
```
查询content的分词中包含洛阳的文档，查询参数字段不分词而文档字段是分词的。

返回：
```
若问古今兴废事，请君只看洛阳城。
洛阳亲友如相问，一片冰心在玉壶。
```

```js
{
	"query": {
		"term": {
			"content": "洛阳城" 
		}
	}
}
```
返回：
```
若问古今兴废事，请君只看洛阳城。
```

### match
match：模糊匹配，需要指定字段名，但是输入会进行分词，比如"hello world"会进行拆分为hello和world，然后匹配，如果字段中包含hello或者world，或者都包含的结果都会被查询出来，也就是说match是一个部分匹配的模糊查询。查询条件相对来说比较宽松。
```js
{
    "query": {
        "match" : {
            "content" : "洛阳城"
        }
    }
}
```
相当于
```js
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "洛阳城"
        }
    }
}
```


### query_string
默认查询所有的字段，default_field默认为`*`，
```js
{
    "query": {
        "query_string" : {
            "query" : "洛阳"
        }
    }
}
等价于，title、content等字段都会被查询。
{
    "query": {
        "query_string" : {
            "default_field" : "*",
            "query" : "洛阳"
        }
    }
}
```


查询特定字段。

```js
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "洛阳城"
        }
    }
}
```
查询content的分词中包含洛阳城分词的文档，查询参数字段分词文档字段也是分词的。

返回：
```
若问古今兴废事，请君只看洛阳城。
洛阳亲友如相问，一片冰心在玉壶。
```

### 控制查询返回的数量、指定返回的字段
from和size控制分页，_source控制返回的字段
```js
{
    "from":0,
	"size":2,
	"_source":["title","content"],
    "query": {
        "match" : {
            "content" : "洛阳城"
        }
    }
}
```

**显示要的字段、去除不需要的字段**

可以使用通配符*
```js
{
	"_source":{
		"includes": "cont*",
		"excludes": ["author","year*"]
	},
    "query": {
        "match" : {
            "content" : "洛阳城"
        }
    }
}
```

### 排序
```js
{
    "query": {
        "match" : {
            "content" : "洛阳城"
        }
    },
    "sort":[{
		"year":{"order": "desc"}
	}]
}
```

###  范围查询
range: 实现范围查询

include_lower: 是否包含范围的左边界，默认是true

include_upper: 是否包含范围的右边界，默认是true
```js
{
    "query": {
        "range": {
            "year": {
                "from": 1200,
                "to": 1300,
                "include_lower": true,
                "include_upper": false
            }
        }
    }
}
```

### wildcard查询
允许使用通配符* 和 ?来进行查询。也是匹配的文档的分词。
```
* 代表0个或多个字符
? 代表任意一个字符
```

```js
{
    "query": {
        "wildcard": {
             "content": "洛*"
        }
    }
}
```

`?洛`匹配不到"只看洛阳城"`洛?`可以。

```js
{
    "query": {
        "wildcard": {
             "content": "?洛"
        }
    }
}
```

### 高亮搜索结果
```js
{
    "query":{
        "match":{
            "content": "洛阳"
        }
    },
    "highlight": {
        "fields": {
             "content": {}
        }
    }
}
```
返回：
```js
{
    "_index": "poetry",
    "_type": "poem",
    "_id": "3",
    "_score": 0.2876821,
    "_source": {
        "id": "3",
        "title": "芙蓉楼送辛渐",
        "author": "王昌龄",
        "year": 1500,
        "content": "寒雨连江夜入吴，平明送客楚山孤。洛阳亲友如相问，一片冰心在玉壶。"
    },
    "highlight": {
        "content": [
            "<em>洛阳</em>亲友如相问，一片冰心在玉壶。"
        ]
    }
}
```






```js

```
返回：
```

```
