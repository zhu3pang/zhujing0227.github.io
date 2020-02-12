---
layout: post
title: MongoDB 在 Java 中动态创建索引
categories: database
tags: [MongoDB]
comments: true
---

每次需要在 MongoDB 中创建索引都要找 DBA,一堆邮件审批,嗯,所以就试试直接在代码里创建索引吧

    public class MongoDBImpl<T> implements MongoDB<T>{
        @Autowired
        private MongoOperations mongoOperations;
        /**
         * 创建索引
         *
         * @param collectionName    表名
         * @param indexes           索引列名集合(可以一次创建多个)
         *                       单索引:{"createTime":1},复合索引:{"createTime":1,"clientId":-1}
         */
        @Override
        public void createIndex(String collectionName, String... indexes) {
            if (!mongoOperations.collectionExists(collectionName))
                mongoOperations.createCollection(collectionName);
    
            Arrays.stream(indexes)
                .filter(index -> !indexExists(collectionName, index))
                .forEach(index -> mongoOperations.getCollection(collectionName)
                    .createIndex(JSON.parseObject(index, BasicDBObject.class)));
    
        }
    
        /**
         * 创建索引
         *
         * @param entityClass   类,默认表名
         * @param indexes       索引列名集合
         *                    单索引:{"createTime":1},复合索引:{"createTime":1,"clientId":-1}
         */
        @Override
        public void createIndex(Class<?> entityClass, String... indexes) {
            createIndex(mongoOperations.getCollectionName(entityClass), indexes);
        }
    
    
        /**
         * 判断 MongoDB 中表"collectionName"是否存在索引"index"
         *
         * @param collectionName    表名
         * @param index             单索引:{"createTime":1},复合索引:{"createTime":1,"clientId":-1}
         * @return
         */
        private boolean indexExists(String collectionName, String index) {
            return mongoOperations.collectionExists(collectionName) &&
                mongoOperations.getCollection(collectionName)
                    .getIndexInfo()
                    .stream()
                    .anyMatch(dbObject -> dbObject.get("key").toString().replaceAll("\\s*", "")
                        .equals(index.replaceAll("\\s*", "")));
    
        }
    }

然后在任意一个 bean 注册时创建索引,大功告成

    @PostConstruct
    public void init() {
        //初始化索引
        String[] names = indexNames.split("&");//{'clientId':-1, 'createTime':-1}&{'clientId':-1}
        mongoDb.createIndex(ClientDTO.class, names);
        logger.info(Constant.loggerHeader_CRM + "初始化索引--->" + indexNames);
    }

