接口中注入依赖第四章。

上一章介绍了依赖嵌套，这一章将介绍在路由中使用`dependencies`。

## 遇到的问题

有时候依赖函数没有返回值，或者依赖项不需要返回值。  
例如：一个项目中有非常多接口，这时需要在每一个接口的`Header`中添加一个参数 `X-Token`，并对这个参数进行验证，怎么做呢？

实现的方式有非常多，这里使用依赖项来实现。

实现步骤：

##### 1. 声明认证方法

```python
async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")
```

##### 2. 在接口路由中使用

```python
@app.get("/items/", dependencies=[Depends(verify_token))
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

##### 3. 使用多个依赖项

```python
from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()


async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


async def verify_key(x_key: str = Header()):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key


@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]

```

这样就在接口`/items/`中的`Header`添加了`X-Token`和`X-Key`的验证。

## 总结

在使用依赖项的时候可以使用单个、多个结合使用。

依赖项可以有返回值，也可以没有返回值。

依赖项可以使用`raise`来主动抛出异常。


***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
