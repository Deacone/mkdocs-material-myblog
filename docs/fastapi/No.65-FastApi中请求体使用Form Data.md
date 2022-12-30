> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

#### 前言

上一章介绍了请求返回的响应码使用，本章将介绍如何使用 Form Data作为请求体。

#### 使用Form Data作为请求体

正常情况下，fastapi默认请求体内容类型为：`application/json`，默认会在请求头中加上`Content-Type`:
```bash
$ curl http://127.0.0.1:8000
HTTP/1.1 404 Not Found
date: Mon, 26 Dec 2022 10:51:25 GMT
server: uvicorn
content-length: 22
content-type: application/json

{"detail":"Not Found"}
```
##### 安装python-multipart
要使用Form Data，要先安装：`python-multipart`
```bash
pip install python-multipart
```

##### 定义Form

1. 从fastapi导出Form
2. 在函数中定义

```python
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

##### 测试

使用json格式请求：
```bash
$ curl -i -X POST -d '{"username":"hope","password":"123456"}' http://127.0.0.1:8000/login/
{
  "detail": [
    {
      "loc": [
        "body",
        "username"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    },
    {
      "loc": [
        "body",
        "password"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```
这个时候就报错了，说字段`username` `password` 是必填参数.

使用`Form`格式请求：
```bash
 $ curl -i -X POST -d "username=hope&password=123456" http://127.0.0.1:8000/login/
 
HTTP/1.1 200 OK
date: Mon, 26 Dec 2022 11:08:51 GMT
server: uvicorn
content-length: 19
content-type: application/json

{"username":"hope"}
```


***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
