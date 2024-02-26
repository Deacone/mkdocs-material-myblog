## 导读

当我们写完了接口，怎么对它快速进行测试呢。

发起接口请求，在`python`体系一般都是使用`requests`库，它能非常快速方便的发起接口请求，并获取返回结果，再结合 `assert` 进行断言。

也可以使用`Pycharm`自带的插件：`HTTP Request`。

在`FastAPI`中，作者封装了一个请求`http(s)` 的请求客户端 `TestClient`。

有了客户端，下一步是运行测试集，这里使用 `pytest` 为例。


## 使用HTTP Request进行测试

1. 新建一个文件，类型选择 `HTTP Request`。
2. 编写测试步骤：
```python
# Test your FastAPI endpoints
GET http://127.0.0.1:9999/
Accept: application/json

###
GET http://127.0.0.1:9999/hello/User
Accept: application/json

###
GET  http://127.0.0.1:9999/items/79
Accept: application/json
Authorization: Bearer 123456

###
POST http://127.0.0.1:9999/token
Content-Type: application/x-www-form-urlencoded

username=johndoe&password=secret

###
POST http://127.0.0.1:9999/users/
Content-Type: application/json

{
    "email": "",
    "password": ""
}
```

这里举了常见的例子：

- 普通`GET`请求
- `form` 格式 `POST`请求
- `json` 格式 `POST`请求
- 带有`token`的 `POST`请求

## 使用 TestClient 进行测试

使用 `TestClient` 之前首先要安装依赖：

```bash
pip install httpx
```

1. 新建一个专门的测试目录: `tests`
2. 新建一个以`test`开头的文件：`test_main.py`
3. 编写测试用例
```python
from fastapi.testclient import TestClient

from .main import app

client = TestClient(app)


def test_read_item():
    response = client.get("/items/foo", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 200
    assert response.json() == {
        "id": "foo",
        "title": "Foo",
        "description": "There goes my hero",
    }


def test_read_item_bad_token():
    response = client.get("/items/foo", headers={"X-Token": "hailhydra"})
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_read_inexistent_item():
    response = client.get("/items/baz", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 404
    assert response.json() == {"detail": "Item not found"}


def test_create_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={"id": "foobar", "title": "Foo Bar", "description": "The Foo Barters"},
    )
    assert response.status_code == 200
    assert response.json() == {
        "id": "foobar",
        "title": "Foo Bar",
        "description": "The Foo Barters",
    }


def test_create_item_bad_token():
    response = client.post(
        "/items/",
        headers={"X-Token": "hailhydra"},
        json={"id": "bazz", "title": "Bazz", "description": "Drop the bazz"},
    )
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_create_existing_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={
            "id": "foo",
            "title": "The Foo ID Stealers",
            "description": "There goes my stealer",
        },
    )
    assert response.status_code == 400
    assert response.json() == {"detail": "Item already exists"}

```
4. 目录结构是这样的
```python
.
├── app
│   ├── __init__.py
│   ├── main.py
|──tests
    ├── __init__.py
    |── test_main.py
```

使用 `pytest` 启动测试：
```bash
# 安装pytest
$ pip install pytest

# 启动测试
$ pytest tests/test_main.py

================ test session starts ================
platform linux -- Python 3.7.9, pytest-5.3.5, py-1.8.1, pluggy-0.13.1
rootdir: /home/user/code/app
plugins: forked-1.1.3, xdist-1.31.0, cov-2.8.1
collected 6 items

████████████████████████████████████████ 100%

test_main.py ......                            [100%]

================= 1 passed in 0.03s =================

# 测试单个用例
$ pytest tests/test_main.py::test_create_existing_item

# 在控制台上打印日志
$ pytest -sv tests/test_main.py::test_create_existing_item

```

## 结语

使用`reuests`库进行测试，当然是非常棒的。

使用 `pytest` + `request` + `allure` + `jenkins` 将可以打造一套流畅的接口测试体系。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`