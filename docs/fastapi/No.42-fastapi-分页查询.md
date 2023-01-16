>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。

#### 前言

最近在使用FastApi为后端框架进行一个项目的开发，在`FastApi`使用过程中，遇到了分页查询的问题，特此记录一下，以便以后查阅。

分页查询在业务逻辑开发过程中是比较常见的一种需求，在`Django`框架中直接使用其本身携带的插件即可，非常的方便。在`fastapi`框架中，找了一大圈，没有找到相关的说明，但是有一个比较直接的方式可以解决分页查询问题

#### 分页查询初尝试

1. 使用`offset`关键字当做页码
2. 使用`limit`关键字当做每一页显示的数据的条数

查询语句如下：
```python
def get_cases(db: Session, skip: int = 0, limit: int = 100):
    """
    查询用例列表

    :param db:
    :param skip:
    :param limit:
    :return:
    """
    return db.query(Case).filter(Case.is_delete == 0).offset(skip).limit(limit).all()
```

#### 对查询语句进行优化

上面的查询语句存在一个问题，`skip`这个参数不是真正的页码，这里需要进行优化下：

```python
def get_cases(db: Session, skip: int = 0, limit: int = 100):
    """
    查询用例列表

    :param db:
    :param skip:
    :param limit:
    :return:
    """
    return db.query(Case).filter(Case.is_delete == 0).offset(skip * limit).limit(limit).all()

```

在`offset`的地方加上`skip *limit`即可完美解决

#### 使用过程中踩过的坑

1. 在入口函数入参颠倒

```python
@app.get("/cases", response_model=List[case_schema.CaseSchemaDetail])
async def read_cases(db: SessionLocal = Depends(get_db), skip=0, limit=100):
    """
    查询用例列表

    :param db: db session
    :param skip: 页码
    :param limit: 每页数据个数
    :return:
    """
    return case_crud.get_cases(db, skip=skip, limit=limit)
```

优化后：
```python
@app.get("/cases", response_model=List[case_schema.CaseSchemaDetail])
async def read_cases(skip=0, limit=100, db: SessionLocal = Depends(get_db)):
    """
    查询用例列表

    :param db: db session
    :param skip: 页码
    :param limit: 每页数据个数
    :return:
    """
    return case_crud.get_cases(db, skip=skip, limit=limit)
```

2. `skip` 和 `limit` 入参没有声明类型，导致错误：`TypeError: can't multiply sequence by non-int of type 'str'`

优化前：
```python
@app.get("/cases", response_model=List[case_schema.CaseSchemaDetail])
async def read_cases(skip=0, limit=100, db: SessionLocal = Depends(get_db)):
    """
    查询用例列表

    :param db: db session
    :param skip: 页码
    :param limit: 每页数据个数
    :return:
    """
    return case_crud.get_cases(db, skip=skip, limit=limit)

```

优化后：
```python
@app.get("/cases", response_model=List[case_schema.CaseSchemaDetail])
async def read_cases(skip: int = 0, limit: int = 100, db: SessionLocal = Depends(get_db)):
    """
    查询用例列表

    :param db: db session
    :param skip: 页码
    :param limit: 每页数据个数
    :return:
    """
    return case_crud.get_cases(db, skip=skip, limit=limit)

```

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。