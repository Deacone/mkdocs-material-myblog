> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

#### 前言

上一章介绍了使用文件上传，本章将介绍文件上传和Form搭配使用

#### 安装python-multipart

因为上传文件是用的`form data`形式，所以使用前需要安装依赖：`python-multipart`

```bash
$ pip install python-multipart
```

#### 使用

```python
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()


@app.post("/files-new/")
async def create_file(
        file: bytes = File(),
        fileb: UploadFile = File(),
        token: str = Form()
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

##### 测试1：使用错误的参数类型

```bash
$ curl -i -d "token=secret&file=999&fileb=b888" http://127.0.0.1:8000/files-new/

HTTP/1.1 422 Unprocessable Entity
date: Wed, 28 Dec 2022 07:31:21 GMT
server: uvicorn
content-length: 111
content-type: application/json

{"detail":[{"loc":["body","fileb"],"msg":"Expected UploadFile, received: <class 'str'>","type":"value_error"}]}
```
说明：UploadFile 类型错误

```bash
$ curl -i -F "file=@/Users/hope/Downloads/brew-3.6.13.tar.gz" -F "fileb=@/Users/hope/Downloads/WPS_Office_5.1.0(7657)_x64.dmg" -d "token=secret" http://127.0.0.1:8000/files-new/

Warning: You can only select one HTTP request method! You asked for both POST 
Warning: (-d, --data) and multipart formpost (-F, --form).
```
说明：http的限制，不能同时使用 `--data`参数类型和`--form`参数类型

##### 测试2：使用正常的参数类型
```bash
$ curl -i -F "file=@/Users/hope/Downloads/brew-3.6.13.tar.gz" -F "fileb=@/Users/hope/Downloads/WPS_Office_5.1.0(7657)_x64.dmg" --form "token=secret" http://127.0.0.1:8000/files-new/

HTTP/1.1 100 Continue

HTTP/1.1 200 OK
date: Wed, 28 Dec 2022 07:36:34 GMT
server: uvicorn
content-length: 86
content-type: application/json

{"file_size":2980915,"token":"secret","fileb_content_type":"application/octet-stream"}
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
