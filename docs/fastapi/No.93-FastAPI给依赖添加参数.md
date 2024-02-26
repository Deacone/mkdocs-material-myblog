## 导读

`Dependencies` 在FastAPI中是非常重要的一种使用方法。

本章将介绍怎么给 `Dependencies` 类传递参数。

## 可调用的实例

想要给 `Dependencies` 传递参数，首先要定义一个 `class`, 声明 `__call__` 方法。

例如：
```python
class FixedContentQueryChecker:
    def __init__(self, fixed_content: str):
        self.fixed_content = fixed_content

    def __call__(self, q: str = ""):
        if q:
            return self.fixed_content in q
        return False
```

在FastAPI中调用：
```python
from fastapi import Depends, FastAPI

app = FastAPI()


class FixedContentQueryChecker:
    def __init__(self, fixed_content: str):
        self.fixed_content = fixed_content

    def __call__(self, q: str = ""):
        if q:
            return self.fixed_content in q
        return False


checker = FixedContentQueryChecker("bar")


@app.get("/query-checker/")
async def read_query_check(fixed_content_included: bool = Depends(checker)):
    return {"fixed_content_in_query": fixed_content_included}
```

分析：
1. 声明一个app
2. 声明一个类：创建这个类需要一个参数：`fixed_content`, 这是一个可以被调用的类。
3. 在视图函数中使用依赖：`checker`.
4. 在请求时带上query参数，实例`checker`将接收query参数

## 测试

不传query参数：
```bash
curl -X GET --location "http://127.0.0.1:9999/query-checker/"

{"fixed_content_in_query":false}
```

传query参数，且参数包含 `bar`:
```bash
curl -X GET --location "http://127.0.0.1:9999/query-checker/?q=bar0"
{"fixed_content_in_query":true}
```
传query参数，且参数不包含 `bar`:
```python
curl -X GET --location "http://127.0.0.1:9999/query-checker/?q=foo" 
{"fixed_content_in_query":false}
```

几个场景就测试完啦。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`