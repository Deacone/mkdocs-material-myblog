## 导读

上一章讲述了简单的获取token的使用，这一章讲完善怎么获取从前端传入的username、password，并怎么获取当前用户。

## 接收用户名及密码

FastAPI封装了一个类专门处理这样的事情：`OAuth2PasswordRequestForm`。

导出这个类：`from fastapi.security import OAuth2PasswordRequestForm`。

使用：
```python
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}
```

逻辑解析：
1. 使用`OAuth2PasswordRequestForm`来校验输入。
2. 用`form_data`来接收用户名及密码。
3. 查询出用户信息赋值给`user_dict`。
4. 判断用户是否存在
5. 生成`UserInDB`对象
6. 获取`hash`密码
7. 比对密码是否正确
8. 返回`token`

## 整体代码及测试
整体代码：
```python
from typing import Union

from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from starlette import status

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}


class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None


class UserInDB(User):
    hashed_password: str


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()


def fake_hash_password(password: str):
    return "fakehashed" + password


def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)


def fake_decode_token(token):
    # This doesn't provide any security at all
    # Check the next version
    user = get_user(fake_users_db, token)
    return user


async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user


async def get_current_active_user(
        current_user: User = Depends(get_current_user)
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user

```

以上代码在`python3.7`环境下经过测试。

### 测试登录
登录正确用户：  
```
curl -X POST --location "http://127.0.0.1:9999/token" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "username=johndoe&password=secret"
```
返回：
```
{
  "access_token": "johndoe",
  "token_type": "bearer"
}
```

登录不存在用户：
```
curl -X POST --location "http://127.0.0.1:9999/token" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "username=johndoe999&password=secret"
```
返回：
```
{
  "detail": "Incorrect username or password"
}
```

### 测试获取当前用户
不传token：
```
curl -X GET --location "http://127.0.0.1:9999/users/me"
```
返回：
```
{
  "detail": "Not authenticated"
}
```

传错误token：
```
curl -X GET --location "http://127.0.0.1:9999/users/me" \
    -H "Authorization: Bearer 111"
```
返回：
```
{
  "detail": "Invalid authentication credentials"
}
```

传正确token：
```
curl -X GET --location "http://127.0.0.1:9999/users/me" \
    -H "Authorization: Bearer johndoe"
```
返回：
```
{
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false,
  "hashed_password": "fakehashedsecret"
}
```

## 结语

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`