接口中注入依赖第三章。

上一章介绍了类作为依赖注入，本章讲介绍依赖嵌套。

## 为什么要用嵌套

有时候依赖项比较多的时候，嵌套可以非常灵活的自由组合，增加代码的复用量。

比如在原先的依赖项上增加一些功能。

## 简单的例子

```python
def query_extractor(q: Union[str, None] = None):
    return q


def query_or_cookie_extractor(
    q: str = Depends(query_extractor),
    last_query: Union[str, None] = Cookie(default=None),
):
    if not q:
        return last_query
    return q
```
`query_or_cookie_extractor` 使用了两个参数：

- `q`: 使用了 `query_extractor`依赖项，将依赖项获取到值赋给`q`。
- `last_query`: 声明了可选参数，如果q没有值，就是用上一次查询使用过的`Cookie`里面保存的值


## 使用依赖项

```python
@app.get("/items/")
async def read_query(query_or_default: str = Depends(query_or_cookie_extractor)):
    return {"q_or_cookie": query_or_default}
```

跟正常的使用方式一样，直接在`Depends`中填写依赖项的名称。

## 完整使用

```python
from typing import Union

from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: Union[str, None] = None):
    return q


def query_or_cookie_extractor(
    q: str = Depends(query_extractor),
    last_query: Union[str, None] = Cookie(default=None),
):
    if not q:
        return last_query
    return q


@app.get("/items/")
async def read_query(query_or_default: str = Depends(query_or_cookie_extractor)):
    return {"q_or_cookie": query_or_default}
```

## 总结

嵌套使用依赖项可以让我们的业务依赖更加灵活，随心所欲搭配。



***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
