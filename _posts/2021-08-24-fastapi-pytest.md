---
layout: single
classes: wide
title: "FastAPI 및 pytest를 이용한 API 서버 테스트 코드 작성하기"
last_modified_at: 2021-08-24
description: "FastAPI 및 pytest를 이용한 API 서버 테스트 코드를 작성하는 방법에 대해서 정리하였습니다."
categories:
  - etc
tags:
  - python, fastapi, pytest, pytest-asyncio, httpx, testing, unit test
---

이번에 FastAPI로 만들어진 API 서버의 테스트 코드를 작성하면서 pytest를 사용 해 보았는데요.
추후 필요할 때 다시 찾아볼 수 있도록 간단하게 정리하였습니다.

---

## Requirements
- Python 3.7+
- FastAPI
- uvicorn
- pytest
- pytest-asyncio
- HTTPX

위 패키지들은 Python만 설치되어 있으면 터미널에서 아래와 같이 pip로 간단하게 설치 가능합니다.

``` bash
% pip install fastapi
% pip install uvicorn
% pip install pytest
% pip install pytest-asyncio
% pip install httpx
```

---

## 모듈 구조

참고로 여기서는 테스트를 위해 아래와 같이 샘플 모듈을 작성하였습니다.

- src
  - api
    - main.py: API 구현 코드
  - tests
    - test_async_main.py: 비동기 테스트 코드
    - test_main.py: 일반 테스트 코드

---

## 일반 테스트

### API 코드 작성

먼저 FastAPI로 총 세가지 API 함수를 작성하였습니다.
- root(): 루트로 요청할 경우 간단하게 응답해주는 GET 방식의 API 함수입니다.
- read_item(): “/items/{item_id}“로 요청할 경우 item_id에 해당하는 item 데이터를 응답해주는 GET 방식의 API 함수입니다.
- create_item(): “/items/“로 요청할 경우 입력받은 json형식의 item 데이터를 fake_db 변수에 추가하는 POST 방식의 API 함수입니다.

``` python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

fake_db = {
    "foo": {"id": "foo", "title": "Foo", "description": "There goes my hero"},
    "bar": {"id": "bar", "title": "Bar", "description": "The bartenders"},
}


class Item(BaseModel):
    id: str
    title: str
    description: Optional[str] = None


@app.get("/")
async def root():
    return {"msg": "Hello World"}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return fake_db.get(item_id, None)


@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    if item.id in fake_db:
        raise HTTPException(status_code=400, detail="Item already exists")

    fake_db[item.id] = item
    return item
```

### 테스트 코드 작성

아래와 같이 FastAPI에서 제공하는 TestClient 객체를 사용하면 위에서 작성한 API 코드를 client 변수를 통해 요청하고 응답 코드 및 json 결과값을 테스트할 수 있습니다.

``` python
from fastapi.testclient import TestClient

from src.api.main import app

client = TestClient(app)

이제 세가지 API 함수에 대해 테스트하는 함수들을 작성하겠습니다.

test_root(): “root“ API 함수에 대해 GET 방식으로 요청하여 테스트하는 함수 입니다.

test_read_item(): “read_item“ API 함수에 대해 item_id가 1인 item을 params 값에 추가하여 GET 방식으로 요청하여 테스트하는 함수 입니다.

test_create_item(): “create_item“ API 함수에 대해 생성할 item을 json 값에 추가하여 POST 방식으로 요청하여 테스트하는 함수 입니다.

def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}


def test_read_item():
    response = client.get("/items/foo", params={"item_id": "1"})
    assert response.status_code == 200
    assert response.json() == {
        "id": "foo",
        "title": "Foo",
        "description": "There goes my hero",
    }


def test_create_item():
    response = client.post(
        "/items/",
        json={"id": "foobar", "title": "Foo Bar", "description": "The Foo Barters"},
    )
    assert response.status_code == 200
    assert response.json() == {
        "id": "foobar",
        "title": "Foo Bar",
        "description": "The Foo Barters",
    }
```

### 테스트 실행

간단하게 터미널에서 pytest로 테스트를 실행하고 테스트 결과를 확인해볼 수 있습니다.

(참고로 -v 옵션은 테스트 결과를 자세하게 확인하기 위해서 추가하였습니다.)

테스트 실행 결과 세가지 API 함수에 대해서 테스트가 통과된 것을 확인할 수 있습니다.

![pytest result](https://github.com/sehoi/sehoi.github.io/blob/master/assets/images/2021-08-24_fastapi_pytest_1.png?raw=true)

---

## 비동기 테스트

API 코드는 기존과 동일합니다.


### 테스트 코드 작성

비동기 테스트는 위에서 사용한 TestClient에서는 지원하지 않기 때문에 아래와 같이 HTTPX 패키지의 AsyncClient를 사용합니다.

``` python
import json
import pytest

from httpx import AsyncClient
```

비동기 테스트를 위해서 아래와 같이 함수명 위에 @pytest.mark.asyncio를 명시해주면 pytest가 해당 테스트를 비동기로 처리해주게 됩니다.

그런 다음 AsyncClient와 await를 이용하여 비동기 요청을 보내고 응답을 받을 수 있습니다.

``` python
@pytest.mark.asyncio
async def test_root():
    async with AsyncClient(base_url="http://127.0.0.1:8000") as ac:
        response = await ac.get("/")
        assert response.status_code == 200
        assert response.json() == {"msg": "Hello World"}


@pytest.mark.asyncio
async def test_read_item():
    async with AsyncClient(base_url="http://127.0.0.1:8000") as ac:
        response = await ac.get("/items/foo", params={"item_id": "1"})
        assert response.status_code == 200
        assert response.json() == {
            "id": "foo",
            "title": "Foo",
            "description": "There goes my hero",
        }


@pytest.mark.asyncio
async def test_create_item():
    async with AsyncClient(base_url="http://127.0.0.1:8000") as ac:
        response = await ac.post(
            "/items/",
            content=json.dumps(
                {
                    "id": "foobar",
                    "title": "Foo Bar",
                    "description": "The Foo Barters",
                }
            ),
        )
        assert response.status_code == 200
        assert response.json() == {
            "id": "foobar",
            "title": "Foo Bar",
            "description": "The Foo Barters",
        }
```

### 테스트 실행

먼저 아래와 같이 uvicorn으로 위에서 구현한 API 요청을 받을 서버를 실행합니다.

![fastapi server run](https://github.com/sehoi/sehoi.github.io/blob/master/assets/images/2021-08-24_fastapi_pytest_2.png?raw=true)

그리고 pytest -v로 테스트를 실행해주면 아래와 같이 새로 작성한 테스트와 기존 테스트 모두 성공한 것을 확인 할 수 있습니다.

![async pytest result](https://github.com/sehoi/sehoi.github.io/blob/master/assets/images/2021-08-24_fastapi_pytest_3.png?raw=true)

---

## 참고 자료

### FastAPI 관련
- https://fastapi.tiangolo.com/tutorial/testing/
- https://fastapi.tiangolo.com/advanced/async-tests/

### pytest
- https://docs.pytest.org/en/6.2.x/
- https://pypi.org/project/pytest-asyncio/

### 그 외
- https://www.python-httpx.org/
