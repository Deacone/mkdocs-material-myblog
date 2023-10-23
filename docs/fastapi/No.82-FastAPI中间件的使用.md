## 导读

中间件在应用程序中有着非常重要的作用。

例如：
- 需要请求执行业务之前统一做权限校验
- 统一给请求的`Response`添加时间戳
- ....

### 如何使用中间件

1. 给需要做中间件的函数添加装饰器：`@app.middleware("http")`。
2. 中间件函数需要接收两个参数：`request` `call_next`.
3. 需要将 `call_next` 的返回对象 `response` 作为中间件的返回体.

### 简单的例子来说明

给返回体的`header`统一添加运行时长：

```python
import time

from fastapi import FastAPI, Request

app = FastAPI()


@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response


@app.get('/hello_world')
async def hello_world():
    return {"code": 0, "message": "success", "data": "hello world"}

```

### 测试

```bash
$ curl -i -X GET --location "http://127.0.0.1:9999/hello_world"
HTTP/1.1 200 OK
date: Sun, 22 Oct 2023 09:44:30 GMT
server: uvicorn
content-length: 51
content-type: application/json
x-process-time: 0.0005321502685546875

{"code":0,"message":"success","data":"hello world"}
```

## 结语

中间件的应用非常广泛，需要在请求处理业务之前统一处理的逻辑或者业务处理完成需要在请求返回之前统一处理的逻辑都可以使用中间件轻松处理。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`