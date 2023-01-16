# fastapi日志配置

>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。

## 遇到的问题

最近在搞压测时候，需要用到mock，就想着怎么快速把它搭建起来，而且性能还不能太差。


## 解决方案

### 为什么选择它
找了很多资料，最终锁定了`fastapi`这款web框架。解决了我的问题。

`fastapi`的官网地址：`https://fastapi.tiangolo.com/`

在`fastapi`官网的首页写到：

>`fastapi`是一款
>高性能的、
>学习成本低的、
>快速编程的、
>为生产环境准备的框架

并且在高性能这个特性上，它是跟NodeJs框架和Go框架来进行对比的。所以我在高性能这个方面就没有太多的担心啦。

### 代码编写体验

本身我的需求需要编写4个api来提供服务，逻辑也非常简单。

1. 通过路由不同的接口路径
2. 根据不同的接口路径，直接返回对应的响应信息模板

所以服务没有跟其他的组件(例如mysql、redis等)进行交互，性能相对比较好。

### 快速开始一个应用服务

使用`pycahrm `创建一个`fastapi`项目，编写几行代码即可。

`main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

运行服务, 自动加载，这个模式在开发环境下非常有用：

```bash
$ uvicorn main:app --reload

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [28720]
INFO:     Started server process [28722]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

测试，pycharm自动化创建了一个文件：`test_main.http`

```bash
# Test your FastAPI endpoints

GET http://127.0.0.1:8000/
Accept: application/json

###
```

直接点击运行按钮就可以了

或者终端命令行模式直接运行

```bash
curl -i http://127.0.0.1:8000/
```

### 编写mock服务代码

```python
@app.get('/sns/userinfo')
async def sns_userinfo():
    """
    mock https://api.weixin.qq.com/sns/userinfo
    """
    # logger.info(f'[请求/sns/userinfo] {request.path}')
    openid = uuid.uuid1().hex
    json_response = {
        "openid": openid,
        "nickname": "我是昵称3333",
        "sex": 1,
        "province": "海南省",
        "city": "三亚市",
        "country": "中国",
        "headimgurl": "https://thirdwx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46",
        "privilege": ["PRIVILEGE1", "PRIVILEGE2", "PRIVILEGE3"],
        "unionid": "%s" % (uuid.uuid1())
    }
    return json_response
```

这个接口是模拟的微信的api，是不是非常简单。

按理说这样直接部署到生产环境就能提供服务了。

接下来就开始了部署环节，也是非常简单：

```bash
uvicorn main:app --workers 16 --port 9999 --host 172.16.0.186
```

- workers: 运行的线程数
- port：提供服务的端口号
- host：提供服务的地址(本机地址)，这个比较坑，不能填写 `127.0.0.1`，那样的话，别的服务器来访问就访问不通。

到此为止部署环节就结束了。

可是存在几个问题：

- 运行的进程不是后台运行
- 运行报错信息不太好看，没有带上时间戳

### 进行改造

#### 日志

日志方面，加上日志模块：

```python
import logging

# create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(logging.INFO)

# create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# add formatter to ch
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)


# 使用：
logger.info(f'[请求/sns/userinfo] {app_id}')
```

这里只添加了控制台的日志。

fastapi 的请求日志改造，添加配置文件(支持多种格式`*.ini` `*.yml` `*.json`)即可：

`log_config.json`

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "default": {
      "()": "uvicorn.logging.DefaultFormatter",
      "fmt": "%(levelprefix)s %(message)s",
      "use_colors": null
    },
    "access": {
      "()": "uvicorn.logging.AccessFormatter",
      "fmt": "%(asctime)s - %(levelprefix)s %(client_addr)s - \"%(request_line)s\" %(status_code)s"
    }
  },
  "handlers": {
    "default": {
      "formatter": "default",
      "class": "logging.StreamHandler",
      "stream": "ext://sys.stderr"
    },
    "access": {
      "formatter": "access",
      "class": "logging.StreamHandler",
      "stream": "ext://sys.stdout"
    }
  },
  "loggers": {
    "uvicorn": {
      "handlers": [
        "default"
      ],
      "level": "INFO"
    },
    "uvicorn.error": {
      "level": "INFO"
    },
    "uvicorn.access": {
      "handlers": [
        "access"
      ],
      "level": "INFO",
      "propagate": false
    }
  }
}
```

这个配置文件可以直接`copy`来用，在启动程序的时候加载配置文件即可

#### 后台运行命令改造

让进程在后台运行，并且将控制台日志写入到文件：

```
nohup uvicorn main:app --workers 16 --port 9999 --host 172.16.0.186 --log-config log_config.json >>mock_server_runtime.log 2>&1 &
```

这样就把日志都写入了文件`mock_server_runtime.log`中。运行进程也在后台运行了。

日志写文件的方式有很多，使用python原生的logger模块写文件当然最好了，这里不再展开叙述。


## 总结

从调研使用框架，到编写代码，再到部署完成用了一天的时间。如果需求不那么大，fastapi是一个非常快速完成需求的框架。

最后祝大家早日实现工具提效，早点下班。