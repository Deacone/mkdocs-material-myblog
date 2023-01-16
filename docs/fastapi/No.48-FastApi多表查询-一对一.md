>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。


#### 前言

在 `FastApi` 使用过程中，多表查询是一个比较常见的需求，本篇文章主要介绍一对一关系的写法，以及在使用过程中的数据的增、删、改、查。

#### `Model`的编写

主要分三步进行：
1. 表1的编写
2. 表2的编写
3. 表1和表2建立链接

##### 1. 表`card`的编写

```python
class Card(Base):
    """银行卡基本信息"""
    __tablename__ = 'card'  # 数据库表名

    id = Column(Integer, primary_key=True, autoincrement=True)
    card_id = Column(String(30))
    card_user = Column(String(10))
    tel = Column(String(30))
```

这里对表`card`表声明了字段：
- id
- card_id
- card_user
- tel

##### 2. 表 `carddetail` 的编写

```python
class CardDetail(Base):
    """银行卡 详情信息"""
    __tablename__ = 'carddetail'  # 数据库表名

    id = Column(Integer, primary_key=True, autoincrement=True)
    mail = Column(String(30))
    city = Column(String(10))
    address = Column(String(30))
```

这里对表 `carddetail` 声明了字段：
- id
- mail
- city
- address

到目前为止，这两张表都是独立的，没有任何关联。
下面开始建立联系。

##### 3. 将两张表进行关联

###### 3.1 在第一张表里加入 `card_detail` 字段，使用关键字 `relationship` 跟表 `CardDetail` 建立对应关系。

```python
class Card(Base):
    """银行卡基本信息"""
    __tablename__ = 'card'  # 数据库表名

    id = Column(Integer, primary_key=True, autoincrement=True)
    card_id = Column(String(30))
    card_user = Column(String(10))
    tel = Column(String(30))
    card_detail = relationship("CardDetail", uselist=False, backref='card')
```

在使用 `relationship` 进行关联时候，`uselist` 很关键，从源码的注释中可以看出：

```
:param uselist:
    A boolean that indicates if this property should be loaded as a
    list or a scalar. In most cases, this value is determined
    automatically by :func:`_orm.relationship` at mapper configuration
    time, based on the type and direction
    of the relationship - one to many forms a list, many to one
    forms a scalar, many to many is a list. If a scalar is desired
    where normally a list would be present, such as a bi-directional
    one-to-one relationship, set :paramref:`_orm.relationship.uselist`
    to
    False.
    
    The :paramref:`_orm.relationship.uselist`
    flag is also available on an
    existing :func:`_orm.relationship`
    construct as a read-only attribute,
    which can be used to determine if this :func:`_orm.relationship`
    deals
    with collections or scalar attributes::
    
      >>> User.addresses.property.uselist
      True
    
    .. seealso::
    
      :ref:`relationships_one_to_one` - Introduction to the "one to
      one" relationship pattern, which is typically when the
      :paramref:`_orm.relationship.uselist` flag is needed.
```

由此可见，只有在 `one-to-one` 的时候才设置为 `False`。


这个时候进行数据结构同步，你会发现，数据库中并没有 `card_detail` 这个字段。没有关系，请看下一步。



###### 3.2 在第二张表 `CardDetail` 中加入 `card_id` 字段

```python
class CardDetail(Base):
    """银行卡 详情信息"""
    __tablename__ = 'carddetail'  # 数据库表名

    id = Column(Integer, primary_key=True, autoincrement=True)
    mail = Column(String(30))
    city = Column(String(10))
    address = Column(String(30))
    card_id = Column(Integer, ForeignKey('card.id'))
```

这个时候再进行数据结构同步，你会发现，表 `carddetail` 中多了一个字段 ：`card_id`

#### 数据查询

查询 `card` 表，再查询出 `carddetail` 的数据：

```python
db_card = db.query(Card).filter(Card.id == card.id).first()

card_detail = db_card.card_detail
```

由 `carddetail` 表，查询 `card` 表：

```python
db_card_detail = db.query(CardDetail).filter(CardDetail.id == card_detail.id).first()

card = db_card_detail.card
```

#### `carddetail`表增加一条数据和`card`表进行关联

```python
db_card = db.query(Card).filter(Card.id == card.id).first()

db_card_detail = db.query(CardDetail).filter(CardDetail.id == card_detail.id).first()

db_card.card_detail = db_card_detail
db.add(db_card)
db.commit()
db.refresh(db_card)
```

---

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`