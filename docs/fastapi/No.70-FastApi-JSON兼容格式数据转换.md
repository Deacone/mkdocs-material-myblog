上一章介绍了`FastApi`中的路由配置，本章将介绍数据存储中的转换。  

有些情况下，我们需要对数据类型进行转换以方便存储到数据库中。  

像`Pydantic model`类型、`datetime`类型数据就不方便直接存储到数据库中，需要对其进行转换成`JSON`类型的数据才能方便存储。  

`FastApi`提供了一个函数专门来处理这种事情：`jsonable_encoder`。

## jsonable_encoder的使用

```python
from datetime import datetime
from typing import Union

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}


class Item(BaseModel):
    title: str
    timestamp: datetime
    description: Union[str, None] = None


app = FastAPI()


@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)
    fake_db[id] = json_compatible_item_data
```

这个例子中`item`实例将会被直接转换为`JSON`类型数据，`timestamp`字段将会被转换为`str`类型数据，例如：

```python
import datetime

from main import Item
from fastapi.encoders import jsonable_encoder

if __name__ == '__main__':
    item = Item(title="测试一下", timestamp=datetime.datetime.now(), description="这里是描述")
    print(jsonable_encoder(item))
```

运行这段代码，得到：

```bash
{'title': '测试一下', 'timestamp': '2023-02-02T22:24:37.379416', 'description': '这里是描述'}
```

这样就方便我们对数据进行后续存储操作啦。  

## 转换结果处理

转换的结果可以直接使用`json.dumps()`进行转换。

```python
import datetime
import json

from main import Item
from fastapi.encoders import jsonable_encoder

if __name__ == '__main__':
    item = Item(title="测试一下", timestamp=datetime.datetime.now(), description="这里是描述")
    print(json.dumps(jsonable_encoder(item)))
```

运行这段代码得到：

```bash
{"title": "\u6d4b\u8bd5\u4e00\u4e0b", "timestamp": "2023-02-02T22:28:29.122139", "description": "\u8fd9\u91cc\u662f\u63cf\u8ff0"}
```


***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`