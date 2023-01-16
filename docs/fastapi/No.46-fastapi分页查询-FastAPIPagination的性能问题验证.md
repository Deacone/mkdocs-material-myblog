>温馨提示: 读完本文大约需要 3 分钟；  
这是一篇技术类文章；  
需要对`fastapi`有一定的了解；  
代码部分横屏观看更佳。


#### 前言

最近在使用 `FastApi` 为后端框架进行一个项目的开发，在 `FastApi` 使用过程中，遇到了分页查询的问题，特此记录一下，以便以后查阅。

继续上一篇分页查询的文章，在上一篇文章的最后，提到使用 `Pageination` 进行分页查询，但是我在 `crud` 方法中使用了 `all()` 方法进行全量查询，这样做会不会有性能影响呢？

#### 问题分析

首先性能问题是否会有影响项目本身的运行，经过测试，50万以内的数据是基本不会收到影响的，50万的数据查询延迟在3秒以内。这个也是在可接受范围内的。

其次需要评估下项目本身的数据量级，如果公司内部使用，数据一年生产的量不超过 50万，可以不用考虑性能问题。

##### 源码剖析

1. 分析 `Page` 这个类：

```python
class Page(BasePage[T], Generic[T]):
    page: conint(ge=1)  # type: ignore
    size: conint(ge=1)  # type: ignore

    __params_type__ = Params

    @classmethod
    def create(
        cls,
        items: Sequence[T],
        total: int,
        params: AbstractParams,
    ) -> Page[T]:
        if not isinstance(params, Params):
            raise ValueError("Page should be used with Params")

        return cls(
            total=total,
            items=items,
            page=params.page,
            size=params.size,
        )
```

这个是表示分页结果的模型，对分页结果进行序列化，使得分页结果呈现如下数据格式：
```json
{
  "items": [
    ...
  ],
  "page": 0,
  "size": 50,
  "total": 100
}

```

2. 分析 `paginate` 这个函数：

```python
def paginate(
    sequence: Sequence[T],
    params: Optional[AbstractParams] = None,
    length_function: Callable[[Sequence[T]], int] = len,
) -> AbstractPage[T]:
    params = resolve_params(params)
    raw_params = params.to_raw_params()

    return create_page(
        items=sequence[raw_params.offset : raw_params.offset + raw_params.limit],
        total=length_function(sequence),
        params=params,
    )
```

- `sequence`: 这个是序列化结果参数，即从数据库查询出来的结果集
- `params`: 这个是接口请求时候传递的参数，默认 `page` = 0 `size` = 50
- `length_function`: 这个参数是对结果集的长度进行计算的参数，默认使用 `len()` 这个函数进行计算

所以没有说查询的事情，都是对查询结果进行处理

3. 鉴于以上的问题，又对 `LimitOffsetPage` 进行了剖析：

```python
class LimitOffsetPage(BasePage[T], Generic[T]):
    limit: conint(ge=1)  # type: ignore
    offset: conint(ge=0)  # type: ignore

    __params_type__ = LimitOffsetParams

    @classmethod
    def create(
        cls,
        items: Sequence[T],
        total: int,
        params: AbstractParams,
    ) -> LimitOffsetPage[T]:
        return cls(
            total=total,
            items=items,
            **asdict(params.to_raw_params()),
        )
```

这个类也是对结果集进行的处理，这样写了之后，返回 `json` 序列就变成了：
```json
{
  "items": [
    ...
  ],
  "total": 0,
  "offset": 0,
  "limit": 50
}
```

#### 总结

以上三个类和方法均没有我想要的结果。

建议：
如果对性能要求比较高，使用过程中还是需要从 `crud` 函数下功夫，对查询过程进行优化，例如加索引等措施，就不能直接使用这样进行查询了：

```python
db.query(Case).all()
```

---

每日踩一坑，生活更轻松。


本期分享就到这里啦。祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`