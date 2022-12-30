> 温馨提示:
> 读完本文大约需要 3 分钟；
> 这是一篇技术类文章；
> 需要对`fastapi`有一定的了解；
> 代码部分横屏观看更佳。

#### 前言

前面介绍了如何自定义返回字段，今天这篇文章将介绍如何定义返回的响应码

#### 响应码的定义适用于哪些请求方法

`restfulapi` 规范内定义的方法基本都适用，例如：

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`

等其他请求方法。

#### 举个简单的例子

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}

```

非常简单的例子，在请求入口函数直接定义`status_code=201`即可。


#### 还可以使用status

`fastapi`帮我们定义好了内置很多响应码，使用`status`即可。
```python
from fastapi import FastAPI, status

app = FastAPI()


@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}

```

可以使用内置的状态码全在这里啦，足够平时开发使用：
```python
__all__ = (
    "HTTP_100_CONTINUE",
    "HTTP_101_SWITCHING_PROTOCOLS",
    "HTTP_102_PROCESSING",
    "HTTP_103_EARLY_HINTS",
    "HTTP_200_OK",
    "HTTP_201_CREATED",
    "HTTP_202_ACCEPTED",
    "HTTP_203_NON_AUTHORITATIVE_INFORMATION",
    "HTTP_204_NO_CONTENT",
    "HTTP_205_RESET_CONTENT",
    "HTTP_206_PARTIAL_CONTENT",
    "HTTP_207_MULTI_STATUS",
    "HTTP_208_ALREADY_REPORTED",
    "HTTP_226_IM_USED",
    "HTTP_300_MULTIPLE_CHOICES",
    "HTTP_301_MOVED_PERMANENTLY",
    "HTTP_302_FOUND",
    "HTTP_303_SEE_OTHER",
    "HTTP_304_NOT_MODIFIED",
    "HTTP_305_USE_PROXY",
    "HTTP_306_RESERVED",
    "HTTP_307_TEMPORARY_REDIRECT",
    "HTTP_308_PERMANENT_REDIRECT",
    "HTTP_400_BAD_REQUEST",
    "HTTP_401_UNAUTHORIZED",
    "HTTP_402_PAYMENT_REQUIRED",
    "HTTP_403_FORBIDDEN",
    "HTTP_404_NOT_FOUND",
    "HTTP_405_METHOD_NOT_ALLOWED",
    "HTTP_406_NOT_ACCEPTABLE",
    "HTTP_407_PROXY_AUTHENTICATION_REQUIRED",
    "HTTP_408_REQUEST_TIMEOUT",
    "HTTP_409_CONFLICT",
    "HTTP_410_GONE",
    "HTTP_411_LENGTH_REQUIRED",
    "HTTP_412_PRECONDITION_FAILED",
    "HTTP_413_REQUEST_ENTITY_TOO_LARGE",
    "HTTP_414_REQUEST_URI_TOO_LONG",
    "HTTP_415_UNSUPPORTED_MEDIA_TYPE",
    "HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE",
    "HTTP_417_EXPECTATION_FAILED",
    "HTTP_418_IM_A_TEAPOT",
    "HTTP_421_MISDIRECTED_REQUEST",
    "HTTP_422_UNPROCESSABLE_ENTITY",
    "HTTP_423_LOCKED",
    "HTTP_424_FAILED_DEPENDENCY",
    "HTTP_425_TOO_EARLY",
    "HTTP_426_UPGRADE_REQUIRED",
    "HTTP_428_PRECONDITION_REQUIRED",
    "HTTP_429_TOO_MANY_REQUESTS",
    "HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE",
    "HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS",
    "HTTP_500_INTERNAL_SERVER_ERROR",
    "HTTP_501_NOT_IMPLEMENTED",
    "HTTP_502_BAD_GATEWAY",
    "HTTP_503_SERVICE_UNAVAILABLE",
    "HTTP_504_GATEWAY_TIMEOUT",
    "HTTP_505_HTTP_VERSION_NOT_SUPPORTED",
    "HTTP_506_VARIANT_ALSO_NEGOTIATES",
    "HTTP_507_INSUFFICIENT_STORAGE",
    "HTTP_508_LOOP_DETECTED",
    "HTTP_510_NOT_EXTENDED",
    "HTTP_511_NETWORK_AUTHENTICATION_REQUIRED",
    "WS_1000_NORMAL_CLOSURE",
    "WS_1001_GOING_AWAY",
    "WS_1002_PROTOCOL_ERROR",
    "WS_1003_UNSUPPORTED_DATA",
    "WS_1005_NO_STATUS_RCVD",
    "WS_1006_ABNORMAL_CLOSURE",
    "WS_1007_INVALID_FRAME_PAYLOAD_DATA",
    "WS_1008_POLICY_VIOLATION",
    "WS_1009_MESSAGE_TOO_BIG",
    "WS_1010_MANDATORY_EXT",
    "WS_1011_INTERNAL_ERROR",
    "WS_1012_SERVICE_RESTART",
    "WS_1013_TRY_AGAIN_LATER",
    "WS_1014_BAD_GATEWAY",
    "WS_1015_TLS_HANDSHAKE",
)
```

#### 附：响应码的含义大全详解

1xx: 信息

| 响应码 | 响应描述 |
| --- | --- |
| 100 继续 | 服务器仅接收到部分请求，但是一旦服务器并没有拒绝该请求，客户端应该继续发送其余的请求。 |
| 101 切换协议 | 服务器转换协议：请求者已要求服务器切换协议，服务器已确认并准备切换。 |
| 103 早期提示 | 	此状态代码主要用于与link链接头一起使用，以允许用户代理在服务器仍在准备响应时开始预加载资源 |

2xx: 成功

| 响应码 | 响应描述 |
| --- | --- |
| 200 成功 | 请求成功（这是对HTTP请求成功的标准应答。） |
| 201 已创建 | 请求被创建完成，同时新的资源被创建。 |
|202 已接受|供处理的请求已被接受，但是处理未完成。|
|203 非授权信息|请求已经被成功处理，但是一些应答头可能不正确，因为使用的是其他文档的拷贝。|
|204 无内容|请求已经被成功处理，但是没有返回新文档。浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而Servlet可以确定用户文档足够新，这个状态代码是很有用的。|
|205 重置内容|请求已经被成功处理，但是没有返回新文档。但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容。|
|206 部分内容|	客户发送了一个带有Range头的GET请求，服务器完成了它。|

3xx: 重定向

| 响应码 | 响应描述 |
| --- | --- |
|300 多种选择|	多重选择。链接列表。用户可以选择某链接到达目的地。最多允许五个地址。|
|301 永久移动|	所请求的页面已经转移至新的 URL 
|302 临时移动|	所请求的页面已经临时转移至新的 URL 。
|303 查看其他位置|	所请求的页面可在别的 URL 下被找到。
|304 未修改	|未按预期修改文档。客户端有缓冲的文档并发出了一个条件性的请求（一般是提供If-Modified-Since头表示客户只想比指定日期更新的文档）。服务器告诉客户，原来缓冲的文档还可以继续使用。
|305 使用代理	|客户请求的文档应该通过Location头所指明的代理服务器提取。
|306|目前已不再使用，但是代码依然被保留。
|307 临时重定向|	被请求的页面已经临时移至新的 URL 。
|308 永久重定向|	308 的定义实际上和 301 是一致的，唯一的区别在于，308 状态码不允许浏览器将原本为 POST 的请求重定向到 GET 请求上。

4xx: 客户端错误
| 响应码 | 响应描述 |
| --- | --- |
|400 错误请求|	因为语法错误，服务器未能理解请求。
|401 未授权|	合法请求，但对被请求页面的访问被禁止。因为被请求的页面需要身份验证，客户端没有|提供或者身份验证失败。
|402|	保留供将来使用
|403 禁止|	合法请求，但对被请求页面的访问被禁止。
|404 未找到|	服务器无法找到被请求的页面。
|405 方法禁用|	请求中指定的方法不被允许。
|406 不接受|	服务器生成的响应无法被客户端所接受。
|407 需要代理授权|	用户必须首先使用代理服务器进行验证，这样请求才会被处理。
|408 请求超时|	请求超出了服务器的等待时间。
|409 冲突|	由于冲突，请求无法被完成。
|410 已删除|	请求的资源已永久删除，被请求的页面不可用。
|411 需要有效长度|	“Content-Length” 未被定义。如果无此内容，服务器不会接受请求。
|412 未满足前提条件|	请求中的前提条件被服务器评估为失败。
|413 请求实体过大|	由于所请求的实体太大，服务器不会接受请求。
|414 请求的 URI 过长|	由于 URL 太长，服务器不会接受请求。当 POST |请求被转换为带有很长的查询信息的 GET 请求时，就会发生这种情况。
|415 不支持的媒体类型|	由于媒介类型不被支持，服务器不会接受请求。
|416 请求范围不符合要求|	客户端请求部分文档，但是服务器不能提供被请求的部分。
|417 未满足期望值|	服务器不能满足客户在请求中指定的请求头。

5xx: 服务器错误
| 响应码 | 响应描述 |
| --- | --- |
|500 服务器内部错误|	请求未完成。服务器遇到不可预知的情况。
|501 尚未实施|	请求未完成。服务器不支持所请求的功能，或者服务器无法完成请求。
|502 错误网关|	请求未完成。服务器充当网关或者代理的角色时，从上游服务器收到一个无效的响应|。
|503 服务不可用|	服务器当前不可用（过载或者宕机）。
|504 网关超时|	网关超时。服务器充当网关或者代理的角色时，未能从上游服务器收到一个及时的响|应。
|505 HTTP 版本不受支持|	服务器不支持请求中指明的HTTP协议版本。
|511 身份验证|	用户需要提供身份验证来获取网络访问入口。|

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
