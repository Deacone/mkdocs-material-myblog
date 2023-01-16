>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。

#### 前言

最近在使用FastApi为后端框架进行一个项目的开发，在`FastApi`使用过程中，遇到了分页查询的问题，特此记录一下，以便以后查阅。

继续上一篇分页查询的文章，在上篇文章中，有读者跟我说分页查询有更好的方案，于是便查资料，将我查到的结果跟大家分享下。

本篇文章主要讲解 `fastapi-pagination` 这个 `fastapi` 的扩展库

#### fastapi-pagination初体验

按照官方文档先来个基础的使用方法：
```python
from fastapi_pagination import Page, add_pagination, paginate

@app.get("/cases", response_model=Page[case_schema.CaseSchemaDetail])
async def read_cases(db: SessionLocal = Depends(get_db)):
    """
    查询用例列表

    :param db: db session
    :return:
    """
    return paginate(case_crud.get_cases(db))
    
add_pagination(app)
```

##### 1.导入类和方法

在使用的时候，首先要导入类 `Page`，方法：`paginate` 、`add_pagination`

##### 2.在response_model处添加 `Page`

```
@app.get("/cases", response_model=Page[case_schema.CaseSchemaDetail])
```

##### 3.在返回数据的地方加上 `paginate`

```python
return paginate(case_crud.get_cases(db))
```

##### 4. 最后不要忘记了加上 `add_pagination`

```python
add_pagination(app)
```

这里注意：`add_pagination` 的引用需要在 `Page` 之后，我在试用的时候，放在了初始化app的后面：

```python
app = FastAPI()
add_pagination(app)
```

结果就程序报错了，放在初始化 `app` 这个文件的最后就OK了

#### 入参示例

```bash
curl -X 'GET' \
  'http://127.0.0.1:9000/cases?page=1&size=2' \
  -H 'accept: application/json'
```

在url地址后面加上两个查询参数即可：

- page
- size

#### 返回示例

```json
{
  "items": [
    {
      "id": 1,
      "name": "叫主产什其",
      "case_level": "10",
      "marks": null,
      "remark": null,
      "module_id": 2,
      "create_time": "2022-08-05T16:05:58",
      "update_time": "2022-08-05T16:05:58"
    },
    {
      "id": 2,
      "name": "称农活上",
      "case_level": "7",
      "marks": null,
      "remark": null,
      "module_id": 2,
      "create_time": "2022-08-05T16:19:19",
      "update_time": "2022-08-05T16:19:19"
    }
  ],
  "total": 13,
  "page": 1,
  "size": 2
}
```

在返回参数中主要添加了几个参数：

- `total`： 统计总共查询出来结果的个数
- `page`：页码
- `size`：每页展示数据条数
- `items`：查询数据结果对象

#### 后记

这个类库做分页查询还是非常方便的。

存在一点疑问：我在写 `crud` 方法进行查询的时候是全部都查询出来，会不会有性能问题。

```python
def get_cases(db: Session):
    """
    查询用例列表

    :param db:
    :return:
    """
    return db.query(Case).all()
```

针对这个问题，我将继续查询资料，在下期文章中跟大家分享。

---

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。