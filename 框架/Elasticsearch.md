分布式的、Restful风格的搜索引擎

支持对各种类型的数据的检索

搜索速度快，可以提供实时的搜索服务

便于水平拓展，每秒处理PB级海量数据

​						

​						JSON

索引、类型、文档、字段

集群、节点、分片、副本

curl -X GET "localhost:9200/_cat/health?v"

curl -X GET "localhost:9200/_cat/nodes?v"

curl -X GET "localhost:9200/_cat/indices?v"

curl -X PUT "localhost:9200/test"

curl -X DELETE "localhost:9200/test"



localhost:9200/test/_search?q=title:互联网



repository返回了带有标签的高亮显示的内容，但是不是合起来的，是元数据+高亮显示数据，需要自己把两个合起来才行



在JAVA里面实现过程：

1. 在实体类里面写好注解

   ```java
   @Document(indexName = "discusspost", type = "_doc", shards = 6, replicas = 3)
   public class DiscussPost {
   
       @Id
       private int id;
   
       @Field(type= FieldType.Integer)
       private int userId;
   
       @Field(type= FieldType.Text,analyzer = "ik_max_word",searchAnalyzer = "ik_smart")
       private String title;
   
       @Field(type= FieldType.Text,analyzer = "ik_max_word",searchAnalyzer = "ik_smart")
       private String content;
   
       @Field(type= FieldType.Integer)
       private int type;
   
       @Field(type= FieldType.Integer)
       private int status;
   
       @Field(type= FieldType.Date)
       private Date createTime;
   
       @Field(type= FieldType.Integer)
       private int commentCount;
   
       @Field(type= FieldType.Double)
       private double score;
   ```

2. 实现一个接口extends ElasticsearchRepository<DiscussPost,Integer>

3. 调用这个接口进行操作



匹配有关的内容有关的一部分