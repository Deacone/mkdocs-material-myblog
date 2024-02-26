## 导读

在前后端分离的项目中，跨域访问资源是很平常的事情了。当前端项目有接口请求向后端发起的时候，就需要后端项目配置允许跨源访问。
本片文章将分享在FastAPI中怎么使用跨域访问。

## 配置CROS

配置CROS也很简单方便，需要引入中间件：`CORSMiddleware`。

### 1. 创建一个列表

创建一个允许跨域访问的列表。

```python
origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]
```

### 2. 添加中间件

给FastAPI的app实例添加一个中间件，全局处理CROS问题。

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 3. 添加一个接口路由请求

```python
@app.get("/")
async def main():
    return {"message": "Hello World"}
```

整体代码如下：
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost",
    "http://localhost:8080",
    "https://blog.csdn.net"
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
async def main():
    return {"message": "Hello World"}

```

## 测试

在浏览器的控制台输入如下代码：
```js
var token= "eyJhbGciOiJIUzUxMiJ9.eyJsb2dpbl91c2VyX2tleV86IjoiMGY4YTlmYzgtODZmMi00NjM3LWFlNGUtYTdmYTQyMzIzMmYwIn0.9NR3VRvgOg2USyCMyUaBEpZKETj3tn9eIdnQo7vXQH_0hwqWOKAkSxCYNtYOnPoRLEOaJQTVdq22grvvqYU4Fw";
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://127.0.0.1:9999');
xhr.setRequestHeader("x-access-token",token);
xhr.send(null);
xhr.onload = function(e) {
    var xhr = e.target;
    console.log(xhr.responseText);
}
```

返回如下结果说明跨域请求没有成功：

```js
Access to XMLHttpRequest at 'http://127.0.0.1:9999/' from origin 'https://blog.csdn.net' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

返回如下结果说明跨域请求成功：

```js
ƒ (e) {
    var xhr = e.target;
    console.log(xhr.responseText);
}
------------------------------------
{"message":"Hello World"}
```

## 结语

跨域请求在前后端分离的项目中是非常重要的一环，在FastAPI中只需要简单的配置即可。

接下来的FastAPI系列文章将讲解`DataBase`的连接和数据模型的建立。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`