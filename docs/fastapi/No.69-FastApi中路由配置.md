> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

## 1. 前言

上一章介绍了`FastApi`中的异常处理，本章将介绍路由配置：
- 状态码配置
- tags配置，即路由分组
- 简要概述
- 详细描述
- 接口文档
- 启用接口标记

## 2. 状态码配置

状态码可以直接写`int`类型数据，也可以引用fastapi内置的常量。
```python
from typing import Set, Union

from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()


@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```

在`fastapi.status`类中，定义了接口开发中使用的所有常量，包含了`http`协议的和`ws`协议的。

## 3. tags配置，即路由分组

给接口路由分组，非常简单，添加`tags`参数即可。
```python
from typing import Set, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()


@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
    return item


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

## 4. 简要概述

简要概述也非常简单，添加`summary`参数。
```python
@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item"
)
```

## 5. 详细描述

详细描述非常简单，添加`description`参数。
```python
@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
```

## 6. 接口文档编写

支持使用markdown文本编写。
```python
@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

## 7. 给接口标记是否启用状态

标记启用状态使用关键字：`deprecated`。
```python
@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
