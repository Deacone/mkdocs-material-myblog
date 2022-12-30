> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

#### 前言

前面介绍了`header`和`cookie`的使用方法，也介绍了请求参数的几种传递方法：`query` `body` `path`，今天这篇文章将介绍如何自定义返回字段。

#### response_model 适用于哪些请求方法

`restfulapi` 规范内定义的方法基本都适用，例如：

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`

#### 举个简单的例子

```python
from typing import List, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    return item

```

这里请求参数的定义`model`和返回参数的定义`model`使用了同一个。

#### 请求参数模型和返回参数模型不一样

在很多情况下，请求参数和返回参数是不一样的，一些很敏感的信息也不会直接返回给用户，例如用户密码：
```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Union[str, None] = None


# Don't do this in production!
@app.post("/user/", response_model=UserIn)
async def create_user(user: UserIn):
    return user
```
注意：这里用到了`EmailStr`，要使用这个校验类，需要先安装：`pip install pydantic[email]`。
这个例子中将`password`直接返回给了用户，显然是不合理的。

将返回模型重新定义,去掉`password`：
```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Union[str, None] = None


class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Union[str, None] = None


@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user
```

#### 不返回默认参数

大多数时候，我们都会对参数定义默认值，有时候我们又不想返回默认值参数，怎么办？
例如定义的模型：
```python
class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: float = 10.5
    tags: List[str] = []
```

这个时候假设`items`有三个：
```json
items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}
```

定义主函数：
```python
@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]
```

当`item_id`为`foo`时候，请求信息：
```bash
curl -X GET http://127.0.0.1:8000/items/foo
```
返回字段为：
```json
{"name":"Foo","description":null,"price":50.2,"tax":10.5,"tags":[]}
```
这个时候，会把Item模型中所有的字段都返回，在主函数加上`response_model_exclude_unset=True`，就可以只返回我们指定的字段了。
```python
@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    print(item_id)
    return items[item_id]
```
这时请求返回信息：
```python
$ curl -X GET http://127.0.0.1:8000/items/foo
{"name":"Foo","price":50.2}
```

#### 用response_model_include 和 response_model_exclude快速定义返回字段

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}


@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include={"name", "description"},
)
async def read_item_name(item_id: str):
    return items[item_id]


@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items[item_id]
```
请求`/items/{item_id}/name`：
```sh
$ curl -X GET http://127.0.0.1:8000/items/bar/name
{"name":"Bar","description":"The Bar fighters"}
```

请求`/items/{item_id}/public`:
```sh
$ curl -X GET http://127.0.0.1:8000/items/bar/public
{"name":"Bar","description":"The Bar fighters","price":62.0}
```

注意：`response_model_include={"name", "description"}`的参数类型可以是`list` `tuple`，`fastapi`会将其自动化转换为 `set` 类型

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
