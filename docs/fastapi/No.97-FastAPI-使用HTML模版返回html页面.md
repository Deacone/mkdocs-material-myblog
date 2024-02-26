## 导读

FastAPI不仅提供了接口形式的返回 `JSON` 格式，还可以返回`html`格式页面。  
本章将介绍如何返回`html`页面。

使用过 `Flask` 的同学应该比较熟悉 `Jinja2` 模板，所以`FastAPI` 也使用了 `Jinja2`来渲染`html`页面。

## 安装jinja2

安装：
```bash
pip install jinja2
```

## 使用 Jinja2Templates

定义模板对象：

```python
templates = Jinja2Templates(directory="templates")

```
指定`html`模板文件的目录

## 返回html

返回html：
```python
@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse(
        name="item.html", context={"request": request, "id": id}
    )
```

## 整体测试代码

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")


@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse(
        name="item.html", context={"request": request, "id": id}
    )

```

测试模板`item.html`：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Test</title>
</head>
<body>
<h1>测试模版{{id}}</h1>
</body>
</html>
```

## 测试

```bash
curl -X GET --location "http://127.0.0.1:9999/items/666"
```

返回：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Test</title>
</head>
<body>
<h1>测试模版666</h1>
</body>
</html>
```


## 结语

以上代码都是在 python3.9 版本进行测试的。  
`templates.TemplateResponse` 的不同版本使用方式可能不同，如果出现使用不一样，可以直接跳转到源码看看怎么使用，源码也是`python`封装的。


***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`