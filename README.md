# Ошибка "no operations defined in spec" в FastAPI обычно возникает, когда в вашем API не определены никакие эндпоинты (операции) или они определены некорректно. Давайте разберем возможные причины и решения:

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
