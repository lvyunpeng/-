1、索引只能在$match和$sort操作，而且可以大大加速查询

2、在管道使用$match和$sort之外的操作符后不能使用索引

3、如果使用分片(分片存储大数据集合)，则$match和$project会在单独的片上执行。一但使用了其他操作符，其余的管道将会在主要片上执行。

4、allowDiskUse, 管道返回了超过MongoDB RAM内存限制的100MB数据，则会抛出exception

5、mongodb推荐使用聚合框架，基于$group等，map-reduce是一个很好的尝试

6、并发，原子性与隔离：mongodb2.2之前，单个读/写锁驻留在整个MongoDB实例；这意味着，在任何时候MongoDB只允许一个写和多个读操作；
   mongoDB2.2版本改成了数据库级别的锁，意味着在整个数据库级别而不是整个manggoDB实例级别使用锁，数据库可以有一个写者和多个读取者。
   在mongoDB3.0版本中，WiredTiger存储引擎工作在集合级别，提供了更加强大的文档级别的锁。

7、mongodb索引：

    1）唯一索引：db.users.createIndex({username: 1}, {unique: true, dropDups: true}); //mongodb3.0已删除dropDups参数
    2）稀疏索引：默认索引是密集的。稀疏索引的情况：一、我们想在集合里不是每个文档都出现的字段上建立唯一索引；二、集合中大量的文档不包含所有键值时[用户匿名评价，可能存在一半多的用户id不存在，为null] db.products.createIndex({user_id, 1}, {sparse: true, unique: false}); //sparse: true，表示该索引为稀疏索引
    3）多键索引：多个索引入口或者键值，引用同一个文档。如：
      {
      	name: "Wheelbarrow",
      	tags: ["tools", "gardening", "soil"]
      }
      在tags上建立缩影
    4）哈希索引：db.recipes.createIndex({recipe_name: 'hashed'})。 哈希查询限制：
       一、等值查询相似，不支持范围查询
       二、不支持多键哈希
       三、浮点数在哈希之前转换为整数，因此4.2和4.3有相同的哈希索引
       哈希索引优点：哈希索引上的入口是均匀分布的。对于分片集合非常有用，分片索引决定文档分配到哪个片中。如果分片索引基于增长的值，如MOngoDB OIDs（mongoDB默认使用的id），那么新创建的文档都只会插入到单个片中，除非索引是哈希的。
       除非显示指定，否则mongoDB都是使用OID作为主键，这就是一串连续的OID（基于创建时间生成的），当新的文档使用这些id插入时，他们索引的入口会彼此相近。如果使用这些id来决定文档保存在哪个片（机器）时，会产生大量的负载压力，因为只有一台机器处理这些请求。
       哈希索引通过均匀分布这些文档来解决这个问题，因此可以跨片或者跨机器存储
    5）地理空间索引（基于经纬度来存储文档）

 6、查看索引与构建：
 
    db.system.index.find().pretty()
    ns: 命名空间； key： 字段或者字段的组合； name：引用索引的名称
    构建索引时，对于大型数据集，可能需要几个小时甚至几天，在生产环境这可能是个噩梦，因为没有办法终止这个过程。需要数据库迁移一样对待。
    索引构建过程：1）对构建索引的值进行排序； 2）排序后的值插入到索引中
    db.currentOP()：查看索引的进度
    后台索引：在生产无法停止数据库访问的情况下，可以指定在后台构建索引。虽然会占用写锁，但是此过程中允许其他用户读/写数据库。db.values.createIndex({open: 1, close: 1}, background: true)
    离线索引：后台构建对服务器造成无法接受的压力时，采用离线索引。即离线构建出一个新的服务器节点，然后在此服务器上创建索引，并且允许此服务器复制主服务器数据。一旦更新完毕，把此服务器作为主服务器，然后采用第二台离线服务器构建其索引的版本。
    后台索引：在生产无法停止数据库访问的情况下，可以指定在后台构建索引。虽然会占用写锁，但是此过程中允许其他用户读/写数据库。db.values.createIndex({open: 1, close: 1}, background: true)
   
   
 7、索引查询计划：
 
    --slowms 50: 指定记录超过50s的操作，mongodb启动时指定
    hint（）强制优化器使用特定的索引，如：
    query = {stock_symbol: "GOGO", close: {$gt: 200}}
    db.values.find(query).hint({close: 1}).explain()
    查询计划缓存：当成功执行一个查询计划后，查询模式，nscanned的值和指定的索引就会被记录下来。记录的数据结构如下：
    {
   	pattern:{
   	  stock_symbol: 'equality',
   	  close: 'bound',
   	  index: {
   	    stock_symbol: 1
   	  },
   	  nscanned: 894
      }
    }
    查询模式记录了每个键的匹配类型。这里，我们查询对于stock_symbol的匹配记录（等于），而且在close范围（边界）匹配。当一个新的查询匹配这个模式时，就会使用相应的索引。
    优化器自动过期计划：
    1）集合写入了100次；
    2）集合新建或者删除了索引
    3）实用查询计划的查询做了比期望更多的工作。这里，衡量”更多的工作“用的是nscanned的值超过了缓存的nscanned的值至少是10
    
   
   



