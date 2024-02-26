## 导读

在FastAPI中，默认返回的数据是json格式，但是想要返回不同的数据格式怎么办？

例如：xml格式，File文件，html格式等。

本章节讲介绍返回体的几种格式：json格式、xml格式、html格式、文件格式

## json格式

默认情况下，FastAPI返回默认的数据格式是：`JSONResponse` 类的`json`格式。

下面，测试下`Response` `JSONResponse` `UJSONResponse` `ORJSONResponse`。

### 测试返回体 `Response`

测试代码：
```python
import json

from fastapi import FastAPI, Response

app = FastAPI()


@app.get("/items/")
async def read_items():
    return Response(json.dumps({"item_id": "Foo"}), media_type="application/json")
```

测试：
```bash
curl -X GET --location "http://127.0.0.1:9999/items/" \
    -H "Accept: application/json"

{"item_id": "Foo"}
```

### 测试返回体 `JSONResponse`

测试代码：
```python
import json

from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/items/")
async def read_items():
    return JSONResponse({"item_id": "Foo"})

```

测试：
```bash
curl -X GET --location "http://127.0.0.1:9999/items/" \
    -H "Accept: application/json"

{"item_id": "Foo"}
```

相比 `Response`，结果没啥区别

### 返回体 `UJSONResponse`

测试代码：
```python
import json

from fastapi import FastAPI
from fastapi.responses import UJSONResponse, JSONResponse

app = FastAPI()


@app.get("/items/")
async def read_items():
    return UJSONResponse({"item_id": "Foo"})

```

测试：
```bash
curl -X GET --location "http://127.0.0.1:9999/items/" \
    -H "Accept: application/json"

{"item_id": "Foo"}
```

### 返回体 `ORJSONResponse`

测试代码：
```python
import json

from fastapi import FastAPI
from fastapi.responses import ORJSONResponse, JSONResponse

app = FastAPI()


@app.get("/items/", response_class=ORJSONResponse)
async def read_items():
    return ORJSONResponse([{"item_id": "Foo"}])
```

测试：
```bash
curl -X GET --location "http://127.0.0.1:9999/items/" \
    -H "Accept: application/json"

[{"item_id": "Foo"}]
```

总结：`Response` `JSONResponse` `UJSONResponse` `ORJSONResponse`。

看起来结果没什么不一样。`Response` 就不用说了，比较自由。想写什么随意都行。

看一下后三种：

`JSONResponse`:
```python
class JSONResponse(Response):
    media_type = "application/json"

    def __init__(
        self,
        content: typing.Any,
        status_code: int = 200,
        headers: typing.Optional[typing.Dict[str, str]] = None,
        media_type: typing.Optional[str] = None,
        background: typing.Optional[BackgroundTask] = None,
    ) -> None:
        super().__init__(content, status_code, headers, media_type, background)

    def render(self, content: typing.Any) -> bytes:
        return json.dumps(
            content,
            ensure_ascii=False,
            allow_nan=False,
            indent=None,
            separators=(",", ":"),
        ).encode("utf-8")
```

就是对 `Response` 的 `__init__` 和 `render` 进行了重写。

`UJSONResponse` 和 `ORJSONResponse`:
```python
class UJSONResponse(JSONResponse):
    def render(self, content: Any) -> bytes:
        assert ujson is not None, "ujson must be installed to use UJSONResponse"
        return ujson.dumps(content, ensure_ascii=False).encode("utf-8")


class ORJSONResponse(JSONResponse):
    def render(self, content: Any) -> bytes:
        assert orjson is not None, "orjson must be installed to use ORJSONResponse"
        return orjson.dumps(
            content, option=orjson.OPT_NON_STR_KEYS | orjson.OPT_SERIALIZE_NUMPY
        )
```

对 `JSONResponse` 的 `render`进行了重写。

也可以自定义 `***JSONResponse`

例如：
```python
from typing import Any

import orjson
from fastapi import FastAPI, Response

app = FastAPI()


class CustomORJSONResponse(Response):
    media_type = "application/json"

    def render(self, content: Any) -> bytes:
        assert orjson is not None, "orjson must be installed"
        return orjson.dumps(content, option=orjson.OPT_INDENT_2)


@app.get("/", response_class=CustomORJSONResponse)
async def main():
    return {"message": "Hello World"}
```

测试结果：
```bash
curl -X GET --location "http://127.0.0.1:9999/" \      
    -H "Accept: application/json"

{
  "message": "Hello World"
}
```

这样就对返回结果进行了格式化。

## html格式

测试代码：
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()


def generate_html_response():
    html_content = """
    <html>
        <head>
            <title>Some HTML in here</title>
        </head>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
    """
    return HTMLResponse(content=html_content, status_code=200)


@app.get("/items/", response_class=HTMLResponse)
async def read_items():
    return generate_html_response()
```

或者：
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()


@app.get("/items/")
async def read_items():
    html_content = """
    <html>
        <head>
            <title>Some HTML in here</title>
        </head>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
    """
    return HTMLResponse(content=html_content, status_code=200)
```

测试结果：
```bash
curl -X GET --location "http://127.0.0.1:9999/items" \
    -H "Accept: application/json"

    <html>
        <head>
            <title>Some HTML in here</title>
        </head>
        <body>
            <h1>Look ma! HTML!</h1>
        </body>
    </html>
```

## xml格式

测试代码：
```python
from fastapi import FastAPI, Response

app = FastAPI()


@app.get("/legacy/")
def get_legacy_data():
    data = """<?xml version="1.0"?>
    <shampoo>
    <Header>
        Apply shampoo here.
    </Header>
    <Body>
        You'll have to use soap here.
    </Body>
    </shampoo>
    """
    return Response(content=data, media_type="application/xml")
```

测试结果：
```bash
curl -X GET --location "http://127.0.0.1:9999/legacy" \
    -H "Accept: application/json"
<?xml version="1.0"?>
    <shampoo>
    <Header>
        Apply shampoo here.
    </Header>
    <Body>
        You'll have to use soap here.
    </Body>
    </shampoo>        
```

## 文件格式

测试代码：
```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

some_file_path = "large-video-file.mp4"
app = FastAPI()


@app.get("/")
async def main():
    return FileResponse(some_file_path)
```

或者
```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

some_file_path = "large-video-file.mp4"
app = FastAPI()


@app.get("/", response_class=FileResponse)
async def main():
    return some_file_path
```

测试结果：
```bash
curl -X GET --location "http://127.0.0.1:9999/" \
    -H "Accept: application/json"

test file content
```

也可以全局设置一个默认的 `Response`.

例如：`app = FastAPI(default_response_class=ORJSONResponse)`


## 结语

本章介绍了返回的各种类型的`Response`，所以FastApi不只是开发`json`类型的接口。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`