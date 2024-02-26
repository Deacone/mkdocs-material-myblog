## 导读

异步接口在应用中也非常常见，使用场景包括文件上传、下载，发送短信验证码、发送邮件，发起繁琐数据计算等。

这个时候由于任务需要处理的时间不确定，不能让用户一直等待，需要立马给用户一个反馈。

下面将通过两个简单的例子来说明。

## 使用异步写文件

这里使用 `FastAPI` 封装的 `BackgroundTasks` 来实现。

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}

```

解析：

1. 定义一个写文件的函数，需要两个入参：`email` | `message`。
2. 在视图函数中定义入参 `email` | `background_tasks` 。
3. 使用  `background_tasks.add(function, *args, **kwargs)` 来发起异步任务。
4. 注意这里视图函数使用了 `async`。

### 测试

```bash
$ curl -X POST --location "http://127.0.0.1:9999/send-notification/123456.qq.com" \
    -H "Content-Type: application/json"

{"message":"Notification sent in the background"}
```

1. 接口正常返回数据：`{"message":"Notification sent in the background"}`
2. 本地会创建一个文件：`log.txt`
3. `log.txt` 文件的内容是：`notification for 123456.qq.com: some notification`


## 在依赖中使用异步操作

```python
from typing import Union

from fastapi import BackgroundTasks, Depends, FastAPI

app = FastAPI()


def write_log(message: str):
    with open("log.txt", mode="a") as log:
        log.write(message)


def get_query(background_tasks: BackgroundTasks, q: Union[str, None] = None):
    if q:
        message = f"found query: {q}\n"
        background_tasks.add_task(write_log, message)
    return q


@app.post("/send-notification/{email}")
async def send_notification(
    email: str, background_tasks: BackgroundTasks, q: str = Depends(get_query)
):
    message = f"message to {email}\n"
    background_tasks.add_task(write_log, message)
    return {"message": "Message sent"}
```

解析：

1. 定义个写完见的操作 `write_log`。
2. 定义一个依赖 `get_query`，接收一个对象：`BackgroundTasks`。
3. 定义视图函数 `send_notification`, 使用 `async`, 接收一个对象 `BackgroundTasks`。
4. 在依赖函数和视图函数中分别启动异步进程。

### 测试

不触发依赖的异步任务操作：

```bash
$ curl -X POST --location "http://127.0.0.1:9999/send-notification/123456.qq.com" \
    -H "Content-Type: application/json"

{"message":"Message sent"}
```

测试结果解析：

1. 请求参数只带 `email`
2. 正常返回数据 `{"message":"Message sent"}`
3. 文件 `log.txt` 写入内容： `message to 123456.qq.com`

触发依赖的异步任务操作，参数中带 `q`：

```bash
$ curl -X POST --location "http://127.0.0.1:9999/send-notification/123456.qq.com?q=IAmBigMan" \     
    -H "Content-Type: application/json"

{"message":"Message sent"}
```

1. 请求参数带 `email` 和 `q`
2. 正常返回数据 `{"message":"Message sent"}`
3. 文件 `log.txt` 写入内容：
```txt
found query: IAmBigMan
message to 123456.qq.com
```

## 结语

在本例中使用简单的 `BackgroundTasks` 来实现异步操作。它所需要的异步运行数据和状态都存储在内存当中，当服务重启之后任务就不存在了，只能接收新任务。

如果是在大型异步项目中，希望异步任务可以断开任务后继续执行，任务重复执行，任务可以查看执行状态，任务可以定时执行等操作，推荐使用异步任务框架 `Celery`。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`