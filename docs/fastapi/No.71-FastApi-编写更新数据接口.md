上一章讲了存储数据的转换，本章将介绍如何编写更新数据接口。

更新数据接口可以使用`PUT`，也可以使用`PATCH`。接下来将分别介绍这两种使用方法。

## 使用PUT模式更新数据

使用`PUT`模式比较简单的例子，部分代码如下：

```python
class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

这里使用装饰器`app.put`直接定义即可，也使用到了上一章讲的`jsonable_encoder`，对入参进行转换。

这个例子中已经有的数据并且本次没有变更将不会变更，传参有的字段的值变更了，本次请求将会改变原来的值。

完整代码：

```python
from typing import List, Union

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]


@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

### 对PUT模式更新数据测试

测试数据：

```bash
$ curl -X GET http://127.0.0.1:8000/items/foo
{"name":"Foo","description":null,"price":50.2,"tax":10.5,"tags":[]}

$ curl -X 'PUT' \
  'http://127.0.0.1:8000/items/foo' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "Foo",
  "description": "ddddd",
  "price": 60,
  "tax": 10.5
}'
-------------
{
  "name": "Foo",
  "description": "ddddd",
  "price": 60,
  "tax": 10.5,
  "tags": []
}
```

测试`bar`：  
`bar`的原始字段值：

```json
{"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2}
```

更改`price`字段的值:

```bash
$ curl -X 'PUT' \
  'http://127.0.0.1:8000/items/bar' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "price": 60
}'
------------
{
  "name": null,
  "description": null,
  "price": 60,
  "tax": 10.5,
  "tags": []
}
```

这个测试发现，我们要想对数据进行更新，需要传全部的字段，如果不传字段，将默认这个字段是`Item`模型的默认值。

如果想要实现更改某一个字段，就传入该字段，需要使用`PATCH`。

## 使用PATCH更改数据

如果想要传入某个字段值就更改某个字段原先的值，需要使用函数`exclude_unset`。\
比如：`item.dict(exclude_unset=True)`。

关键代码如下：

```python
@app.patch("/items2/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```
解析：  

`stored_item_data = items[item_id]`: 查询要更新的`item`。  

`stored_item_model = Item(**stored_item_data)`: 将`item`转换为`Item`模型。 

`update_data = item.dict(exclude_unset=True)`: 提取要更新的字段。 

`updated_item = stored_item_model.copy(update=update_data)`: 更新模型。

### 对PATCH模式进行测试：

`bar`原先的值：

```json
{"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2}
```

发起请求：

```bash
curl -X 'PATCH' \
  'http://127.0.0.1:8000/items2/bar' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "price": 70
}'
-----------
{
  "name": "Bar",
  "description": "The bartenders",
  "price": 70,
  "tax": 20.2,
  "tags": []
}
```

可以看到只有`price`字段发生了变更。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
