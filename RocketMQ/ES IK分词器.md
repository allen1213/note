

ES中的分词效果很差，需要用其他的分词器对搜索的内容进行分词，中文分词插件，这里使用IK，也可以用 [smartcn](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-smartcn.html)



##### IK [Github地址](https://github.com/medcl/elasticsearch-analysis-ik/tags)

下载，注意版本要和ES一致

```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.1/elasticsearch-analysis-ik-7.8.1.zip
```



下载完成后将解压过后的文件夹复制到ES的 `/usr/share/elasticsearch/plugins` 目录下，若使用docker安装的ES，可以使用 `-v` 参数映射



##### 测试分词效果

analyzer=chinese 默认分词：

```
curl http://127.0.0.1:9200/_analyze?analyzer=chinese&pretty=true&text=我是程序员

```



analyzer=ik_smart 最少切分：

```
http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=我是程序员
```



analyzer=ik_max_word 最细切分：

```
http://127.0.0.1:9200/_analyze?analyzer=ik_max_word&pretty=true&text=我是程序员
```







重新启动 Elastic，新建一个名为order的 Index，有source/payMethod/desc三个字段，三个字段都是中文，类型都是文本text，所以需要指定中文分词器，不能使用默认的英文分词器，**Elastic 的分词器称为** [**analyzer**](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

```json
POST http://127.0.0.1:9200/order

//添加json参数，设置mapping
{
  "mappings": {
    "doc": {
      "dynamic": true,
      "properties": {
        "source": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "payMethod": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}
```



`analyzer`是字段文本的分词器，`search_analyzer`是搜索词的分词器，**ik_max_word/ik_smart分词器由ik提供**，可以对文本进行最大数量的分词