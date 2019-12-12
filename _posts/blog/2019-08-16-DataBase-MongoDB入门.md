---
layout: post
title: MongoDb入门
categories: database
description: MongoDB
keywords: MongoDB
---

MongoDB是一种基于文档存储的NoSQL。

### MongoDB的安装
https://blog.51cto.com/13641879/2141129

### MongoDB Shell命令
https://www.cnblogs.com/clsn/p/8214194.html#auto_id_25

#### MongoDB连接命令

```
mongo ip:port/dbName -u user -p password

bin/mongo 172.17.17.5:27017/admin -u xxx -p xxx

bin/mongo 172.17.17.5:27017/nubia_passport -u xxx -p xxx --authenticationDatabase 'admin'
```
#### MongoDB Shell命令

根据ObjectId查找
```
db.oauth_token.findOne({"_id":"KoiBru5VDVNy8sU0bDadNTXidD3KG3ea8sAL4AMZEpnV1YCoXRLfqmIlrdSvzyfN4PEni3wn1q4ELEYDmJj0d-fEdCz1jNgkXpl981T_x21VFxl850ZA6XF_3DZi_YmG"});
```

```
// 创建数据库  不存在则自动创建，没有数据则退出后删除
use test;
// 查看所有数据库
show dbs;
// 删除数据库
db.dropDatabase()
// 集合 类似关系型数据库中的表
// 查看已有集合
show collections
// 或者
show tables;
// 创建一个固定集合，大小为1024Kb，最大数量为1000条
db.createCollection("myTbl",{capped : true, autoIndexId : true, size : 1024, max:1000 })
// 删除集合
db.myTbl.drop();
// 直接往不存在的集合插入文档，则自动创建集合
db.a.insert({name:'zdq',age:'18',sex:'1'})
// 往集合中插入文档
db.a.insert({name:'mongo',age:'20',sex:'1'})
// 查询数据
db.a.find();
// 查找年龄大于19的数据
db.a.find({name:{$gt:"19"}});
// 查找年龄大于19并且性别为0的人
db.a.find({name:{$gt:"19"},sex:"0"});
// 查找年龄大于19或者性别为1的人
db.a.find({name:{$gt:"19"},$or:[{sex:"0"}]});
// 易读的方式查询文档
db.a.find().pretty();
// 更改文档，固定集合不能更改
db.a.update({name:'zdq'},{$set:{name:'mongo2'}})
// 删除文档
db.a.remove({name:"mongo2"})

```

#### MongoDB 常用性能监控命令

```
Mongostat
Mongotop
db.serverStatus()
db.stats()
db.collection.stats()
rs.status()
sh.status()
```

### Java操作MongoDB
Maven相关依赖
```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver</artifactId>
    <version>3.11.0</version>
</dependency>
```
Java简单操作MongoDB
```
public class MongoUtil {

    private static final Logger LOGGER = LoggerFactory.getLogger(MongoUtil.class);

    // 获取客户端连接
    public static MongoClient getMongoClient (){
        ServerAddress serverAddress = new ServerAddress("192.168.128.131", 27017);
        List<ServerAddress> list = new ArrayList<ServerAddress>();
        list.add(serverAddress);
        MongoCredential credential = MongoCredential.createScramSha1Credential("zdq", "zdq", "123456".toCharArray());
        List<MongoCredential> credentials = new ArrayList<MongoCredential>();
        credentials.add(credential);
        return new MongoClient(list, credentials);
    }
    // 获取所有的tables
    public static void showCollection (){
        MongoClient mongoClient = getMongoClient();
        MongoDatabase db = mongoClient.getDatabase("zdq");
        //db.createCollection("passport");
        MongoIterable<String> tables = db.listCollectionNames();
        MongoCursor<String> cursor = tables.iterator();
        while (cursor.hasNext()){
            LOGGER.info(cursor.next());
        }
    }
    // 获取passport下面的所有数据
    public static void showDataList (){
        MongoClient mongoClient = getMongoClient();
        MongoDatabase db = mongoClient.getDatabase("zdq");
        MongoCollection<Document> collection = db.getCollection("passport");
        FindIterable<Document> documents = collection.find();
        MongoCursor<Document> mongoCursor = documents.iterator();
        while (mongoCursor.hasNext()) {
            Document doc = mongoCursor.next();
            LOGGER.info(doc.toJson());
        }
    }
    // 往passport添加数据
    public static void saveData (){
        MongoClient mongoClient = getMongoClient();
        MongoDatabase db = mongoClient.getDatabase("zdq");
        MongoCollection<Document> collection = db.getCollection("passport");
        Map<String, Object> map = new HashMap<>();
        map.put("name", "lucy");
        map.put("age", "23");
        Document document = new Document(map);
        collection.insertOne(document);
    }
    // 查找数据
    public static void findData(){
        MongoClient mongoClient = getMongoClient();
        MongoDatabase db = mongoClient.getDatabase("zdq");
        MongoCollection<Document> collection = db.getCollection("passport");
        FindIterable<Document> documents = collection.find(Filters.gt("age", "23"));
        MongoCursor<Document> mongoCursor = documents.iterator();
        while (mongoCursor.hasNext()) {
            Document doc = mongoCursor.next();
            LOGGER.info(doc.toJson());
        }
    }
    // 修改数据
    public static void updateDate(){
        MongoClient mongoClient = getMongoClient();
        MongoDatabase db = mongoClient.getDatabase("zdq");
        MongoCollection<Document> collection = db.getCollection("passport");
        Map<String, Object> map = new HashMap<>();
        map.put("name", "lancy");
        map.put("age", 17);
        map.put("sex", "1");
        UpdateResult updateResult = collection.updateMany(new Document("name", "jack"), new Document("$set", new Document(map)));
        LOGGER.info("matchedCount:" + updateResult.getMatchedCount() + " modifiedCount:" + updateResult.getModifiedCount());
    }

    public static void main (String[] args){
        showDataList();
    }
}
```
### SpringBoot整合MongoDB 使用MongoTemplate

Maven相关依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```
yml配置文件
```
spring:
  data:
    mongodb:
      uri: mongodb://zdq:123456@192.168.128.131:27017/zdq  #这里使用了权限认证
```
定义一个UserInfo对象存储到MongoDB
```
@Data
@ToString
@Document(collection = "userInfo")  //指定默认的保存的集合
public class UserInfo implements Serializable {

private static final long serialVersionUID = -4709412514538567218L;

@Id // MongoDB存储的文档必须有一个唯一的"_id"值，没有指定则会自动生成
private String id;
private String username;
private Integer age;
private Integer sex;
}

```
插入数据可用insert或者save。save语句如果原数据_id已经存在那么就相当于update操作，否则为insert操作。
```
        UserInfo userInfo = new UserInfo();
        userInfo.setUsername("sunhonglei");
        userInfo.setAge(50);
        userInfo.setSex(1);
        mongoTemplate.save(userInfo);
//        mongoTemplate.insert(userInfo);
```
指定条件查询
```
        // 查找性别为男并且年龄大于20的用户信息
        Query query = new Query();
        query.addCriteria(Criteria.where("sex").is(1).and("age").gt(20));
        List<UserInfo> list = mongoTemplate.find(query, UserInfo.class);
        for (UserInfo u : list){
            logger.info(u.toString());
        }
```
修改语句
```
        Query query = new Query();
        query.addCriteria(Criteria.where("_id").is("zhangxueyou11111"));
        Update update = Update.update("age", 55);
        UpdateResult updateResult = mongoTemplate.updateFirst(query, update, UserInfo.class);
        logger.info("matchedCount: " + updateResult.getMatchedCount() + " modifiedCount: " + updateResult.getModifiedCount());
```
删除语句，删除id为zhangxueyou11111的文档
```
        Query query = new Query();
        query.addCriteria(Criteria.where("_id").is("zhangxueyou11111"));
        DeleteResult deleteResult = mongoTemplate.remove(query, UserInfo.class);
        logger.info("deletedCount: " + deleteResult.getDeletedCount());
```