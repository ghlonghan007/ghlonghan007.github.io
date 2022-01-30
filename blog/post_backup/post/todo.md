---
title: "tortoise学习笔记"
date: 2020-05-28T17:22:59+08:00
---

一个异步ORM 支持MySQL

使用分3步:

1.定义一个模型

```python
from tortoise.models import Model
from tortoise import fields

class Tournament(Model):
    id = fields.IntField(pk=True)
    name = fields.TextField()
```

2.初始化orm,将模型注册到上面

```python
from tortoise import Tortoise, run_async

async def init():
    await Tortoise.init(
        db_url='sqlite://db.sqlite3',
        modules={'models': ['app.models']}
    )
    # Generate the schema
    await Tortoise.generate_schemas()

# run_async is a helper function to run simple async Tortoise scripts.
run_async(init())
```

3.使用ORM接口 自带很多方便的接口

```python 
# Create instance by save
tournament = Tournament(name='New Tournament')
await tournament.save()

# Or by .create()
await Tournament.create(name='Another Tournament')

# Now search for a record
tour = await Tournament.filter(name__contains='Another').first()
print(tour.name)
```