## 导读

当一个项目比较大的时候，随着API的数量不断增加，管理起来也会增加难度，有一种方法就是另起新项目，进行拆分。

当启动一个新项目时候，可以独立出来项目的API，项目的文档等等。

也可以将新项目挂载到主项目上。

## 挂载子项目

测试代码：
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/app")
def read_main():
    return {"message": "Hello World from main app"}


subapi = FastAPI()


@subapi.get("/sub")
def read_sub():
    return {"message": "Hello World from sub API"}


app.mount("/subapi", subapi)
```

分析：
1. 主项目的使用方式不变。
2. 新实例化出一个`app`: `subapi`。
3. 编写子项目的视图函数。
4. 将子项目挂载到主项目上：`app.mount("/subapi", subapi)`。

这样子项目就拥有了自己独立的管理系统。例如：`Header`的管理、文档的管理、路由的管理等。

## 测试

请求主APP的API：
```bash
$ curl -i http://127.0.0.1:9999/app
HTTP/1.1 200 OK
date: Mon, 01 Jan 2024 08:46:58 GMT
server: uvicorn
content-length: 39
content-type: application/json

{"message":"Hello World from main app"}
```

请求子APP的API：
```bash
$ curl -i http://127.0.0.1:9999/subapi/sub
HTTP/1.1 200 OK
date: Mon, 01 Jan 2024 08:49:34 GMT
server: uvicorn
content-length: 38
content-type: application/json

{"message":"Hello World from sub API"}
```

## 结语

可以看到，子APP完全拥有主APP的所有属性，并且保持跟主APP隔离性。主APP的改动，不会影响到子APP。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`