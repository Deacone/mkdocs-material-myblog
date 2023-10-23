## 导读

上一章讲述了怎么接收前端传的用户名和密码，获取token，再使用token进行登录。  
但是token过于简单，这一章将引入JWT格式token。

## 为啥使用JWT

JWT 全称 "JSON Web Tokens"。

使用JWT生成的token像这样：`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
`

使用JWT可以提高安全性，可以给token添加过期时间，让token失效，还可以实现用户权限的控制。

## 安装依赖库

1. python-jose: `pip install "python-jose[cryptography]"`
2. passlib: `pip install "passlib[bcrypt]"`

## 使用JWT

### 1. 生成秘钥

```bash
$ openssl rand -hex 32
c0ec2acb6f93fd2258513511b1d9ce76cd86d9e7364f1ba2858a3c8973b6d488

```

### 2. 设置签名算法

ALGORITHM = "HS256"

### 3. 设置token过期时间

ACCESS_TOKEN_EXPIRE_MINUTES = 30

### 4. 定义Token

```python
class Token(BaseModel):
    access_token: str
    token_type: str
```

### 5. 创建 token

```python
def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

```

### 6. 获取当前用户的方法

```python
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user
```

### 7. 定义post请求获取access_token

```python
@app.post("/token", response_model=Token)
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}
```

请注意看：
```python
access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
```
data字段使用了字典键： `sub`，它的值需要是唯一的。

完整的测试代码如下：

```python
from datetime import datetime, timedelta
from typing import Annotated, Union

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# to get a string like this run:
# openssl rand -hex 32
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    }
}


class Token(BaseModel):
    access_token: str
    token_type: str


class TokenData(BaseModel):
    username: Union[str, None] = None


class User(BaseModel):
    username: str
    email: Union[str, None] = None
    full_name: Union[str, None] = None
    disabled: Union[bool, None] = None


class UserInDB(User):
    hashed_password: str


pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()


def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    return pwd_context.hash(password)


def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)


def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user


async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)]
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token", response_model=Token)
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return current_user


@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return [{"item_id": "Foo", "owner": current_user.username}]

```

以上代码均在`python3.9`环境下通过测试


## 测试

### 1. 测试登录
登录正确用户：  
```
curl -X POST --location "http://127.0.0.1:9999/token" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "username=johndoe&password=secret"
```
返回：
```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huZG9lIiwiZXhwIjoxNjk3OTY3MDYxfQ.xzbdevhY45Svdv7x3ciIUTpdc25Cz9JRBqT8m22lNfs",
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

### 2. 测试获取当前用户
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
    -H "Authorization: Bearer johndoe111"
```
返回：
```
{
  "detail": "Could not validate credentials"
}
```

传正确token：
```
curl -X GET --location "http://127.0.0.1:9999/users/me" \
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huZG9lIiwiZXhwIjoxNjk3OTY3NDU2fQ.z4eKldnS37bTH79OEJgDvaP7DJKTAkZ03Lct6m1pJHo"
```
返回：
```
{
  {
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false
}
}
```

## 结语

至此，FastAPI相关安全方面的 `JWT` `OAuth2`开发就完成了。

`OAuth2`中有一个概念叫：`scopes`，利用这个改变，你将很轻松的向 `JWT` 令牌中添加权限，实现权限控制。

这个概念将在后续更新中逐一体现。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`