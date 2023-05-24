# FastApi教程-依赖项中使用yield

接口中注入依赖第五章。  
上一章介绍了全局使用`dependencies`，这一章将介绍依赖项不使用`return`，而是使用`yield`。  
在一些场景中使用`yield`将非常方便，例如操作`db`。

## 依赖项中使用yield场景：获取db
```python
async def get_db():
    db = DBSession()
    try:
        yield db
    finally:
        db.close()
```

这段代码首先建立`db session`。  
通过`yield`声明的部分代码会在`response`返回之前被执行。
最后通过关键字`finally`，在`response`返回之后再执行`db.close()`部分。

关于`try`:  
这里使用了`try`来捕获异常，会抛出`try`和`finally`之间的代码。若依赖项中有子依赖项，那么子依赖项中如果有异常也会捕捉到并抛出。  
如果有已知异常还能这样：
```python
async def get_db():
    db = DBSession()
    try:
        yield db
    except SomeException as e:
        print(str(e))
        db.close()
```

或者直接使用`finally`关键字，无需关系中间有什么异常，fastapi会自动捕获：
```python
async def get_db():
    db = DBSession()
    try:
        yield db
    finally:
        db.close()
```

## 多个子依赖项中使用yield

可以放心使用多个子依赖项，fastapi会帮助你确保每一个子依赖的推出码，并且保证树状结构的子依赖能够有一个正确的执行顺序。

```python
from typing import Annotated

from fastapi import Depends


async def dependency_a():
    dep_a = generate_dep_a()
    try:
        yield dep_a
    finally:
        dep_a.close()


async def dependency_b(dep_a: Annotated[DepA, Depends(dependency_a)]):
    dep_b = generate_dep_b()
    try:
        yield dep_b
    finally:
        dep_b.close(dep_a)


async def dependency_c(dep_b: Annotated[DepB, Depends(dependency_b)]):
    dep_c = generate_dep_c()
    try:
        yield dep_c
    finally:
        dep_c.close(dep_b)
```

## 子依赖项中使用yield和HTTPException

注意：在子依赖项中使用`yield`和`HTTPException`，异常必须在`yield`之前，而不是之后，`在yield`之后抛出`HTTPException`异常是不会起到作用的。

因为在 `fastapi` 内，`response` 被发送到客户端之后，再去处理 `HTTPException` 是不会生效的。

## 使用上下文管理器来创建依赖

创建上下文管理器需要两个方法：`__enter__()` 和 `__exit__()`

例如：
```python
class MySuperContextManager:
    def __init__(self):
        self.db = DBSession()

    def __enter__(self):
        return self.db

    def __exit__(self, exc_type, exc_value, traceback):
        self.db.close()

# 创建依赖项
async def get_db():
    with MySuperContextManager() as db:
        yield db
```


***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`


