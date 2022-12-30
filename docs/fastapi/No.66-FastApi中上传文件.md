> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

#### 前言

上一章介绍了使用`form`格式提交`POST`请求数据，本章将介绍如何使用`fastapi`上传文件。

#### 安装python-multipart

因为上传文件是用的`form data`形式，所以使用前需要安装依赖：`python-multipart`

```bash
$ pip install python-multipart
```

#### 使用File类

1. 导出`File`
2. 在请求`Body`定义`File`

```python
from fastapi import FastAPI, File

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File()):
    return {"file_size": len(file)}
```

测试：
```bash
$  curl -i -X POST -d 'file=3333' http://127.0.0.1:8000/files/

HTTP/1.1 200 OK
date: Tue, 27 Dec 2022 03:01:44 GMT
server: uvicorn
content-length: 15
content-type: application/json

{"file_size":4}

# 或者
$ curl -i -F "file=@test.txt" http://127.0.0.1:8000/files/

HTTP/1.1 200 OK
date: Tue, 27 Dec 2022 04:06:07 GMT
server: uvicorn
content-length: 40
content-type: application/json

{"file_size":13,"file":"hello world!\n"}
```

总结：
`File()`类使用起来非常简单便捷，定义内容类型时为`bytes`，字节内容是存在系统内存中的，因此这种方法适用于上传文件比较小的情况。

#### 上传大文件使用UploadFile

```python
from fastapi import FastAPI, UploadFile

app = FastAPI()



@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename, "file": file.file}
```

测试：
```bash
$ curl -i -F "file=@test.txt" http://127.0.0.1:8000/uploadfile/

HTTP/1.1 200 OK
date: Tue, 27 Dec 2022 04:14:39 GMT
server: uvicorn
content-length: 200
content-type: application/json

{"filename":"test.txt","file":{"_file":{},"_max_size":1048576,"_rolled":false,"_TemporaryFileArgs":{"mode":"w+b","buffering":-1,"suffix":null,"prefix":null,"encoding":null,"newline":null,"dir":null}}}
```

总结：
1、使用`UploadFile`类不需要定义默认参数类型：`bytes`
2、`UploadFile`类有一个内存处理机制：小文件会存储在内存，当bytes大小超过系统设置的内存限制，会存放在本地磁盘
3、基于2，使用`UploadFile`就可以支持上传大文件，例如：音频、视频、表格、文档、压缩包等大文件
4、基于`UploadFile`的属性，可以对上传的文件灵活处理。

##### UploadFile类的属性

- filename: 文件名
- content_type: 文件类型
- file: `SpooledTemporaryFile` 临时文件类

例如：
```bash
$  curl -i -F "file=@/Users/hope/Downloads/brew-3.6.13.tar.gz" http://127.0.0.1:8000/uploadfile/

HTTP/1.1 100 Continue

HTTP/1.1 200 OK
date: Tue, 27 Dec 2022 08:27:42 GMT
server: uvicorn
content-length: 130
content-type: application/json

{"filename":"brew-3.6.13.tar.gz","file":{"_file":{},"_max_size":1048576,"_rolled":true},"content-type":"application/octet-stream"}
```

##### UploadFile类的方法

- write(data)：写文件到磁盘
- read(size)：读取文件，可以指定每次读取大小
- seek(offset)：指定当前指针位置：`await myfile.seek(0)` 可以将指针位置挪到文件开头

读取文件内容：
```python
contents = await myfile.read()
```

##### File 类型定义为 UploadFile

```python
@app.post("/uploadfile/")
async def create_upload_file(
    file: UploadFile = File(description="A file read as UploadFile"),
):
    return {"filename": file.filename}
```

##### 上传多个文件

```python
from typing import List

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()


@app.post("/files/")
async def create_files(files: List[bytes] = File()):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(files: List[UploadFile]):
    return {"filenames": [file.filename for file in files]}


@app.get("/")
async def main():
    content = """
<body>
<form action="/files/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
<form action="/uploadfiles/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
</body>
    """
    return HTMLResponse(content=content)

```

测试：
```bash
$ curl -i -F "files=@/Users/hope/Downloads/brew-3.6.13.tar.gz" -F "files=@/Users/hope/Downloads/WPS_Office_5.1.0(7657)_x64.dmg" http://127.0.0.1:8000/uploadfiles/

HTTP/1.1 100 Continue

HTTP/1.1 200 OK
date: Tue, 27 Dec 2022 08:41:49 GMT
server: uvicorn
content-length: 69
content-type: application/json

{"filenames":["brew-3.6.13.tar.gz","WPS_Office_5.1.0(7657)_x64.dmg"]}

```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
