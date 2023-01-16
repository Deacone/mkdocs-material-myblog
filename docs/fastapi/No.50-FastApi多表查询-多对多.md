>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。


#### 前言

在 `FastApi` 使用过程中，多表查询是一个比较常见的需求；

继上一篇文章 `No.49-FastApi多表查询-一对多`，

本篇文章主要介绍多对多关系的写法，以及在使用过程中的数据的增、删、改、查。

#### `Model`的编写

这里使用用户和性格作为 `多对多` 关系映射：

一个用户可以有多重性格；
一种性格可以对应多个用户；

主要分三步进行：
1. `User` 用户表编写
2. `Nature` 性格表编写
3. `User` 和 `Nature` 建立链接

##### 1. `User` 表的编写

```python
class User(Base):
    """
    用户表
    """
    __tablename__ = "user"

    user_name = Column(String(32), comment='用户名')
    age = Column(Integer, comment='年龄')
    sex = Column(String(8), comment='性别')

    cards = relationship('Card', backref='user')

```
这张表三个字段描述了用户信息：

- user_name: 用户名
- age: 年龄
- sex: 性别

##### 2. `Nature` 表的编写

```python
class Nature(Base):
    """
    性格表
    """
    __tablename__ = 'nature'

    name = Column(String(64))
```

这张表只有一个字段：
- name: 性格名称(开朗、内向等)

##### 3. `User` 和 `Nature` 建立链接

整体改动代码如下：
```python
class UserNature(Base):
    """
    用户、性格中间表
    """
    __tablename__ = "user_nature"
    __table_args__ = {'comment': '用户-性格中间表'}

    user_id = Column(Integer, ForeignKey('user.id'), primary_key=True, nullable=False, index=True)
    nature_id = Column(Integer, ForeignKey('nature.id'), primary_key=True, nullable=False, index=True)


class User(Base):
    """
    用户表
    """
    __tablename__ = "user"

    user_name = Column(String(32), comment='用户名')
    age = Column(Integer, comment='年龄')
    sex = Column(String(8), comment='性别')

    cards = relationship('Card', backref='user')
    natures = relationship('Nature', secondary='user_nature', back_populates='users')
    

class Nature(Base):
    """
    性格表
    """
    __tablename__ = 'nature'

    name = Column(String(64))

    users = relationship('User', secondary='user_nature', back_populates='natures')
```
###### 改动代码解释：
先创建一个中间表，有两个字段：

- `user_id`: 对应 `User` 表的外键
- `nature_id`: 对应 `Nature` 表的外键


```python
class UserNature(Base):
    """
    用户、性格中间表
    """
    __tablename__ = "user_nature"
    __table_args__ = {'comment': '用户-性格中间表'}

    user_id = Column(Integer, ForeignKey('user.id'), primary_key=True, nullable=False, index=True)
    nature_id = Column(Integer, ForeignKey('nature.id'), primary_key=True, nullable=False, index=True)
```

`User` 表增加一行，建立连接

```python
natures = relationship('Nature', secondary='user_nature', back_populates='users')
```

`Nature` 表增加一行，建立连接

```python
users = relationship('User', secondary='user_nature', back_populates='natures')
```

#### 数据查询

1. 查询一个用户的所有性格

```python
db.query(User).filter(User.id == 2).first().natures

```

2. 查询一个性格的所有用户

```python
db.query(Nature).filter(Nature.id == 2).first().users
```

#### 数据新增

1. 给一个用户新增多种性格

```python
user = User(user_name='张三', age=20, sex='男')

nature1 = Nature('开朗')
nature2 = Nature(name='阳光')

user.natures = [nature1, nature2]  # 这种写法直接覆盖关联关系
user.natures.extend([nature1, nature2])  # 原有的基础上新增关联关系

db.add(user)
db.commit()
db.refresh(user)

```

2. 给一种性格新增多个用户

```python
db_nature = db.query(Nature).filter(Nature.id == 1).first()

db_user = db.query(User).all()
db_nature.users.extend(db_user)


db.add(db_nature)
db.commit()
db.refresh(db_nature)
```

#### 数据修改

数据修改跟新增逻辑基本差不多，这里不在详细展开

#### 数据删除

1. 删除一个用户

```python
user1 = db.query(User).filter(User.id == 3).first()
db.delete(user1)
db.commit()
```

2. 删除一种性格

```python
nature1 = db.query(Nature).filter(Nature.id == 2).first()
db.delete(nature1)
db.commit()
```


---

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`