接口中注入依赖第二章。

上一章介绍了方法作为依赖注入，本章讲介绍类作为依赖注入。

## 方法作为依赖有哪些优缺点

方法作为依赖注入，非常方便，直接声明，非常灵活。

```python
async def common_parameters(q: Union[str, None] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
```

使用这种方式在编辑器中编写业务的时候，因为返回的是字典`dict`类型，编辑器不知道key，就没有办法自动补全，造成调用参数不太方面。

## 类作为依赖解决的问题

有一种方法可以实现,使用类的方式，因为返回的是一个类的实例，所以可以用`实例.属性`的方式进行使用。

```python
class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit
```

这种方式跟用方法的例子是完全一样的。但是你可以非常方便的调用它。官方也是比较推荐这种方式的。


## 一个简单的使用例子

```python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Union[str, None] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

在实际使用过程中，`commons`作为入参，业务中可以直接调用它的属性：`commons.skip` `commons.limit` `commons.q`。

因为声明的入参类型和依赖的类是同一个，依赖注入的类还可以简化成这样：

```python
async def read_items(commons: CommonQueryParams = Depends()):
```

`fastapi`会自动化进行转换处理。


***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
