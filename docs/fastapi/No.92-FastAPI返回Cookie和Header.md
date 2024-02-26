## 导读

本章将介绍返回 `cookie` `header`

## 返回 Cookie

测试代码：
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.post("/cookie/")
def create_cookie():
    content = {"message": "Come to the dark side, we have cookies"}
    response = JSONResponse(content=content)
    response.set_cookie(key="fakesession", value="fake-cookie-session-value")
    return response

```

主要是这行代码：`response.set_cookie(key="fakesession", value="fake-cookie-session-value")`

测试结果：
```bash
curl -X POST --location "http://127.0.0.1:9999/cookie" \
    -H "Accept: application/json"

{"message":"Come to the dark side, we have cookies"}
```

## 返回 Header

测试代码：
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()


@app.get("/headers/")
def get_headers():
    content = {"message": "Hello World"}
    headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}
    return JSONResponse(content=content, headers=headers)
```

核心代码：`headers = {"X-Cat-Dog": "alone in the world", "Content-Language": "en-US"}`

测试结果：
```bash
curl -i -X GET --location "http://127.0.0.1:9999/headers" \
    -H "Accept: application/json"
HTTP/1.1 307 Temporary Redirect
date: Fri, 15 Dec 2023 07:04:26 GMT
server: uvicorn
content-length: 0
location: http://127.0.0.1:9999/headers/

HTTP/1.1 200 OK
date: Fri, 15 Dec 2023 07:04:26 GMT
server: uvicorn
x-cat-dog: alone in the world
content-language: en-US
content-length: 25
content-type: application/json

{"message":"Hello World"}
```



## 结语

返回 Cookie Headers，也是非常方便。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`