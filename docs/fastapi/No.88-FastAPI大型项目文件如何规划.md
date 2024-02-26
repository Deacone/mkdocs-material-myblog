## 导读

在大型项目中，文件众多，多个模块并行开发，一个项目接口也有很多，怎么管理起来呢？

 ## 初始化一个项目

 一个初始化的项目目录应该是这样的：

 ```
app
├── __init__.py
├── dependencies.py
├── main.py
├── admin
│   ├── __init__.py
│   └── routers.py
├── items
│   ├── __init__.py
│   ├── crud.py
│   ├── models.py
│   ├── routers.py
│   └── schemas.py
└── users
    ├── __init__.py
    ├── crud.py
    ├── models.py
    ├── routers.py
    └── schemas.py
 ```

根目录文件：  
 - app: 是根目录，这个根目录下要有 `__init__.py`文件，代表是一个 `python` 的模块目录。
 - main.py: 是项目的入口模块
 - dependencies.py: 项目的所有依赖模块

 目录文件`internal`: 这是内部模块  
   - admin.py: 管理台接口

目录文件 `users` 和 `items` 类似，这里以 `users` 为例：  
- crud.py: 跟数据库交互的，对数据库数据增删改查。
- models.py: 数据模型，定义数据库表。
- routers.py: 接口路由模块，定义接口路由和接口逻辑。
- schemas.py: 输入输出数据校验模块。

这种写法比较`Django`。。。


## 模块之间怎么协作？

### main.py
```python
from fastapi import Depends, FastAPI

from .dependencies import get_query_token, get_token_header
from bigger_app.items.routers import router as items_router
from bigger_app.users.routers import router as users_router
from bigger_app.admin.routers import router as internal_router

app = FastAPI(dependencies=[Depends(get_query_token)])

app.include_router(
    users_router,
    prefix="/users"
)
app.include_router(
    items_router,
    prefix="/items"
)
app.include_router(
    internal_router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)


@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}

```

1. app中添加了全局依赖：`get_query_token`
2. 对各个模块进行了路由设置 `app.include_router()`

### dependencies.py

编写依赖模块：

```python
from typing import Annotated

from fastapi import Header, HTTPException


async def get_token_header(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")


async def get_query_token(token: str):
    if token != "jessica":
        raise HTTPException(status_code=400, detail="No Jessica token provided")
```

定义了两个函数：
- 检测header中 `x_token` 的值的正确性
- 检测`token`的正确性

### users/routers.py

编写users模块的路由：
```python
from fastapi import APIRouter

router = APIRouter(
    tags=["users"]
)


@router.get("/")
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


@router.get("/me")
async def read_user_me():
    return {"username": "fakecurrentuser"}


@router.get("/{username}")
async def read_user(username: str):
    return {"username": username}

```

使用APIRouter类来定义路由，使用tags来给这个模块打标签。

### items/routers.py

编写items模块的路由：
```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

router = APIRouter(
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}


@router.get("/")
async def read_items():
    return fake_items_db


@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}


@router.put(
    "/{item_id}",
    tags=["custom"],
    responses={403: {"description": "Operation forbidden"}},
)
async def update_item(item_id: str):
    if item_id != "plumbus":
        raise HTTPException(
            status_code=403, detail="You can only update the item: plumbus"
        )
    return {"item_id": item_id, "name": "The great Plumbus"}

```

### admin/routers.py

编写admin模块的路由：
```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

router = APIRouter(
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)


@router.get("/")
async def main():
    return {"name": "I am Admin"}

```

## 得到的接口

经过上面的布置之后，因为每一个模块都打了tag，接口文档也将按照tag进行分类展示，将得到如下接口：
- users
  - /users/
  - /users/me
  - /users/{username}
- items
  - /items/
  - /items/{item_id}
  - /items/{item_id}
- admin
  - /admin/
- default
  - / 


## 结语

在一个文件比较多的大型项目中，合理规划目录结果能够加快开发人员的快速投入和理解，提升效率。

以上只是一个推荐的方案。

FastAPI也有工具可以直接生成大型项目的目录结构，后面会提及。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`