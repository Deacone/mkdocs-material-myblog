## 导读

上一章介绍了自动化迁移脚本，将数据模型迁移到数据库，映射到表结构。

本章将创建两个接口：
 - 创建用户
 - 查询用户
 
 从用户的创建和查询来介绍数据的创建和查询过程。

 ### 创建schemas

`sql_app/user/schemas.py`

 用schemas来管理接口的输入和输出是非常方便的，有利于减少让代码更加简洁。

 ```python
 from typing import Union

from pydantic import BaseModel


class UserBase(BaseModel):
    email: str


class UserCreate(UserBase):
    password: str


class User(UserBase):
    id: int
    is_active: bool
    items: list[Item] = []

    class Config:
        orm_mode = True

 ```

 ### 创建crud

 `sql_app/user/crud.py`

 这个文件主要做业务处理

 ```python
 from sqlalchemy.orm import Session

from . import models, schemas


def get_user(db: Session, user_id: int):
    return db.query(models.User).filter(models.User.id == user_id).first()


def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(email=user.email, hashed_password=fake_hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
 ```

 ### database添加依赖

 `sql_app/database.py`

 这里添加依赖，获取db session。最终使用完毕进行回收关闭连接。

 ```python
 def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
 ```

### 在路由视图中调用

`sql_app/main.py`

先看创建用户：  
1. 定义路由：`/users/`,定义返回数据字段：`schemas.User`
2. 定义入参：`schemas.UserCreate`,定义依赖：`db: Session = Depends(get_db)`
3. 通过用户邮箱查询用户： `crud.get_user_by_email(db, email=user.email)`
4. 判断用户是否存在：若存在返回 状态码 `400`，若不存在返回状态码 `200`,并创建用户。

```python
@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)
```

再看查询单个用户详情：  
1. 定义路由：`/users/{user_id}`, 定义返回字段：`schemas.User`
2. 定义入参: `user_id`，定义依赖：`db: Session = Depends(get_db)`
3. 查询用户：`crud.get_user(db, user_id=user_id)`
4. 判断用户是否存在，若不存在返回状态码：`404`，若存在返回状态码：`200`，并返回用户`response_model`信息

```python
@app.get("/users/{user_id}", response_model=schemas.User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = crud.get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user
```

## 结语

本章从创建用户、查询单个用户详情的例子介绍了编写接口时跟数据库的交互流程。这里只简单介绍了一个数据模型的使用，两个接口跟数据库的交互，当接口数量越来越多的情况下怎么管理呢？

下一章将介绍在一些大型项目中如何去管理规划项目文件。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`