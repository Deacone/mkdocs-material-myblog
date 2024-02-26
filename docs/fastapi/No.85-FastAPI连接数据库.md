## 导读

使用过`Django`、`Flask`的同学，应该对`ORM`不陌生。同样，`FastAPI`也是采用`ORM`的形式来连接数据库，操作数据库。

`ORM`的全称是：`object-relational mapping`。这种模式可以让我们操作数据库不用写复杂的`SQL`，只需要写 `python` 类对象即可，`ORM` 会将类对象按照一定的规则映射到数据库表中。

Django框架使用的是`Django-ORM`，`Flask`框架一般使用的是`SQLAlchemy`，也可以是`Peewee`。

下面将介绍在`FastAPI`中怎么使用`SQLAlchemy`。

### 安装SQLAlchemy

使用 `pip` 工具进行安装：
```bash
$  pip install sqlalchemy
```

### 创建一个项目

项目目录结构如下：
```
.
sql_app
├── __init__.py
├── database.py
├── main.py
└── user
    ├── __init__.py
    ├── crud.py
    ├── models.py
    └── schemas.py

```

###  跟数据建立连接

因为`MySQL`使用的比较多，这里以`MySQL`为例。

需要安装依赖`pymysql`: 
```bash
pip install pymysql
```

创建数据库：
```sql
CREATE DATABASE IF NOT EXISTS sql_app default charset utf8mb4;
```

在文件`sql_app/database.py`中编写如下代码：

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# SQLite 专用配置
# SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
# engine = create_engine(
#     SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
# )


# MySQL专用配置
SQLALCHEMY_DATABASE_URL = "mysql+pymysql://root:yuiedhj456312@localhost:3307/sql_app?charset=utf8"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL
)

# postgresql 专用配置
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"
# engine = create_engine(
#     SQLALCHEMY_DATABASE_URL
# )

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

```

### 创建数据模型

在文件`sql_app/user/models.py`创建数据模型：

```python
from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

from sql_app.database import Base


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")

```

创建两张表：`User` `Item`

一对多，一个用户有多个item, 一个item只能有一个user。

### 将数据模型映射到数据库

这里使用 `alembic` 来管理模型迁移。

#### 初始化项目

1. 进入项目根目录
2. 激活虚拟环境
3. 初始化项目

```bash
$ cd /path/to/yourproject
$ source /path/to/yourproject/.venv/bin/activate   # assuming a local virtualenv
$ alembic init alembic
```

此时项目目录变成这样：

```bash
$ tree
.
├── __init__.py
├── alembic
│   ├── README
│   ├── env.py
│   ├── script.py.mako
│   └── versions
├── alembic.ini
├── database.py
├── main.py
└── user
    ├── __init__.py
    ├── crud.py
    ├── models.py
    └── schemas.py

```

#### 编辑配置

`alembic.ini`: 将 sqlalchemy.url 的值配置成你的MySQL连接。

```ini
sqlalchemy.url = mysql+pymysql://root:xiaoe123@localhost:3307/sql_app?charset=utf8
```

#### 生成迁移文件

```bash
 $ alembic revision -m "create user item tables"

```

此时多了个文件：`sql_app/alembic/versions/b23d49a6e41e_create_user_item_tables.py`

```bash
"""create user item tables

Revision ID: b23d49a6e41e
Revises: 
Create Date: 2023-11-06 10:14:03.940261

"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision: str = 'b23d49a6e41e'
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    pass


def downgrade() -> None:
    pass

```

#### 执行迁移

```bash
$ alembic upgrade head
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> b23d49a6e41e, create user item tables
```

查看数据库：
```sql
show tables;

alembic_version
```

```sql
desc alembic_version

version_num,varchar(32),NO,PRI,,""
```

此时看到并没有生成 模型 User Item 的表。  
因为 生成的 `upgrade` 函数为空，并没有自动化生成对应的迁移内容，需要手动自己填入内容，并不是想要的。

## 结语

本章介绍了如何连接数据库，创建数据模型，生成迁移脚本。  

下一章将介绍如何自动化生成迁移脚本。

***

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝君在测开之路上越走越顺，越走越远。

gzh：`测开工程师的烦恼`