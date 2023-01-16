>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。

#### 前言

最近在使用 `FastApi` 为后端框架进行一个项目的开发，在 `FastApi` 使用过程中，在写返回数据列表的时候，不知道怎么处理，特此记录一下，以便以后查阅。

#### 编写列表接口时候整体流程

1.  编写 `schema` , 以便对返回的数据进行序列化处理
2.  编写 `crud` , 进行业务处理，跟数据库进行交互
3.  编写入口函数，定义 路由 `url` 以及 `入参`

##### 1. 编写 `schema`

```python
from pydantic import BaseModel

class CaseSchemaDetail(BaseModel):
    id: int
    name: str
    case_level: str
    marks: str = None
    remark: str = None
    module_id: int

    create_time: datetime
    update_time: datetime

    class Config:
        orm_mode = True
```

这里先导出 `pydantic` 的类 `BaseModel`，再继承 `BaseModel`。

书写格式举例子：`marks: str = None`

*   marks: 需要展示的字段的名字，跟对应的 `model` 的名字一致
*   str：字段的类型
*   None：字段的默认值

##### 2. 编写 `crud`

```python
def get_cases(db: Session):
    """
    查询用例列表

    :param db:
    :return:
    """
    return db.query(Case).all()
```

这里比较简单，直接进行查询就可以啦

##### 3. 编写入口函数

```python
@app.get("/cases", response_model=Page[case_schema.CaseSchemaDetail])
async def read_cases(db: SessionLocal = Depends(get_db)):
    """
    查询用例列表

    :param db: db session
    :return:
    """
    return paginate(case_crud.get_cases(db))
```

在 `response_model` 这里使用了 `Page` 对返回结果进行处理，返回结果是这样的：

```json
{
  "items": [
    {
      "id": 0,
      "name": "string",
      "case_level": "string",
      "marks": "string",
      "remark": "string",
      "module_id": 0,
      "create_time": "2022-08-21T02:42:29.209Z",
      "update_time": "2022-08-21T02:42:29.209Z"
    }
  ],
  "total": 0,
  "page": 1,
  "size": 1
}
```

##### 后记：返回字段中，其中格式是 list 的字段怎么处理，即多层嵌套

在 `schema` 类里面进行处理就好啦：

```python
from typing import List

class CaseSchemaDetail(BaseModel):
    id: int
    name: str
    case_level: str
    marks: str = None
    remark: str = None
    module_id: int
    modules: List[ModuleSchemaDetail] = []

    create_time: datetime
    update_time: datetime

    class Config:
        orm_mode = True
```

这里新添加了一个字段 `modules` , 这个字段的类型声明为 `List`, `list` 里面的内容为：`ModuleSchemaDetail`, 默认为 \[]

返回结果是这样的：

```json
{
  "items": [
    {
      "id": 0,
      "name": "string",
      "case_level": "string",
      "marks": "string",
      "remark": "string",
      "module_id": 0,
      "modules": [
        {
          "id": 0,
          "name": "string",
          "remark": "string",
          "create_time": "2022-08-21T02:46:37.003Z",
          "update_time": "2022-08-21T02:46:37.003Z"
        }
      ],
      "create_time": "2022-08-21T02:46:37.003Z",
      "update_time": "2022-08-21T02:46:37.003Z"
    }
  ],
  "total": 0,
  "page": 1,
  "size": 1
}
```

***

每日踩一坑，生活更轻松。

本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`
