接口中注入依赖第四章。

上一章介绍了在路由中使用`dependencies`，这一章将介绍设置全局依赖项。

## 遇到的问题

上一章我们将某一个接口添加上依赖，如果整个应用都需要这个依赖怎么办呢

## 设置全局依赖

只需要将依赖设置到app对象上即可：

```python
app = FastAPI(dependencies=[Depends(verify_token), Depends(verify_key)])
```

是不是很简单。。。

## 总结

接口入口函数的入参设置依赖，接口路由中设置依赖（dependencies关键字），全局设置依赖需要根据实际应用合理使用方能发挥出最大能力。

完成的全局设置依赖示例如下：

```python
from fastapi import Depends, FastAPI, Header, HTTPException


async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


async def verify_key(x_key: str = Header()):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key


app = FastAPI(dependencies=[Depends(verify_token), Depends(verify_key)])


@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]


@app.get("/users/")
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
