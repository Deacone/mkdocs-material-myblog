## 导读

`Request` 类是`FastAPI`中比较特殊的类。

在`FastAPI`中，当按照规则将参数传入，它会自动将参数进行解析，比如`Headers` `Cookies` `Path` `Query`等。

但当需要获取客户端请求的IP/Host时候，怎么处理呢？

## 使用Request获取客户端的IP

获取客户端的IP使用：`request.client.host`。

测试代码：
```python
from fastapi import FastAPI, Request

app = FastAPI()


@app.get("/items/{item_id}")
def read_root(item_id: str, request: Request):
    client_host = request.client.host
    return {"client_host": client_host, "item_id": item_id}

```

## 测试

```bash
$  curl http://127.0.0.1:9999/items/1
{"client_host":"127.0.0.1","item_id":"1"}

```

## 扩展

查看`Request`的帮助文档，可以看到，它还有非常多有用的功能。

例如：获取用户`token`，获取当前用户信息、获取当前`session`信息等等。

如果想要存储一些临时信息到当前请求，可以使用`request.state`。

例如：存储请求开始时间：

`request.state.time_started = time.time()`

```bash
>>> from fastapi import FastAPI, Request
>>> help(Request)
Help on class Request in module starlette.requests:

class Request(HTTPConnection)
 |  Request(scope: MutableMapping[str, Any], receive: Callable[[], Awaitable[MutableMapping[str, Any]]] = <function empty_receive at 0x7fcc8726b790>, send: Callable[[MutableMapping[str, Any]], Awaitable[NoneType]] = <function empty_send at 0x7fcc872ace50>)
 |  
 |  Method resolution order:
 |      Request
 |      HTTPConnection
 |      collections.abc.Mapping
 |      collections.abc.Collection
 |      collections.abc.Sized
 |      collections.abc.Iterable
 |      collections.abc.Container
 |      typing.Generic
 |      builtins.object
...
|  ----------------------------------------------------------------------
 |  Readonly properties inherited from HTTPConnection:
 |  
 |  app
 |  
 |  auth
 |  
 |  base_url
 |  
 |  client
 |  
 |  cookies
 |  
 |  headers
 |  
 |  path_params
 |  
 |  query_params
 |  
 |  session
 |  
 |  state
 |  
 |  url
 |  
 |  user
 |  
 |  ----------------------------------------------------------------------

```

## 结语

实践出真知。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`