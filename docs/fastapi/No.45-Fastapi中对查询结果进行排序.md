>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。

#### 前言

最近在使用FastApi为后端框架进行一个项目的开发，在`FastApi`使用过程中，遇到了一个排序的需求，一般这个需求比较常见，所以进行了总结，以便后续查阅。
下面是通过官网文档阅读，得到的两种方案，仅供参考。

#### 第一种方案：在`Query`对象基础上进行排序

##### order_by 中传入对象

升序
```python
db.query(Step).filter(Step.page == db_page).order_by(Step.order_number).all()
```

降序
```python
db.query(Step).filter(Step.page == db_page).order_by(Step.order_number.desc()).all()
```

##### order_by 中传入字符串

升序
```python
db.query(Step).filter(Step.page == db_page).order_by('create_time').all()
```

降序
```python
db.query(Step).filter(Step.page == db_page).order_by(desc('create_time')).all()
```

#### 第二种方案：定义模型时声明

模型代码：
```python
class User(Base):　　
　　__tablename__ = "user"　　
　　id = Column(Integer , primary_key=True , autoincrement=True)　　
　　name = Column(String(50) , nullable=False)　　
　　create_time = Column(DateTime , nullable=False , default=datetime.now)
　　
　　__mapper_args__ = {"order_by": create_time.desc()}
```

在模型中加上这一行：
```python
__mapper_args__ = {"order_by": create_time.desc()}
```

再进行查询：
```python
results = session.query(User).all()
```

这两种方案都可以实现排序的需求，如果项目不大，使用的数据量级一般，这样写是ok的，但是遇到数据量级比较大的时候，就需要进行优化啦。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。