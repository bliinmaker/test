# Витя говорит нужно прописать импорт всех файлов с ручками  в файл __init__ это в api/book/__init__
from . import book - у тебя хто есть в твоем проекте если что

1. Проверьте содержимое `mem_router`:
```python
# app/api/mem/router.py
from fastapi import APIRouter

mem_router = APIRouter()

# Должны быть определены эндпоинты, например:
@mem_router.get("/items")
async def read_items():
    return {"message": "Hello World"}
```

2. Убедитесь, что все маршруты правильно импортированы и определены. В вашем случае проверьте:
   - Правильность пути импорта `from app.api.mem.router import mem_router`
   - Наличие декораторов маршрутов (@mem_router.get(), @mem_router.post() и т.д.) в файле router.py

3. Добавьте префикс и теги при подключении роутера:
```python
def setup_routers(app: FastAPI) -> None:
    app.add_route('/metrics', metrics)
    app.include_router(
        mem_router,
        prefix="/api/v1",  # опционально
        tags=["mem"]       # опционально
    )
```

4. Проверьте, что `/metrics` endpoint корректно определен:
```python
# app/metrics.py
from prometheus_client import make_asgi_app

metrics = make_asgi_app()
```

5. Для отладки можно временно добавить тестовый эндпоинт прямо в основной файл:
```python
def create_app() -> FastAPI:
    app = FastAPI(docs_url='/swagger', lifespan=lifespan)
    
    @app.get("/test")
    async def test_endpoint():
        return {"message": "Test endpoint"}
        
    setup_middleware(app)
    setup_routers(app)
    return app
```


Если после этих проверок ошибка всё ещё возникает, покажите содержимое файла `app/api/mem/router.py` и убедитесь, что все импорты работают корректно.

# 2 решение
Давайте рассмотрим другие возможные причины, если базовая структура правильная:

1. Проверьте точку входа (main.py):
```python
# main.py
from app.app import create_app

app = create_app()

# Убедитесь, что используете именно эту переменную app
```

2. Проверьте структуру проекта:
```
project/
├── app/
│   ├── __init__.py   # должен существовать
│   ├── app.py
│   ├── api/
│   │   ├── __init__.py   # должен существовать
│   │   └── mem/
│   │       ├── __init__.py   # должен существовать
│   │       └── router.py
```

3. Убедитесь, что все `__init__.py` существуют и пустые файлы присутствуют в каждой директории.

4. Проверьте запуск сервера:
```bash
# Убедитесь, что запускаете правильно
uvicorn main:app --reload
```

5. Посмотрите логи запуска:
```python
def create_app() -> FastAPI:
    app = FastAPI(docs_url='/swagger', lifespan=lifespan)
    
    # Добавьте отладочный вывод
    print("Setting up middleware...")
    setup_middleware(app)
    print("Setting up routers...")
    setup_routers(app)
    print("App created successfully")
    
    return app
```

6. Проверьте декораторы в роутере:
```python
from fastapi import APIRouter

mem_router = APIRouter()

# Попробуйте добавить response_model
from pydantic import BaseModel

class ResponseModel(BaseModel):
    message: str

@mem_router.get("/test", response_model=ResponseModel)
async def test_endpoint():
    return {"message": "Test"}
```

7. Временно уберите middleware и проверьте работу:
```python
def create_app() -> FastAPI:
    app = FastAPI(docs_url='/swagger', lifespan=lifespan)
    # Закомментируйте middleware
    # setup_middleware(app)
    setup_routers(app)
    return app
```

8. Проверьте доступность Swagger UI:
```python
def create_app() -> FastAPI:
    # Попробуйте изменить путь к документации
    app = FastAPI(
        docs_url="/docs",  # вместо /swagger
        redoc_url="/redoc",
        openapi_url="/openapi.json"
    )
    setup_middleware(app)
    setup_routers(app)
    return app
```

9. Проверьте версию FastAPI:
```bash
pip show fastapi
```

10. Попробуйте очистить кеш Python:
```bash
find . -type d -name __pycache__ -exec rm -r {} +
```

Если после всех этих проверок проблема сохраняется:
1. Создайте минимальный рабочий пример в одном файле
2. Проверьте работу через curl или Postman
3. Просмотрите сетевые запросы в браузере (Network tab)
4. Убедитесь, что порт не занят другим процессом

Также можно включить дополнительное логирование:
```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger("uvicorn")
```

Если ничего не помогает, возможно стоит:
1. Создать новое виртуальное окружение
2. Переустановить все зависимости
3. Обновить FastAPI и его зависимости до последних версий
