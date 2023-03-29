# arangodb

## 1.基本教程

### 1.增删改查

1.在collection中插入一条document：

```
INSERT {
    "name": "Ned",
    "surname": "Stark",
    "alive": true,
    "age": 41,
    "traits": ["A","H","C","N","P"]
} INTO Characters
```

2. AQL 不允许在单个查询中以同一集合为目标的多个操作，但是可以循环

```
LET data = []
FOR d IN data
    INSERT d INTO Characters
```

3.在记录中，_key,_id,_rev,为文档的唯一标识，可以系统分配，也可以自己定义，可以用来检索，下划线开头的变量为只读的

```
{
    "_key": "2861650",
    "_id": "Characters/2861650",
    "_rev": "_V1bzsXa---",
    "name": "Ned",
    "surname": "Stark",
    "alive": true,
    "age": 41,
    "traits": ["A","H","C","N","P"]
  }
  
单个检索
RETURN DOCUMENT("Characters", "2861650")
// --- or ---
RETURN DOCUMENT("Characters/2861650")

多个检索
RETURN DOCUMENT("Characters", ["2861650", "2861653"])
// --- or ---
RETURN DOCUMENT(["Characters/2861650", "Characters/2861653"])
```

4.更新文档

```
UPDATE "2861650" WITH { alive: false } IN Characters
也可以向所有文档加入属性
FOR c IN Characters
    UPDATE c WITH { season: 1 } IN Characters

```

5.删除文档

```
REMOVE "2861650" IN Characters
FOR c IN Characters
    REMOVE c IN Characters
```

6.更新插入

```
UPSERT { name: 'superuser' } 
INSERT { name: 'superuser', logins: 1, dateCreated: DATE_NOW() } 
UPDATE { logins: OLD.logins + 1 } IN users
```



### 2.匹配文档

1.相等匹配

```
FOR c IN Characters
    FILTER c.name == "Ned"
    RETURN c
```

2.范围条件

```
FOR c IN Characters
    FILTER c.age < 13
    RETURN { name: c.name, age: c.age }
```

3.多个条件

```
FOR c IN Characters
    FILTER c.age < 13 AND c.age != null
    RETURN { name: c.name, age: c.age }
```

4.替代条件

```
FOR c IN Characters
    FILTER c.name == "Jon" OR c.name == "Joffrey"
    RETURN { name: c.name, surname: c.surname }
```

### 3.排序与限制

1.限制返回数

```
FOR c IN Characters
    LIMIT 2，5 #前一个数表示跳过2
    RETURN c.name
```

2.排序

```
FOR c IN Characters
    SORT c.name DESC #desc降序，允许多个值排序 SORT c.surname, c.name
    LIMIT 10
    RETURN c.name
```

### 4.关系查询

1.多表查询

```
FOR c IN Characters
    RETURN DOCUMENT("Traits", c.traits)  #c.traits 为一个列表，值在traits中为_key,返回完整的document文档
```

2.合并字符和特征，将数据添加到C集合之中，若C存在则覆盖

```
FOR c IN Characters
    RETURN MERGE(c, { traits: DOCUMENT("Traits", c.traits)[*].en } )
[*]表示解聚合，此处表示对集合中每一个元素使用.en
```

3.一种多表查询的方式

```
FOR c IN Characters
  RETURN MERGE(c, {
    traits: (
      FOR key IN c.traits
        FOR t IN Traits
          FILTER t._key == key
          RETURN t.en
    )
  })
```

### 5.图形遍历

父母和孩子之间的关系可以建模为图形。在ArangoDB中，两个文档（父文档和子字符文档）可以通过边缘文档链接。边缘文档存储在边缘集合中，并具有两个附加属性

```
#创建边缘集合

LET data = [
    {
        "parent": { "name": "Ned", "surname": "Stark" },
        "child": { "name": "Robb", "surname": "Stark" }
    }, {
        "parent": { "name": "Ned", "surname": "Stark" },
        "child": { "name": "Sansa", "surname": "Stark" }
    }, {
        "parent": { "name": "Ned", "surname": "Stark" },
        "child": { "name": "Arya", "surname": "Stark" }
    }, {
        "parent": { "name": "Ned", "surname": "Stark" },
        "child": { "name": "Bran", "surname": "Stark" }
    }, {
        "parent": { "name": "Catelyn", "surname": "Stark" },
        "child": { "name": "Robb", "surname": "Stark" }
    }, {
        "parent": { "name": "Catelyn", "surname": "Stark" },
        "child": { "name": "Sansa", "surname": "Stark" }
    }, {
        "parent": { "name": "Catelyn", "surname": "Stark" },
        "child": { "name": "Arya", "surname": "Stark" }
    }, {
        "parent": { "name": "Catelyn", "surname": "Stark" },
        "child": { "name": "Bran", "surname": "Stark" }
    }, {
        "parent": { "name": "Ned", "surname": "Stark" },
        "child": { "name": "Jon", "surname": "Snow" }
    }, {
        "parent": { "name": "Tywin", "surname": "Lannister" },
        "child": { "name": "Jaime", "surname": "Lannister" }
    }, {
        "parent": { "name": "Tywin", "surname": "Lannister" },
        "child": { "name": "Cersei", "surname": "Lannister" }
    }, {
        "parent": { "name": "Tywin", "surname": "Lannister" },
        "child": { "name": "Tyrion", "surname": "Lannister" }
    }, {
        "parent": { "name": "Cersei", "surname": "Lannister" },
        "child": { "name": "Joffrey", "surname": "Baratheon" }
    }, {
        "parent": { "name": "Jaime", "surname": "Lannister" },
        "child": { "name": "Joffrey", "surname": "Baratheon" }
    }
]

FOR rel in data
    LET parentId = FIRST(
        FOR c IN Characters
            FILTER c.name == rel.parent.name
            FILTER c.surname == rel.parent.surname
            LIMIT 1
            RETURN c._id
    )
    LET childId = FIRST(
        FOR c IN Characters
            FILTER c.name == rel.child.name
            FILTER c.surname == rel.child.surname
            LIMIT 1
            RETURN c._id
    )
    FILTER parentId != null AND childId != null
    INSERT { _from: childId, _to: parentId } INTO ChildOf
    RETURN NEW
    
#在具有用户自定义的键的情况下，可以
INSERT { _from: "Characters/robb", _to: "Characters/ned" } INTO ChildOf
```

遍历父母

```
FOR v IN 1..1 OUTBOUND "Characters/2901776" ChildOf
    RETURN v.name
OUTBOUND 查询节点从子节点到父节点,INBOUND从父节点，any表示双向
1..1 最小最大深度为1 ，便利更深层次或可移动层次则可以1..2
ChildOf  边缘图

```

遍历孙子孙女

```
FOR c IN Characters
    FILTER c.name == "Tywin"
    FOR v IN 2..2 INBOUND c ChildOf
        RETURN v.name
```

### 6.地理空间查询

查询与位置与位置之间的距离

```
FOR loc IN Locations
  LET distance = DISTANCE(loc.coordinate[0], loc.coordinate[1], 53.35, -6.25)
  SORT distance
  LIMIT 3
  RETURN {
    name: loc.name,
    latitude: loc.coordinate[0],
    longitude: loc.coordinate[1],
    distance
  }
```

查询半径范围内的位置

```
FOR loc IN Locations
  LET distance = DISTANCE(loc.coordinate[0], loc.coordinate[1], 53.35, -6.25)
  SORT distance
  FILTER distance < 200 * 1000
  RETURN {
    name: loc.name,
    latitude: loc.coordinate[0],
    longitude: loc.coordinate[1],
    distance: ROUND(distance / 1000)
  }
```

## 2.AQL语法

### 1.基础知识

1.@可以绑定参数，数组集合使用@@

```
FILTER u.name == @name   // correct
FOR doc IN @@collection   // correct

在查询语句中使用需要指定绑定值

{
  "query": "FOR u IN users FILTER u.id == @id && u.name == @name RETURN u",
  "bindVars": {
    "id": 123,
    "name": "John Smith"
  }
}
```

2.包含串联字符串，CONCAT

3.比较时首先使用数据类型，数据类型相同时，才使用实际值

```
null  <  bool  <  number  <  string  <  array/list  <  object/document

对于数组，集合等，则先按顺序比较对应位置
```

### 2.数据访问查询，修改查询

数据访问查询

```
RETURN "Hello ArangoDB!"

RETURN DOCUMENT("users/phil")

FOR doc IN users
    RETURN { user: doc, newAttribute: true }
    
FOR doc IN users
    FILTER doc.status == "active"
    SORT doc.name
    LIMIT 10
```

替换文档

```
REPLACE {
    _key: "NatachaDeclerck", //知道文档ID
    firstName: "Natacha",
    name: "Leclerc",
    status: "active",
    level: "premium"
} IN users
```

删除文档

```
REMOVE { _key: "GilbertoGil" } IN users
```

修改多个文档

```
FOR u IN users
    FILTER u.status == "not active"
    UPDATE u WITH { status: "inactive" } IN users
    
FOR u IN users
    INSERT u IN backup
    
FOR u IN users
    FILTER u.status == "deleted"
    REMOVE u IN backup
    
LET r1 = (FOR u IN users  REMOVE u IN users)
LET r2 = (FOR u IN backup REMOVE u IN backup)
RETURN true
```

归还文件

在修改新增删除操作以后可以使用NEW 或者 OLD 进行代指，使用类似NEW._key可以返回，也可以 { old: OLD, new: NEW }

```
UPSERT { name: "test" }
    INSERT { name: "test" }
    UPDATE { } IN users
LET opType = IS_NULL(OLD) ? "insert" : "update"
RETURN { _key: NEW._key, type: opType }
```

## 3.高级操作

### 1.FOR

用法

```
//一般语法
FOR variableName IN expression
//图中的特殊语法
FOR vertexVariableName [, edgeVariableName [, pathVariableName ] ] IN traversalExpression
//视图的特殊语法
FOR variableName IN viewName SEARCH searchExpression

//嵌套用法
FOR u IN users
  FOR l IN locations
    RETURN { "user" : u, "location" : l }
    
//

```

选项

```
5 个属性的阈值是任意的，可以使用提示进行调整

FOR doc IN collection OPTIONS { maxProjections: 7 } 
  RETURN [ doc.val1, doc.val2, doc.val3, doc.val4, doc.val5, doc.val6, doc.val7 ]
```

### 2.RETURN

```
FOR what IN 1..2
  RETURN DISTINCT (
    FOR i IN [ 1, 2, 3, 4, 1, 3 ] 
      RETURN i
  )
 
子层不会被去重，改函数返回  [ 1, 2, 3, 4, 1, 3 ] 
```

### 3.搜索

```
FOR doc IN viewName
  SEARCH expression
  OPTIONS { … }
```

### 4.赋值

```
LET a = [1, 2, 3]  // initial assignment
LET b = PUSH(a, 4) // allowed, result: [1, 2, 3, 4]
```

### 5.分组

```
FOR u IN users
  COLLECT city = u.city
  RETURN { 
    "city" : city 
  }
  
 //比如此处根据城市分组，有多少城市有多多少组
```

```
FOR u IN users
  COLLECT city = u.city INTO groups
  RETURN { 
    "city" : city, 
    "usersInCity" : groups 
  }
  //此处将分组结果与详细信息写入group，结果为字典列表
```

```
FOR u IN users
  COLLECT country = u.country, city = u.city INTO groups = u.name
  RETURN { 
    "country" : country, 
    "city" : city, 
    "userNames" : groups 
  }
  //指定=需要加入组的数据，不会全部加入
```

```
FOR u IN users
  COLLECT country = u.country, city = u.city INTO groups = { 
    "name" : u.name, 
    "isActive" : u.status == "active"
  }
  RETURN { 
    "country" : country, 
    "city" : city, 
    "usersInCity" : groups //此处也可以  groups[*].name 
  }
  //可以多个选择条件，[*]表示解聚合，对groups中每一个元素使用.name
```

```
FOR u IN users
  COLLECT age = u.age WITH COUNT INTO length
  RETURN { 
    "age" : age, 
    "count" : length 
  }
  //with count into 的固定语法可以计算长度
 
```

## 4.python-arango用法

### 1.使用客户端

```
from arango import ArangoClient

# Initialize the ArangoDB client.
client = ArangoClient(hosts='http://localhost:8529')

# Connect to "_system" database as root user.
# This returns an API wrapper for "_system" database.
sys_db = client.db('_system', username='root', password='passwd')

# Create a new database named "test" if it does not exist.
if not sys_db.has_database('test'):
    sys_db.create_database('test')

# Connect to "test" database as root user.
# This returns an API wrapper for "test" database.
db = client.db('test', username='root', password='passwd')

# Create a new collection named "students" if it does not exist.
# This returns an API wrapper for "students" collection.
if db.has_collection('students'):
    students = db.collection('students')
else:
    students = db.create_collection('students')

# Add a hash index to the collection.
students.add_hash_index(fields=['name'], unique=False)

# Truncate the collection.
students.truncate()

# Insert new documents into the collection.
students.insert({'name': 'jane', 'age': 19})
students.insert({'name': 'josh', 'age': 18})
students.insert({'name': 'jake', 'age': 21})

# Execute an AQL query. This returns a result cursor.
cursor = db.aql.execute('FOR doc IN students RETURN doc')

# Iterate through the cursor to retrieve the documents.
student_names = [document['name'] for document in cursor]
```

### 2.数据库

```
from arango import ArangoClient

# Initialize the ArangoDB client.
client = ArangoClient()

# Connect to "_system" database as root user.
# This returns an API wrapper for "_system" database.
sys_db = client.db('_system', username='root', password='passwd')

# List all databases.
sys_db.databases()

# Create a new database named "test" if it does not exist.
# Only root user has access to it at time of its creation.
if not sys_db.has_database('test'):
    sys_db.create_database('test')

# Delete the database.
sys_db.delete_database('test')

# Create a new database named "test" along with a new set of users.
# Only "jane", "john", "jake" and root user have access to it.
if not sys_db.has_database('test'):
    sys_db.create_database(
        name='test',
        users=[
            {'username': 'jane', 'password': 'foo', 'active': True},
            {'username': 'john', 'password': 'bar', 'active': True},
            {'username': 'jake', 'password': 'baz', 'active': True},
        ],
    )

# Connect to the new "test" database as user "jane".
db = client.db('test', username='jane', password='foo')

# Make sure that user "jane" has read and write permissions.
sys_db.update_permission(username='jane', permission='rw', database='test')

# Retrieve various database and server information.
db.name
db.username
db.version()
db.status()
db.details()
db.collections()
db.graphs()
db.engine()

# Delete the database. Note that the new users will remain.
sys_db.delete_database('test')
```

### 3.Collect

```
from arango import ArangoClient

# Initialize the ArangoDB client.
client = ArangoClient()

# Connect to "test" database as root user.
db = client.db('test', username='root', password='passwd')

# List all collections in the database.
db.collections()

# Create a new collection named "students" if it does not exist.
# This returns an API wrapper for "students" collection.
if db.has_collection('students'):
    students = db.collection('students')
else:
    students = db.create_collection('students')

# Retrieve collection properties.
students.name
students.db_name
students.properties()
students.revision()
students.statistics()
students.checksum()
students.count()

# Perform various operations.
students.load()
students.unload()
students.truncate()
students.configure()

# Delete the collection.
db.delete_collection('students')
```

### 4.文档

```
{
    '_id': 'students/bruce',
    '_key': 'bruce',
    '_rev': '_Wm3dzEi--_',
    'first_name': 'Bruce',
    'last_name': 'Wayne',
    'address': {
        'street' : '1007 Mountain Dr.',
        'city': 'Gotham',
        'state': 'NJ'
    },
    'is_rich': True,
    'friends': ['robin', 'gordon']
}
```

边缘文档，表示了文档与文档之间的边的关系

```
{
    '_id': 'students/bruce',
    '_key': 'bruce',
    '_rev': '_Wm3dzEi--_',
    'first_name': 'Bruce',
    'last_name': 'Wayne',
    'address': {
        'street' : '1007 Mountain Dr.',
        'city': 'Gotham',
        'state': 'NJ'
    },
    'is_rich': True,
    'friends': ['robin', 'gordon']
}
```

