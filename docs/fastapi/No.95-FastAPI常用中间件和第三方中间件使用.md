## 导读

中间件的使用非常重要，有一些需要全局处理的任务，比如安全认证，强制重定向，统一错误日志上报等等。

## 全局重定向中间件HTTPSRedirectMiddleware

将`http` | `ws` 协议的请求自动转化为 `https` | `wss`。

```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

### 测试HTTPSRedirectMiddleware

如果不添加`HTTPSRedirectMiddleware`
```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

# app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

```bash
$ curl -i http://127.0.0.1:9999/
HTTP/1.1 200 OK
date: Mon, 01 Jan 2024 08:01:42 GMT
server: uvicorn
content-length: 25
content-type: application/json

{"message":"Hello World"}
```
如果添加`HTTPSRedirectMiddleware`:
```python
from fastapi import FastAPI
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

# app.add_middleware(HTTPSRedirectMiddleware)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

```bash
$  curl -i http://127.0.0.1:9999/ 
HTTP/1.1 307 Temporary Redirect
date: Mon, 01 Jan 2024 08:01:24 GMT
server: uvicorn
content-length: 0
location: https://127.0.0.1:9999/

```

可以看到返回的状态码不是200，而是307，并且location自动化转为：`https://127.0.0.1:9999/`

## 全局信任host：TrustedHostMiddleware

使用：
```python
from fastapi import FastAPI
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app = FastAPI()

app.add_middleware(
    TrustedHostMiddleware, allowed_hosts=["example.com", "*.example.com"]
)


@app.get("/")
async def main():
    return {"message": "Hello World"}
```

## 数据压缩中间件：GZipMiddleware

使用：
```python
from fastapi import FastAPI
from fastapi.middleware.gzip import GZipMiddleware

app = FastAPI()

app.add_middleware(GZipMiddleware, minimum_size=1000)


@app.get("/")
async def main():
    return "somebigcontent"
```

## 日志数据&性能数据上报中间件：Sentry

当服务上线后，想要对日志进行分析，对服务的性能进行分析，可使用：`sentry`

先注册一个`sentry`账号。

安装：`pip install --upgrade 'sentry-sdk[fastapi]'`。

使用：
```python
from fastapi import FastAPI
import sentry_sdk

sentry_sdk.init(
    dsn="https://YourPublicKey@o0.ingest.sentry.io/0",
    enable_tracing=True,
)

app = FastAPI()

@app.get("/sentry-debug")
async def trigger_error():
    division_by_zero = 1 / 0
```

## 结语

这一章介绍了几个中间件的使用，下一章讲介绍子项目的使用。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`