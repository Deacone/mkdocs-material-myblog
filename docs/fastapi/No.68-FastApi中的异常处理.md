> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

## 1. 前言

上一章介绍了使用Form 和 文件上传，本章将介绍请求过程中的异常处理。  

很多情况下，当请求参数不符合规范时，我们需要对请求异常进行处理，返回相应的状态码和提示。这样方便用户快速知道哪里的参数出错了

经常使用的情形如下：  

- 客户端没有执行操作的权限
- 客户端没有访问资源的权限
- 客户端要访问的资源不存在
- 等

这些情况，通常情况下要返回400 ~ 499的状态码，表示客户端发生错误。

## 2. HTTPException

向客户端返回 HTTP 错误响应，可以使用 HTTPException。

### 2.1 导入HTTPException

```python
from fastapi import FastAPI, HTTPException
```

### 2.2 使用 HTTPException

```python
raise HTTPException(status_code=404, detail="Item not found")
```

### 2.3 完整使用例子
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```
这里使用`raise`将异常抛出来，当程序执行到这里的时候，主动抛出异常，将不再执行后续的操作，将异常信息和状态码返回给客户端。

### 2.4 测试
```bash
$ curl http://127.0.0.1:8000/items/foo
{"item":"The Foo Wrestlers"}
```

```bash
$ curl http://127.0.0.1:8000/items/foo2
{"detail":"Item not found"}
```

触发 `HTTPException` 时，可以用参数 `detail` 传递任何能转换为 `JSON` 的值，不仅限于 `str`。

还支持传递 `dict`、`list` 等数据结构。

`FastAPI` 能自动处理这些数据，并将之转换为 `JSON`。

## 3. 添加自定义请求头

在一些安全场景下，在请求头中添加错误信息是非常有用的，这种错误信息可能不需要返回给响应体`response body`。  
例如：
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "Something error"},
        )
    return {"item": items[item_id]}
```

测试：
```bash
$ curl -i http://127.0.0.1:8000/items-header/foo2
HTTP/1.1 404 Not Found
date: Sun, 15 Jan 2023 01:45:54 GMT
server: uvicorn
x-error: Something error
content-length: 27
content-type: application/json

{"detail":"Item not found"}
```

## 4. 使用自定义异常处理器

自定义异常处理器，需要使用 `Starlette` 的异常工具。  
假设要触发的自定义异常叫作 UnicornException。  
且需要 FastAPI 实现全局处理该异常。  

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```
测试：
```bash
$ curl -i http://127.0.0.1:8000/unicorns/yolo
HTTP/1.1 418
date: Sun, 15 Jan 2023 01:56:29 GMT
server: uvicorn
content-length: 63
content-type: application/json

{"message":"Oops! yolo did something. There goes a rainbow..."}
```

## 5. 使用自定义异常处理器覆盖默认异常处理器

### 5.1 自定义请求验证异常
需要使用`RequestValidationError`来作为装饰器：
```python
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)
```
这样就覆盖了原来的异常请求返回信息。
测试代码：
```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

如果不覆盖，测试结果：
```bash
$ curl -i http://127.0.0.1:8000/items/foo
HTTP/1.1 422 Unprocessable Entity
date: Sun, 15 Jan 2023 07:05:58 GMT
server: uvicorn
content-length: 104
content-type: application/json

{"detail":[{"loc":["path","item_id"],"msg":"value is not a valid integer","type":"type_error.integer"}]}
```
覆盖之后，测试结果：
```bash
$ curl -i http://127.0.0.1:8000/items/foo
HTTP/1.1 400 Bad Request
date: Sun, 15 Jan 2023 07:07:06 GMT
server: uvicorn
content-length: 103
content-type: text/plain; charset=utf-8

1 validation error for Request
path -> item_id
  value is not a valid integer (type=type_error.integer)
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
