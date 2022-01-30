---
title: "Fastapi min代码示例"
date: 2021-03-04T22:49:49+08:00

---



### Fastapi jwt验正

```python
import jwt

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.hash import bcrypt
from tortoise import fields 
from tortoise.contrib.fastapi import register_tortoise
from tortoise.contrib.pydantic import pydantic_model_creator
from tortoise.models import Model 

app = FastAPI()

JWT_SECRET = 'myjwtsecret'

class User(Model):
    id = fields.IntField(pk=True)
    username = fields.CharField(50, unique=True)
    password_hash = fields.CharField(128)

    def verify_password(self, password):
        return bcrypt.verify(password, self.password_hash)

User_Pydantic = pydantic_model_creator(User, name='User')
UserIn_Pydantic = pydantic_model_creator(User, name='UserIn', exclude_readonly=True)

oauth2_scheme = OAuth2PasswordBearer(tokenUrl='token')

async def authenticate_user(username: str, password: str):
    user = await User.get(username=username)
    if not user:
        return False 
    if not user.verify_password(password):
        return False
    return user 

@app.post('/token')
async def generate_token(form_data: OAuth2PasswordRequestForm = Depends()):
    user = await authenticate_user(form_data.username, form_data.password)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, 
            detail='Invalid username or password'
        )

    user_obj = await User_Pydantic.from_tortoise_orm(user)

    token = jwt.encode(user_obj.dict(), JWT_SECRET)

    return {'access_token' : token, 'token_type' : 'bearer'}

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=['HS256'])
        user = await User.get(id=payload.get('id'))
    except:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, 
            detail='Invalid username or password'
        )

    return await User_Pydantic.from_tortoise_orm(user)


@app.post('/users', response_model=User_Pydantic)
async def create_user(user: UserIn_Pydantic):
    user_obj = User(username=user.username, password_hash=bcrypt.hash(user.password_hash))
    await user_obj.save()
    return await User_Pydantic.from_tortoise_orm(user_obj)

@app.get('/users/me', response_model=User_Pydantic)
async def get_user(user: User_Pydantic = Depends(get_current_user)):
    return user    

register_tortoise(
    app, 
    db_url='sqlite://db.sqlite3',
    modules={'models': ['main']},
    generate_schemas=True,
    add_exception_handlers=True
)
```

### tortoise 库的基本使用

```Python
from fastapi import FastAPI 
from pydantic import BaseModel

from tortoise import fields
from tortoise.models import Model
from tortoise.contrib.fastapi import register_tortoise
from tortoise.contrib.pydantic import pydantic_model_creator

import requests

app = FastAPI()

class City(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(50, unique=True)
    timezone = fields.CharField(50)

    def current_time(self) -> str:
        r = requests.get(f'http://worldtimeapi.org/api/timezone/{self.timezone}')
        current_time = r.json()['datetime']
        return current_time

    class PydanticMeta:
        computed = ('current_time', )

City_Pydantic = pydantic_model_creator(City, name='City')
CityIn_Pydantic = pydantic_model_creator(City, name='CityIn', exclude_readonly=True)

@app.get('/')
def index():
    return {'key' : 'value'}

@app.get('/cities')
async def get_cities():
    return await City_Pydantic.from_queryset(City.all())

@app.get('/cities/{city_id}')
async def get_city(city_id: int):
    return await City_Pydantic.from_queryset_single(City.get(id=city_id))

@app.post('/cities')
async def create_city(city: CityIn_Pydantic):
    city_obj = await City.create(**city.dict(exclude_unset=True))
    return await City_Pydantic.from_tortoise_orm(city_obj)

@app.delete('/cities/{city_id}')
async def delete_city(city_id: int):
    await City.filter(id=city_id).delete()
    return {}

register_tortoise(
    app,
    db_url='sqlite://db.sqlite3',
    modules={'models': ['main']},
    generate_schemas=True,
    add_exception_handlers=True
)
```

### aiohttp异步请求外部接口

```Python
from fastapi import FastAPI 
from pydantic import BaseModel

from tortoise import fields
from tortoise.models import Model
from tortoise.contrib.fastapi import register_tortoise
from tortoise.contrib.pydantic import pydantic_model_creator

import requests
import aiohttp
import asyncio

app = FastAPI()

session = None

@app.on_event('startup')
async def startup_event():
    global session 
    session = aiohttp.ClientSession()

@app.on_event('shutdown')
async def shutdown_event():
    await session.close()

class City(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(50, unique=True)
    timezone = fields.CharField(50)

    def current_time(self) -> str:
        return ''

    @classmethod
    async def get_current_time(cls, obj, session):
        async with session.get(f'http://worldtimeapi.org/api/timezone/{obj.timezone}') as response:
            result = await response.json()
            current_time = result['datetime']
            obj.current_time = current_time

    class PydanticMeta:
        computed = ('current_time', )

City_Pydantic = pydantic_model_creator(City, name='City')
CityIn_Pydantic = pydantic_model_creator(City, name='CityIn', exclude_readonly=True)

@app.get('/')
def index():
    return {'key' : 'value'}

@app.get('/cities')
async def get_cities():
    cities = await City_Pydantic.from_queryset(City.all())

    global session

    tasks = []
    for city in cities:
        task = asyncio.create_task(City.get_current_time(city, session))
        tasks.append(task)

    await asyncio.gather(*tasks)
    return cities

@app.get('/cities/{city_id}')
async def get_city(city_id: int):
    return await City_Pydantic.from_queryset_single(City.get(id=city_id))

@app.post('/cities')
async def create_city(city: CityIn_Pydantic):
    city_obj = await City.create(**city.dict(exclude_unset=True))
    return await City_Pydantic.from_tortoise_orm(city_obj)

@app.delete('/cities/{city_id}')
async def delete_city(city_id: int):
    await City.filter(id=city_id).delete()
    return {}

register_tortoise(
    app,
    db_url='sqlite://db.sqlite3',
    modules={'models': ['main']},
    generate_schemas=True,
    add_exception_handlers=True
)
```

