上一章讲了更新数据接口的编写，本章将介绍接口中注入依赖。

由于依赖注入部分比较重要，而且内容也比较多，分为六章讲解。

本章将介绍依赖注入的好处，简单依赖注入的使用。

## 依赖注入有哪些好处

依赖注入就是将一些公用的代码进行封装，然后在需要引用的地方声明，这样就能减少重复代码，让程序看起来更加有序，可读，灵活性更高。

在一些场景中使用频率较高：  
- 代码重复率较高的地方，例如登录token验证。
- 数据库建立连接。
- 安全、认证、规则校验。
- ...

## 一个简单的使用例子

```python
async def common_parameters(q: Union[str, None] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
```

这里写了一个非常简单的例子，这个函数中声明了三个参数：  
- q: 查询关键词
- skip: 跳过的步长
- limit: 查询结果的条数

这个函数其实就是在API的查询功能中频繁的被使用。  

```python
@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

Tips:
`common_parameters`函数的定义可以是同步，也可以是异步的，FastApi框架会自动识别。

完整代码如下：
```python
from typing import Union

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(
    q: Union[str, None] = None, skip: int = 0, limit: int = 100
):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

进行测试：
```bash
# 测试items
$ curl -X 'GET' \
  'http://127.0.0.1:8000/items_commons?q=hahaha&skip=0&limit=100' \
  -H 'accept: application/json'
  
{
  "q": "hahaha",
  "skip": 0,
  "limit": 100
}

# 测试users
$ curl -X 'GET' \
  'http://127.0.0.1:8000/users_commons?q=hehehe&skip=5&limit=50' \
  -H 'accept: application/json'
  
{
  "q": "hehehe",
  "skip": 5,
  "limit": 50
}
```

## 总结

`FastApi`的这个功能非常重要，掌握了它能简单化的非常便捷的减少你编写代码的代码量，又能快捷的实现你需要的功能。仅仅几行代码的事情，但是在`FastApi`内部已经帮你做好了实现。  

本章只是做了个简单的试验，后续章节将以此展开更深层次的讲解。


***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
