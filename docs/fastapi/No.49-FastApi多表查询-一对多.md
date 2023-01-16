>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。


#### 前言

在 `FastApi` 使用过程中，多表查询是一个比较常见的需求。

继上一篇文章 `No.48-FastApi多表查询-一对一`,
本篇文章主要介绍`一对多`关系的写法，以及在使用过程中的数据的增、删、改、查。

#### `Model`的编写

主要分三步进行：
1. 表1的编写
2. 表2的编写
3. 表1和表2建立链接

这里进行举例，一个人可以有多张银行卡，人和银行卡的关系设定为 `一对多`

- `User` 表为用户表
- `Card` 表为银行卡表


##### 1. 表`User`的编写

```python
class User(Base):
    """
    用户表
    """
    __tablename__ = "user"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_name = Column(String(32), comment='用户名')
    age = Column(Integer, comment='年龄')
    sex = Column(String(8), comment='性别')
```

这里简单写了示例，3个字段：

- user_name: 用户名
- age: 年龄
- sex: 性别

##### 2. 表`Card`的编写

```python
class Card(Base):
    """
    银行卡基本信息表
    """
    __tablename__ = 'card'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    card_id = Column(String(30))
```

银行卡信息，简单给一个字段：

- card_id: 银行卡号

到目前为止，两张表都有各自的字段，没有建立任何联系。

下面为他们建立联系。

##### 3. 表`User` 和 `Card` 进行关联

```python
class User(Base):
    """
    用户表
    """
    __tablename__ = "user"

    id = Column(Integer, primary_key=True, autoincrement=True)
    user_name = Column(String(32), comment='用户名')
    age = Column(Integer, comment='年龄')
    sex = Column(String(8), comment='性别')
    cards = relationship('Card', backref='user')


class Card(Base):
    """
    银行卡基本信息表
    """
    __tablename__ = 'card'

    id = Column(Integer, primary_key=True, autoincrement=True)
    card_id = Column(String(30))

    user_id = Column(Integer, ForeignKey('user.id'))
```

增加了两行代码：

`User`表：
```python
cards = relationship('Card', backref='user')
```


`Card`表：
```python
user_id = Column(Integer, ForeignKey('user.id'))
```

这样 `Card` 表就通过外键 `user_id` 建立了连接

#### 数据查询

1. 当我们拿到 user 信息之后，怎么查询他名下的银行卡呢？

```python
db_user = db.query(User).filter(User.id == user.id).first()  # 获得用户信息

user_cards = db_user.cards  # 获得用户名下所有的银行卡信息
```

2. 当我们拿到 card 信息之后，怎么知道这个银行卡是属于哪个用户呢？

```python
db_card = db.query(Card).filter(Card.id == card.id).first()  # 获得银行卡信息

card_user = db_card.user  # 获得该银行卡的所属用户
```

#### 数据新增

1. 怎么给已知用户添加多张银行卡信息呢？

```python
db_user = db.query(User).filter(User.id == user.id).first()  # 获得用户信息

card1 = Card(card_id='1234567890')
card2 = Card(card_id='0987654321')


# 写法1：这种写法替换原来的所属关系，如果原来用户跟其他银行卡有关联关系将清除关系
db_user.cards = [card1, card2]  
# 写法2：这种写法在原来的关联关系基础上新增关联关系
db_user.cards.extend([card1, card2]) 


db.add(db_user)
db.commit()
db.refresh(db_user)
```

这里关联银行卡信息使用了两种方法，请根据实际情况酌情使用。

2. 怎么给已知的银行卡添加用户信息呢？

```pyton
db_card = db.query(Card).filter(Card.id == card.id).first()  # 获得银行卡信息

user = User(user_name='张三', age=20, sex='男')

db_card.user = user  # 更新银行卡所属用户信息

db.add(db_card)
db.commit()
db.refresh(db_card)
```

#### 数据修改

数据修改跟新增逻辑基本差不多，这里不在详细展开

#### 数据删除

因为数据之间产生了关联，所以删除的时候不能直接删除。

1. 删除 `User`

```python
db_user = db.query(User).filter(User.id == user.id).first()
db.delete(db_user)
db.commit()
```
这样就直接删除了`User`信息。
在`Card`表中，`user_id` 字段也会自动清空为：`null`

2. 删除 `Card`

```python
db_card = db.query(Card).filter(Card.id == card.id).first()
db.delete(db_card)
db.commit()
```
这样就直接删除了

---

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`