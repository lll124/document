# SQLAlchemy模块使用

定义：

SQLAlchemy是Python编程语言下的一款ORM框架，该框架建立在数据库API之上，使用关系对象映射进行数据库操作，简言之便是：将对象转换成SQL，然后使用数据API执行SQL并获取执行结果。

说明：

SQLAchemy 本身无法操作数据库，其本质上是依赖pymysql.MySQLdb,mssql等第三方插件。

Dialect用于和数据库API进行交流，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作。

安装：

pip3 install SQLAlchemy

链接：

```python
MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>
  
pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]
  
MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>
  
cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]
```



```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# auth : pangguoping
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, UniqueConstraint, Index
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
# mysql+pymysql 链接方式://root:ycc962464 账户密码@127.0.0.1:3306链接地址及端口/tests链接的数据库名称?charset=utf8mb4 字符集
engine = create_engine("mysql+pymysql://root:ycc962464@127.0.0.1:3306/tests?charset=utf8mb4", max_overflow=5) 

Base = declarative_base()

# 创建单表
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True ) #主键 autoincrement=True 自增
    name = Column(String(32))
    extra = Column(String(16))
    num = Column(Integer)
    __table_args__ = (
    UniqueConstraint('id', 'name', name='uix_id_name'),
        Index('ix_id_name', 'name', 'extra'), #索引
#Index('my_index', my_table.c.data, mysql_length=10) length 索引长度
#Index('a_b_idx', my_table.c.a, my_table.c.b, mysql_length={'a': 4,'b': 9})
# Index('my_index', my_table.c.data, mysql_prefix='FULLTEXT') 指定索引前缀
# Index('my_index', my_table.c.data, mysql_using='hash') 指定索引类型
    )
class Home(Base):
    __tablename__ = 'home'
    id = Column(Integer,primary_key = True)
    tab = Column(String(20))
    values = Column(String(60))

# 一对多
class Favor(Base):
    __tablename__ = 'favor'
    nid = Column(Integer, primary_key=True)
    caption = Column(String(50), default='red', unique=True) # unique 唯一


class Person(Base):
    __tablename__ = 'person'
    nid = Column(Integer, primary_key=True)
    name = Column(String(32), index=True, nullable=True)
    favor_id = Column(Integer, ForeignKey("favor.nid"))

# 多对多
class ServerToGroup(Base):
    __tablename__ = 'servertogroup'
    nid = Column(Integer, primary_key=True, autoincrement=True) 
    server_id = Column(Integer, ForeignKey('server.id')) #外键
    group_id = Column(Integer, ForeignKey('group.id'))

class Group(Base):
    __tablename__ = 'group'
    id = Column(Integer, primary_key=True)
    name = Column(String(64), unique=True, nullable=False)


class Server(Base):
    __tablename__ = 'server'

    id = Column(Integer, primary_key=True, autoincrement=True)
    hostname = Column(String(64), unique=True, nullable=False)
    port = Column(Integer, default=22)

#定义初始化数据库函数
def init_db():
    Base.metadata.create_all(engine) 

#顶固删除数据库函数
def drop_db():
    Base.metadata.drop_all(engine)

# drop_db()
# init_db() 

Session = sessionmaker(bind=engine) #创建session
session = Session()

# def search(name): 
#     user = User.query.filter(User.name == name).first()
#     if user is None or user.name.strip == '':
#         print('用户不存在')
#     else:
#         print(' 用户 %s' % user.name)


# 增
# obj = User(name="111guanyu", extra='hanjiang') #添加单条
# session.add(obj) #加入到队列

# session.add_all([ #添加多条
#     User(name="liubei", extra='leader'),
#     User(name="zhangfei", extra='xiaodi'),
# ])
# session.commit()  # 添加到sql

#删
# session.query(User).filter(User.id > 2).delete()
# session.commit()

#改
# # 更新user表中id大于2的name列为099
# session.query(User).filter(User.id > 2).update({"name" : "099"})
# # 更新user表中id大于2的name列，在原字符串后边增加099
# session.query(User).filter(User.id > 2).update({User.name: User.name + "099"}, synchronize_session=False)
# # 更新user表中id大于2的num列，使最终值在原来数值基础上加1
# session.query(User).filter(User.id > 2).update({"num": User.num + 1}, synchronize_session="evaluate") # 数字相加，必须设置synchronize_session="evaluate"
# session.commit()


#查

# ret = session.query(User).all() # 查询所有
# sql = session.query(User) # 查询生成的sql
# print(sql)
# ret = session.query(User.name, User.extra).all() #查询User表的name和extra列的所有数据
# ret = session.query(User).filter_by(name='alex').all()  # 取全部name列为alex的数据
# ret = session.query(User).filter_by(name='alex').first() # 第一个匹配name列为alex的数据

# ret是一个对象列表。这个对象可以通过 “对象[索引].字段”来获取对应的值

# ret = session.query(User).all() #查询列表所有数据
# print(ret[0].name) #结果为元组，可通过下标的形式
# for i in ret: #循环元组，
#     print(i.name) 


#其他操作

#　条件
# ret = session.query(User).filter_by(name='alex').all() #查询所有name = alex 内容 
# ret = session.query(User).filter(User.id > 1, User.name == 'eric').all() # 且的关系

# ret = session.query(User).filter(User.id.between(1, 3), User.name == 'eric').all()
# print(ret[0].name)
# ret = session.query(User).filter(User.id.in_([1,3,4])).all() 
# print(ret)
# ret = session.query(User).filter(~User.id.in_([1,3,4])).all() # ~表示非。就是not in的意思
# print(ret)
# ret = session.query(User).filter(User.id.in_(session.query(User.id).filter_by(name='eric'))).all() # 联表查询
# print(ret)
# from sqlalchemy import and_, or_   # 且和or的关系
# ret = session.query(User).filter(and_(User.id > 3, User.name == 'eric')).all() # 条件以and方式排列

# # print(ret)
# ret = session.query(User).filter(or_(User.id < 2, User.name == 'eric')).all() # 条件以or方式排列
# for i in ret:
#     print(i.id)
# ret = session.query(User).filter(
#     or_( #这部分表示括号中的条件都以or的形式匹配
#         User.id < 2, # 或者 or User.id < 2
#         and_(User.name == 'eric', User.id > 3),# 表示括号中这部分进行and匹配
#         User.extra != ""
#     )).all()
 

# 通配符
ret = session.query(User).filter(User.name.like('e%')).all()
# print(list(ret))

# ret = session.query(User).filter(~User.name.like('e%')).all() # 表示not like
 
# # 限制 limit用法
ret = session.query(User)[0:3] # 等于limit ，具体功能需要自己测试 从几行到几行！
for i in ret:
    lists =[i.id,i.name,i.extra,i.num]
    print(lists)
# # 排序
# ret = session.query(User).order_by(User.name.desc()).all()
# ret = session.query(User).order_by(User.name.desc(), User.id.asc()).all() # 按照name从大到小排列，如果name相同，按照id从小到大排列
 
# # 分组
# from sqlalchemy.sql import func
 
# ret = session.query(User).group_by(User.extra).all()
# ret = session.query(
#     func.max(User.id),
#     func.sum(User.id),
#     func.min(User.id)).group_by(User.name).all()
 
# ret = session.query(
#     func.max(User.id),
#     func.sum(User.id),
#     func.min(User.id)).group_by(User.name).having(func.min(User.id) >2).all() # having对聚合的内容再次进行过滤
 
# 连表
 
# ret = session.query(User, Favor).filter(User.id == Favor.nid).all()
 
# ret = session.query(Person).join(Favor).all()
# # 默认是inner join
# ret = session.query(Person).join(Favor, isouter=True).all() # isouter表示是left join
 
# # 组合
# q1 = session.query(User.name).filter(User.id > 2)
# q2 = session.query(Favor.caption).filter(Favor.nid < 2)
# ret = q1.union(q2).all() #union默认会去重
 
# q1 = session.query(User.name).filter(User.id > 2)
# q2 = session.query(Favor.caption).filter(Favor.nid < 2)
# ret = q1.union_all(q2).all() # union_all不去重
```



 

# sqlalchemy 常用数据类型：

![img](../../static/img/1221916-20190307165714484-1026386596.png)

 

第一种：Integer

我们在Person模型中新增一个age字段

```python
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    age = Column(Integer)
```

 

```python
第二种：String
新增一个name字段（括号中的20表示该字符串最大长度为20）
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    name = Column(String(20))
```

 

第三种：Float

什么情况下会用到Float类型？比如存储体重、价格等.....

```python
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    price = Column(Float)
```

嗯！！我明明写的是123.456789，但是存储到数据库中却变成了123.457，为什么会这样呢？原因我之前说过：float单精度类型，单精度数据类型存储到表中容易被丢失。既然我们知道了原因，哪如何解决呢？？方法就是用接下来要讲的定点类型（DECIMAL）。

 

第四种：DECIMAL

DECIMAL可以防止数据精度

```python
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    price = Column(DECIMAL(7,3))
```

DECIMAL有两个参数,第一个参数用于指定一共多少位数，第二个参数用于指定小数点后最多多少位数

例如：DECIMAL(4,2)表示一共存储4位数字，小数点后最多有两位

如果传入不符合规则数值时会报如下错误：

![img](../../static/img/1221916-20190307170132951-1041544963.png)

 

 

第五种：Boolean

我们知道，1代表true，0代表false

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    delete = Column(Boolean)
```

 

第六种：Enum

什么情况下会用到枚举类型呢？比如用户填写性别时，固定只能选男或者女，不可能不男不女，对吧！

Enum()括号中为枚举列表，在这个里面可以罗列出可输入的值！

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    sex = Column(Enum("男","女"))
```

 

 

第七种：Date

Date只能存储指定的年月日，不能存储时分秒

说到日期类型，相信大家都熟悉，比如某年某月某日生。嗯、下面咱们就谈谈这个Date类型。

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    create_time = Column(Date)
```

然后从datetime导入datetime这个包，将数据添加至数据库

datetime()中的数值用于传递指定的年月日

```
from datetime import datetime
p = Person(create_time = datetime(2018,8,8))
session.add(p)
session.commit()
```

 

 

第八种：DateTime

DateTime存储指定的年月日时分秒

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    create_time = Column(DateTime)
```

添加测试数据：

```
p = Person(create_time = datetime(2018,8,8,16,11,50))
session.add(p)
session.commit()
```

 

 

第九种：Time

Time只能存储时分秒，不能存储年月日

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    create_time = Column(Time)
```

插入测试数据，time(）后面传递关键字参数，用于指定时分秒

```
from datetime import datetime,time
p = Person(create_time=time(hour=12,minute=20,second=50))
session.add(p)
session.commit()
```

 

第十种：Text

这个没什么好讲的啊，当字符串长度比较长时就可以使用Text类型

```
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)
    content = Column(Text)
```

 

 

第十一种：LongText

由于Text的存储长度有限，我们就可以使用LongText来存储数据。

由于LongText类型在mysql数据库才有，其它数据库没有该数据类型，在使用前，记得从mysql数据库导入该数据类型



```
from sqlalchemy.dialects.mysql import LONGTEXT
class Person(Base):
    __tablename__ = "person"
    id = Column(Integer , primary_key=True , autoincrement=True)

    content = Column(LONGTEXT)
```



 