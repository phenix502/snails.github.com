---
layout:       post
title:        "MongoDB Basic"
description: 
banner: 
categories: 
- DB,MongoDB
---

#MongoDB特点

* 数据类型丰富。
  MongoDB是面向文档的数据库，不是关系型数据库。它将原来关系型数据库的“行”概念变换为更为灵活的“文档”模型。面向文档的方式可以将文档或数组内嵌进来，用一条记录表示可能需要关系数据库多个表关联的数据。

* 扩展容易
  MongoDB采用面向文档的数据模型可以很方便的在多台服务器之间分割数据，环科院实现集群的数据和负载，自动重排文档。

* 功能丰富
  MongoDB支持通用辅助索引，能进行多种快速查询，也提供唯一、复合和地理空间索引能力。

  可以直接在服务器端存取Javascript的函数和值。

  支持MapReduce等聚合工具。

  集合的大小有上限，对某些数据特别有用。

  MongoDB支持易用的协议存储大型文件和文件的元数据。

* 高速
  MongoDB使用自己的传输协议作为与服务器交互的方式，对文档进行动态填充，预分配数据文件，用空间换取性能的文档。

* 易管理
  MongoDB尽量让服务器自己来管理自己，而且还可以自己在主服务器down掉的时候，自动切换到备用服务器。它的管理理念就是尽量让服务器自动配置，让用户能在需要的时候调整设置。

#MongoDB基础

MongoDB的基本单元是：文档。每个文档都有一个特殊的键:"_id。MongoDB的单个实例可以容纳多个独立的数据库，每个数据库都有自己的集合和权限。

文档是MongoDB的基础，多个键及其相关的值有序地放在一起就构成了文档。

MongoDB文档的键是字符串，但是这些键不能使用`\0`（表示键的结束),`.`和`$`一般 不可以使用，以下划线"_"开头的键是保留的，一般不建议使用。

MongoDB的键不可以有重复的。

文档是构成MongoDB的基本单元，多个文档组合到一起就是`集合`。集合的名字不能包含`\0`，`system.`以及`$`。

多个集合构成了一个数据库，数据库的名字不能使用空字符串，不能含`.`,`$`,`/`,`\`和`\0`，不能超过64字节，全部小写。

##插入

    post = { "title": "My First Blog", "content": "test,test","date": new Date() }
    db.blog.insert(post)

##查询
   
    db.blog.find()  #默认返回20条记录

##更新

    update至少需要2个参数，第一个是更新文档的限定条件，第二个是新的文档。
    post.comments = ["yeah, It is cool."]
    db.blog.update({title: "My First Blog"}, post)

##删除

    remove默认删除集合内的所有文档，当然也可以设置限定条件。
    db.blog.remove({title: "My First Blog"})

-----

##数据类型

MongoDB的文档类似JSON，在概念上和JavaScript相似。JSON包含null, 布尔，整数、字符串、数组和对象几种类型。MongoDB扩展了JSON类型。MongoDB还包括64位浮点数，符号、对象ID，日期，正则表达式、代码，二进制数据，最大值，最小值和未定义的类型，内嵌文档等等。

###数字

MongoDB的数字都是以64位浮点数的形式保存的。内嵌文档表示shell显示的是一个用6位浮点数近似表示的64位整数。如果插入的64位整数不能精确地表示为双精度数字显示，shell会添加两个键top和bottom，分别表示高32位和低32位。

###Date

MongoDB的日期通常使用new Date()来记录，如果使用Date()函数就会导致日期和字符串不匹配，对更新表和查询带来问题。它记录的是从标准纪元开始的毫秒数，但是不包含任何时区相关的数据，如果要保存，那么就需要额外的字段来保存了。

###数组

数组可以包含不同数据类型的元素。MongoDB可以“理解”文档中的数组的结构，并可以深入数组内部对数组进行操作。

###内嵌文档 

内嵌文档是把MongoDB文档作为另一个文档键的一个值。这样数据组织的更加自然，而且不用保存成扁平结构的。

###对象ID

MongoDB中存储的每个文档都有一个"_id"键，这个键的值可以是任何类型的，默认为ObjectId对象。在每个集合里面，每个文档都有唯一的"_id"值，来确保集合里面的每个文档都可以被唯一标识。

ObjectId在不同机器上都能用全局唯一的同种方法方便地自动生成它。所以MongoDB采用它，而不是使用自动增长的主键。ObjectId使用12字节的存储空间，每个字节两位十六进制，所以一个ObjectId是一个有24位字符的字符串。

>ObjectId的前4位表示从标准纪元开始的时间戳，3位表示机器名的散列值，2位表示PID，3位表示计数器。

ObjectId分别从时间上，同一机器的同一进程的不同时间上产生的ObjectId是唯一的。

##插入数据

MongoDB可以批量插入数据，这样可以大大提高插入的速度。原因在于批量的插入相当于有一个TCP头，服务器可以减少解析众多其它TCP头的时间，从而提高了速度。但是批量也不是对TCP数据的大小无所要求，它要求它不能超过__16MB__.

在MongoDB插入数据的时候，数据首先会被转换成BSON格式的数据，然后检查数据是否包含"_id"键和文档大小是否超过4MB，然后将数据原样保存到数据库。不执行插入数据中的任何命令，可以有效保证数据库不会被注入攻击。但是这样做的坏处就是数据库可能会插入很多无效数据。

如果需要对数据进行检查，在启动数据库的时候可以添加`--objcheck`选项。

##删除数据

MongoDB删除数据是永久性的，无法进行回滚撤销，所以不太适合事务类的业务。或许这也是它删除文档会很快的原因之一——不用去关注数据库日志保证数据库的完整性。

##更新数据

MongoDB使用update对数据进行更新，它接收两个参数，一个是更新条件，另一个是描述对文档做了哪些修改。

`$set`修改器可以更新某个原子的值：

    db.blog.update({"title": "test"}, {"$set": {"what": "what the hell?"}})

`$unset`可以取消某个原子文档。

    db.blog.update({"title": "test", {"$unset": {"what": 1}}})

`$inc`修改器可以增加已有的键值，或者在不存在的时候创建一个键。

    db.blog.update({"title": "test", {$inc: {"_id": 1}}})

`update`想已有的数组末尾添加一个元素，如果没有该数组，就会创建一个新的数组。
  
    db.blog.update({"title": "hello"}, {$push : {"comments" : {"name":"test","content":"just for test"}}})

如果一个值不再某个数组里面则加入，如果存在则跳过：

    db.blog.update({"comments": {"$ne": "test"},{$push: {"comments": "test it"}})

`$addToSet`也可以达到同样的效果。

    db.blog.update({"_id":ObjectId("34234")}, {$addToSet: {"emails": "test@example.com"}})

`$addToSet`和`$each`搭配可以插入多个不同的值：

    db.blog.update({"_id":ObjectId("1231")},{$addToSet:{"emails:":{"$each": ["a","b","c"]}}})

`$pop`可以从数组的任一端删除元素。{$pop : { key : 1 }}从数组尾部删除，{$pop : { key : -1 }}从头部删除。
如果需要基于特定条件删除元素，而不是根据位置，可以使用`$pull`命令。

    db.blog.update({},{"$pull" : { "title" : "hello world"}})

使用定位符`$`可以删除__第一个匹配__的元素：

    db.blog.update({"title":"test"}, {"$set" : {"comments.$.author" : "David"}})

如果知道某个记录的下标还可以使用下标进行指定操作：

    db.blog.update({"title":"test"}, {$inc : { "comments.0.votes" : 1 }})

在文档大小不变的情况下，修改器速度会很快。`$inc`由于不会修改文档的大小，所以速度非常快。而`$set`可能会修改文档的大小，所以速度会比较低。

`upsert`是一种特殊的更新。如果没有文档符合更新条件，就会以这个条件和更新文档为基础创建一个新的文档。如果找到了匹配的文档，则正常更新。upsert属于upadte的第三个参数，如果需要使用，只需要将update的第三个参数设置为`true`即可。

    db.blog.update({"title":"my blog"}, {$inc : { "page.counts" : 1 }}, true)

`save`函数是一个shell函数，可以在文档不存在时插入，在存在时进行更新。save函数会调用upsert。

    post = db.blog.findOne();
    post.counts = 0;
    db.blog.save(post) # db.blog.update({"_id" : post._id}, x)

>如果需要更新多个文档，则需要将update的第四个参数设置为true.

获取更新记录的条数可以使用`getLastError`命令，键n的值就是所要的数字。

    db.runCommand({getLastError : 1})

获取已更新的文档可以使用`findAndModify`命令。

    db.runCommand({"findAndModify" : "processes"}, "sort" : { "priority" : -1 }, "update": {"$set" : { "status" : "READY" }})

#查询

##find

find的第一个参数决定了返回哪些文档，也就是它是作为查询的条件的。如果为空，则查询全部数据。

    db.blog.find({ "title": "hello", comments: 0 })

`$lt`,`$lte`, `$gt`,`$gte`,`ne`分别表示 `>`, `>=`, `<`, `<=`,`!=`,使用方法是：
    db.blog.find({"age" : { "$lt" : 30}})

`$or`的用法:
    
    db.blog.find({"$or" : [{condition1: c1, condition2 : c2}]})

`$in`的用法：

    db.blog.find({"cond1" : {"$in" : [c1, c2]})

取反使用`$nin`。

`$not`的用法：

    db.blog.find({"age" : { "$not" : { "$mod" : [10,1] } }})

上面的句子返回年龄mod 10不为1的文档。

第二个参数用于设定返回的键，如果希望某个键返回，在`:`后数字应该为1，否则为0，这样就不会返回这个字段了。在没有特殊设定下，`_id`总是默认返回的。

`$all`用于查询同时满足若干条件的数组，注意它只适用于查询数组。

    db.food.find({"fruit" : { $all : ['apple','banana'] }})

查询数组还可以指定匹配元素的下标。比如上面的fruit可以添加fruit.2，当然这样一来，后面的$all就不能使用了。

`$slice`既可以指定查询结果的条数，也可以制定查询结果的偏移。

    db.blog.posts.find({},{"comments" : { $slice : [20,10] }}) #21-30

    db.blog.posts.find({},{"comments" : { $slice : 10 }}) #前十条

##查询内嵌文档

    db.blog.find({"post.title" : "test", "post.username" : "author1"})

与之相比`db.blog.find({"post" : {"title" : "test", "username" : "author1"}})`要严格根据条件的顺序进行匹配的，如果文档中添加了其它的字段，就会导致数据无法正常查找到。

点表示法表示"深入潜入文档内部"，这也导致MongoDB不允许插入的文档包含`.`的原因，如果保存的文档中包含`.`，那么需要在保存前对其进行特殊处理。

如果匹配的内容是一个数组，那么上面的查询方法是无法起作用的，上面的会匹配数组中的所有元素，这也导致结果是不正确的。为了避免这个问题，需要使用`$eleMatch`.

    db.blog.find({"post" : { "$eleMatch" : { "title" : post, "username" : "author1" } }})

如果需要比较某个文档内部满足条件的文档，则前面的查询条件是无法得到满足的，所以就有了`$where`子句。它后面可以跟一个Javascript函数，函数如果返回true，则返回该文档。不过使用它的坏处就是__性能__，因为BSON数据与要转换为JavaScript对象，然后通过$where字句的表达式运行，而且还不能使用索引，所以效率很低。在使用的时候可以私用非$where子句对条件先进行筛选，使用$where对结果进行调优。

##游标

如果将执行结果赋值给某个变量，那么这个变量就是一个游标了。游标用于对结果进行遍历，`cursor.next`取下一个文档，`cursor.hasNext()`用于查询结果集是否遍历完成。另外，游标实现了`forEach`迭代器方法：

    var cursor = db.blog.find()
    cursor.forEach(function(x) {
     print(x.name);
        });

在调用find的时候，shell并不立即查询数据库，而是等待真正要获取数据的时候才发送查询；这样也可以对查询附加额外的玄仙。

    var cursor = db.blog.find().sort({"x" : 1}).limit(2).skip(3)

在执行`hasNext`时，shell会立即去获取前100条或4MB大小的数据（取小）。

    var cursor = db.blog.find().limit(50).sort({"price" : -1})

skip在略过的记录数量比较大时，通常就会出现性能问题。如果不能避免使用skip，那么可以利用上次查询的结果来计算获取的下一次数据。

##索引

创建索引
  
    db.blog.ensureIndex({"title" : 1, "date" : -1}, {"name" : "myindex", background : true})

删除索引

    db.runCommand({"dropIndexes" : "db_name", "index" : "index_name"})
    db.runCommand({"dropIndexes" : "db_name", "index" : "*"}) #delete all the indexes of db_name

地理空间索引

    db.map.ensureIndex({"gps" : "2d"}) #这里需要使用2d

查找附近的点：

    db.map.find({"gps" : { "$near" : [40,30] }}).limit(10)
    db.runCommand({geNear : "map", near : [40,30], num : 10})

后者还会返回每个文档到查询点的距离，单位为插入数据的单位。

##聚合

获取查询的结果的数据个数：

    db.blog.count()
    db.blog.count({"title" : ”test})

获取不同的值

    db.runCommand({"distinct" : "blog", "key" : "title"}) #键是必需的

分组获取数据。以获取大于某日的股票收市价格为例:

    db.runCommand({"group" : {
      "ns" : "price", #文档名称
      "key" : "init_date", #key
      "initial" : {"time" : 0} #initial value
      "$reduce" : function(doc, prev){
                    if (doc.time > prev.time){
                      prev.price = doc.price;
                      prev.time = doc.time;
                    }
                  }
    }}, {"condition" : {"day" : {"$gt" : "20120801"}}})


    #eg.2
    db.runCommand({"group" :{
     "ns" : "price",
     "key" : {"tags" : true}],
     "initial" : {"tags" : {}},
     "$reduce" : function(doc,prev){
                   for (i in doc.tags){
                     if(doc.tags[i] in prev.tags){
                       prev.tags[doc.tags[i]] ++;
                       }else{
                        prev.tags[doc.tags[i]] = 1;
                       }
                    }
                 },
    "finalize" : function(prev) {
      var mostPopular = 0;
      for (i in prev.tags){
        if (prev.tags[i] > mostPopular){
          prev.tag = i;
          mostPopular = prev.tags[i];
        }
      }
      delete prev.tags;
    }
    }})

MapReduce的操作过程：映射，将操作映射到集合中的每个文档；洗牌，按照键分组，并将产生的键值组成列表放在对应的键中；化简，将列表中的值化简成一个单值。继续上面的过程，直到每个键的列表只有一个值结束。

    #get all the keys in a document
    map = function(){
      for(var key in this){
        emit(key, {count : 1});
      }
    };

    reduce = function(key, emits){
      total = 0;
      for (var i in emits){
       total += emits[i].count;
      }
      return {"count" : total}
    }

    mr = db.runCommand({"mapreduce": "blog", "map" : map, "reduce" : reduce},out : {replace:"mr", db: "test"})
    {
	   "result" : {
	  	"db" : "test",
	  	"collection" : "mr"
    	},
    	"timeMillis" : 353, #执行时间
    	"counts" : {  
		  "input" : 1,  #集合个数
	   	"emit" : 2,  #emit被调用的次数
	  	"reduce" : 0, #reduce的次数
	  	"output" : 2  #结果集合中创建文档的数量
    	},
     	"ok" : 1
    }
    
    db.mr.find() #显示详细信息

##MongoDB命令

MongoDB的命令其实是作为特殊的查询来实现的，这些查询的集合为`$cmd`，runCommand仅仅是接收命令文档，执行等价查询，如db.runCommand({"drop" : "test"})等价于`db.$cmd.find({"drop" : "test"})`。

创建固定集合:
    db.createCollection("my_collection", {capped: true, size:10000, max:100}) #固定集合，大小为1000字节，最大可以容纳100个文档。

固定集合的文档按照顺序存储的，而且只有固定集合是顺序存储的，在对查询结果进行排序的时候，可以制定排序的方式。

    db.my_collection.find().sort({"natural": -1}) #逆序

`尾部有标`和`tail -f`命令类似，它不会获取不到数据后自动销毁游标，一旦有新的文档添加到集合中，新添加的文档就会被取出并输出。

##GridFS存储文件

GridFS是MongoDB存储大二进制文件的机制，使用它可以:
* 简化需求，GridFS不需要使用独立文件存储架构。
* 直接利用业已建立的复制或分片机制，对文件存储和故障恢复及扩展都很容易。
* 避免用户用于存储的文件系统出现问题。
* 可以不产生磁盘碎片，因为MongoDB分配数据空间以2GB为一块。

    ─ ➤  mongofiles put logfile 
    connected to: 127.0.0.1
    added file: { _id: ObjectId('501e5b94d6079bdd94ce708d'), filename: "logfile", chunkSize: 262144, uploadDate: new Date(1344166804850), md5: "70377c17d05fe2aa19bf8209137dfdc4", length: 27 }
    done!
    ─ ➤  mongofiles list       
    connected to: 127.0.0.1
    logfile	27
    ─ ➤  rm logfile
    ─ ➤  mongofiles get logfile
    connected to: 127.0.0.1
    done write to: logfile
    ─ ➤  mongofiles search logfile
    connected to: 127.0.0.1
    logfile	27
    ─ ➤  mongofiles delete logfile
    connected to: 127.0.0.1
    done!

GridFS是一个建立在普通MongoDB文档基础上的轻量级文件存储规范。GridFS的基本思想是将大文件分成很多块，每块作为单独的文档存储，这样就能存储大文件。MongoDB支持在文档中存储二进制数据，所以可以最大限度地减小块的存储开销。除了存储文件本身的块，还有一个单独的文档用来存储分块信息和文件元数据。


