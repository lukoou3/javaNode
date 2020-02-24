# ElasticSearch java api


## 测试

### 依赖
```xml
<properties>
    <es-version>6.5.3</es-version>
</properties>

<dependencies>
    <!-- es客户端的依赖 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>transport</artifactId>
        <version>${es-version}</version>
    </dependency>

    <!--  elasticsearch依赖2.x的log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.10.0</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.10.0</version>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.10.0</version>
    </dependency>
</dependencies>
```

### pojo
```java
package com.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String id;
    private String name;
    private int age;
    private String desc;
}
```

### IndexTest
```java
public class IndexTest {
    private TransportClient client;

    @Before
    public void setUp() throws UnknownHostException {
        //注意，如果你使用一个不同于“ elasticsearch”的集群名称，你必须设置集群名称
        Settings settings = Settings.builder().put("cluster.name", "My Elasticsearch").build();
        client = new PreBuiltTransportClient(settings);

        //用来指定集群中的节点,TCP/IP协议，es服务器的端口号是：9300； HTTP协议，端口号是9200
        TransportAddress hadoop01 = new TransportAddress(InetAddress.getByName("hadoop01"), 9300);
        TransportAddress hadoop02 = new TransportAddress(InetAddress.getByName("hadoop02"), 9300);
        TransportAddress hadoop03 = new TransportAddress(InetAddress.getByName("hadoop03"), 9300);

        client.addTransportAddresses(hadoop01, hadoop02, hadoop03);
    }

    @Test
    public void createTest(){
        CreateIndexResponse indexResponse = client.admin()
                .indices()
                //设置要创建的索引库的名称
                .prepareCreate("blog")
                //执行操作
                .get();

        System.out.println(indexResponse.toString());
    }

    @Test
    public void setmappingTest(){
        Map<String,Object> mapping = new LinkedHashMap<>();
        Map<String,Object> properties = new LinkedHashMap<>();

        Map<String,Object> field = new LinkedHashMap<>();
        field.put("type","long");
        field.put("store", true);
        properties.put("id", field);

        field = new LinkedHashMap<>();
        field.put("type","text");
        field.put("store", true);
        field.put("analyzer", "ik_max_word");
        properties.put("title", field);

        field = new LinkedHashMap<>();
        field.put("type","text");
        field.put("store", true);
        field.put("analyzer", "ik_max_word");
        properties.put("content", field);

        mapping.put("properties", properties);


        //使用client设置mapping信息
        client.admin()
                .indices()
                //设置要设置mapping的索引库的名称
                .preparePutMapping("blog")
                //设置type的名称。Elasticsearch 在6.0版本以后，一个index下，只允许创建一个type，不允许存在多个type。并且在官网提供信息，7.0以后不再使用type。
                .setType("article")
                //type的定义
                .setSource(mapping)
                //执行操作
                .get();
    }

    @Test
    public void createIndexWithMapping() throws Exception {
        //2、XContentBuilder对象描述一个json数据
        /*{
            "mappings":{
            "article":{
                "properties":{
                    "id":{
                        "type":"long",
                                "store":true
                    },
                    "title":{
                        "type":"text",
                                "store":"true",
                                "analyzer":"ik_max_word"
                    },
                    "content":{
                        "type":"text",
                                "store":"true",
                                "analyzer":"ik_max_word"
                    }
                }
            }
        }
        }*/
        XContentBuilder builder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("mappings")
                .startObject("article")
                .startObject("properties")
                .startObject("id")
                .field("type","long")
                .field("store", true)
                .endObject()
                .startObject("title")
                .field("type", "text")
                .field("store", true)
                .field("analyzer", "ik_max_word")
                .endObject()
                .startObject("content")
                .field("type", "text")
                .field("store", true)
                .field("analyzer", "ik_max_word")
                .endObject()
                .endObject()
                .endObject()
                .endObject()
                .endObject();
        //3、使用client创建索引库，设置mapping信息
        client.admin()
                .indices()
                .prepareCreate("blog2")
                //设置mappings信息
                .setSource(builder)
                .get();
    }

    @Test
    public void deleteIndex() {
        client.admin()
                .indices()
                .prepareDelete("blog")
                .get();
    }

    @After
    public void cleanUp() {
        if (client != null) {
            client.close();
        }
    }


}
```


### DocumentTest
```java
public class DocumentTest {
    private TransportClient client;
    private ObjectMapper objectMapper = new ObjectMapper();

    @Before
    public void setUp() throws UnknownHostException {
        //注意，如果你使用一个不同于“ elasticsearch”的集群名称，你必须设置集群名称
        Settings settings = Settings.builder().put("cluster.name", "My Elasticsearch").build();
        client = new PreBuiltTransportClient(settings);

        //用来指定集群中的节点,TCP/IP协议，es服务器的端口号是：9300； HTTP协议，端口号是9200
        TransportAddress hadoop01 = new TransportAddress(InetAddress.getByName("hadoop01"), 9300);
        TransportAddress hadoop02 = new TransportAddress(InetAddress.getByName("hadoop02"), 9300);
        TransportAddress hadoop03 = new TransportAddress(InetAddress.getByName("hadoop03"), 9300);

        client.addTransportAddresses(hadoop01, hadoop02, hadoop03);
    }

    @Test
    public void setmapping(){
        Map<String,Object> mapping = new LinkedHashMap<>();
        Map<String,Object> properties = new LinkedHashMap<>();

        Map<String,Object> field = new LinkedHashMap<>();
        field.put("type","long");
        field.put("store", false);
        properties.put("id", field);

        field = new LinkedHashMap<>();
        field.put("type","text");
        field.put("store", true);
        field.put("analyzer", "ik_max_word");
        field.put("search_analyzer", "ik_max_word");
        properties.put("name", field);

        field = new LinkedHashMap<>();
        field.put("type","long");
        field.put("store", true);
        properties.put("age", field);

        field = new LinkedHashMap<>();
        field.put("type","text");
        field.put("store", true);
        field.put("analyzer", "ik_max_word");
        field.put("search_analyzer", "ik_max_word");
        properties.put("desc", field);

        mapping.put("properties", properties);


        //使用client设置mapping信息
        client.admin()
                .indices()
                //设置要设置mapping的索引库的名称
                .preparePutMapping("twitter")
                //设置type的名称。Elasticsearch 在6.0版本以后，一个index下，只允许创建一个type，不允许存在多个type。并且在官网提供信息，7.0以后不再使用type。
                .setType("user")
                //type的定义
                .setSource(mapping)
                //执行操作
                .get();
    }

    @Test
    public void prepareIndex() throws JsonProcessingException {
        //prepareIndex可以新增和替换
        User user = new User("1", "司马光", 20, "若问古今兴废事，请君只看洛阳城。");
        String json = objectMapper.writeValueAsString(user);
        IndexResponse response = client.prepareIndex("twitter", "user", "1")
                .setSource(json, XContentType.JSON)
                .get();
        System.out.println(response);
        System.out.println(response.status());
    }

    @Test
    public void prepareGet() {
        GetResponse response = client.prepareGet("twitter", "user", "1")
                .get();
        System.out.println(response);
        System.out.println(response.getSource());
    }

    @Test
    public void prepareUpdate() {
        //prepareUpdate 只更新相应的字段，不存在id会抛出异常
        Map<String,String> map = new LinkedHashMap<>();
        map.put("title", "a");
        UpdateResponse response = client.prepareUpdate("twitter", "user", "1")
                .setDoc(map)
                .get();
        System.out.println(response);
    }

    @Test
    public void prepareUpdate1() {
        UpdateResponse response = client.prepareUpdate("twitter", "user", "2")
                .setDoc("name", "王安石", "age", 15)//key value,key value
                .get();
        System.out.println(response);
    }

    @Test
    public void prepareUpdate2() {
        UpdateResponse response = client.prepareUpdate("twitter", "user", "1")
                .setScript(new Script("ctx._source.age += 5"))
                .get();
        System.out.println(response);
    }

    @Test
    public void prepareDelete() {
        DeleteResponse response = client.prepareDelete("twitter", "user", "1")
                .get();
        System.out.println(response);
    }

    @Test
    //批量操作
    public void prepareBulk() throws JsonProcessingException {
        User user = new User("10", "张溥", 20, "五人墓碑记");
        String json = objectMapper.writeValueAsString(user);
        BulkResponse bulkResponse = client.prepareBulk()
                .add(new IndexRequest("twitter", "user", "10").source(json, XContentType.JSON))
                .add(new UpdateRequest("twitter", "user", "2").doc("name", "韩愈"))
                .get();
        System.out.println(bulkResponse);
    }

    @Test
    public void prepareMultiGet(){
        long start = System.currentTimeMillis();
        MultiGetResponse multiGetItemResponses = client.prepareMultiGet()
                .add("twitter", "user", "1")// get by a single id
                .add("twitter", "user", "2", "10")//or by a list of ids for the same index
                .add("bank", "account", "25", "26", "27")//you can also get from another index
                .get();

        for (MultiGetItemResponse itemResponse : multiGetItemResponses) {
            GetResponse response = itemResponse.getResponse();
            //you can check if the document exists
            if (response.isExists()) {
                String json = response.getSourceAsString();
                System.out.println(json);
            }
        }
        System.out.println(System.currentTimeMillis() - start);
    }

    @After
    public void cleanUp() {
        if (client != null) {
            client.close();
        }
    }

}
```


### QueryTest
```java
public class QueryTest {
    private TransportClient client;
    private ObjectMapper objectMapper = new ObjectMapper();

    @Before
    public void setUp() throws UnknownHostException {
        //注意，如果你使用一个不同于“ elasticsearch”的集群名称，你必须设置集群名称
        Settings settings = Settings.builder().put("cluster.name", "My Elasticsearch").build();
        client = new PreBuiltTransportClient(settings);

        //用来指定集群中的节点,TCP/IP协议，es服务器的端口号是：9300； HTTP协议，端口号是9200
        TransportAddress hadoop01 = new TransportAddress(InetAddress.getByName("hadoop01"), 9300);
        TransportAddress hadoop02 = new TransportAddress(InetAddress.getByName("hadoop02"), 9300);
        TransportAddress hadoop03 = new TransportAddress(InetAddress.getByName("hadoop03"), 9300);

        client.addTransportAddresses(hadoop01, hadoop02, hadoop03);
    }

    @Test
    public void findById() throws Exception {
        //1）创建一个client对象
        //2）创建一个QueryBuilder对象，查询条件。使用QueryBuilders工具类创建。
        QueryBuilder queryBuilder = QueryBuilders.idsQuery().addIds("1", "2");
        //3）使用client对象执行查询
        //设置查询的索引库
        //设置查询的索引库、type、查询条件
        SearchResponse response = client.prepareSearch("twitter")
                //设置查询的type
                .setTypes("user")
                //设置查询条件
                .setQuery(queryBuilder)
                //执行查询
                .get();
        //4）执行查询，并返回查询结果。QueryResponse对象。
        //5）从QueryResponse对象中取查询结果。
        SearchHits searchHits = response.getHits();
        printSearchHits(searchHits);
    }

    @Test
    public void findByTerm() {
        //洛阳、洛阳城可以搜到，只看洛阳城不能搜到
        QueryBuilder queryBuilder = QueryBuilders.termQuery("desc", "洛阳城");
        //执行查询
        SearchResponse response = client.prepareSearch("twitter")
                .setTypes("user")
                .setQuery(queryBuilder)
                .get();
        SearchHits searchHits = response.getHits();
        printSearchHits(searchHits);
    }

    @Test
    public void findByQueryString() {
        //先分词再查询
        QueryBuilder queryBuilder = QueryBuilders.queryStringQuery("只看洛阳城").defaultField("desc");
        SearchResponse response = client.prepareSearch("twitter")
                .setTypes("user")
                .setQuery(queryBuilder)
                //设置分分页信息
                .setFrom(0)
                .setSize(5)
                .get();
        SearchHits searchHits = response.getHits();
        printSearchHits(searchHits);
    }

    @Test
    public void findByHighlighting() {
        //查询条件
        QueryBuilder queryBuilder = QueryBuilders.termQuery("desc", "洛阳城");
        //查询之前先设置高亮信息
        HighlightBuilder highlightBuilder = new HighlightBuilder()
                //设置高亮显示的字段
                .field("desc")
                //设置高亮显示的前缀
                .preTags("<em>")
                //设置高亮显示的后缀
                .postTags("</em>");
        //执行查询
        SearchResponse response = client.prepareSearch("twitter")
                .setTypes("user")
                //查询条件
                .setQuery(queryBuilder)
                //高亮条件
                .highlighter(highlightBuilder)
                .get();
        //取查询结果
        SearchHits searchHits = response.getHits();
        //总记录数
        long totalHits = searchHits.getTotalHits();
        System.out.println("总记录数：" + totalHits);
        //取结果列表
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String json = hit.getSourceAsString();
            System.out.println(json);
            //取高亮的结果
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            //根据高亮字段取高亮结果
            HighlightField highlightField = highlightFields.get("desc");
            //高亮的结果，一般情况下只有一个值。
            Text[] fragments = highlightField.getFragments();
            Text fragment = fragments[0];
            //最终的高亮结果
            String hlContent = fragment.string();
            System.out.println(hlContent);
        }
    }

    public void printSearchHits(SearchHits searchHits){
        //取查询结果的总记录数。
        long totalHits = searchHits.getTotalHits();
        System.out.println("查询结果总记录数：" + totalHits);
        //取查询结果列表
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String json = hit.getSourceAsString();
            System.out.println(json);
        }
    }

    @After
    public void cleanUp() {
        if (client != null) {
            client.close();
        }
    }
}
```





```java

```

