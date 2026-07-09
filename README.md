# 🍲 Recipe API

Асинхронный CRUD-сервис рецептов на FastAPI с тестами на pytest и CI на GitHub Actions.

## Что это

REST API для хранения рецептов: создание, список с сортировкой по популярности и времени готовки, детальная карточка с ингредиентами. Каждый рецепт хранит счётчик просмотров, который увеличивается при открытии карточки.

## Стек

- **FastAPI** — веб-фреймворк, роуты в `main.py`
- **SQLAlchemy 2.0 (async ORM)** — модели `Recipe` / `Ingredient`, отношение один-ко-многим с `cascade="all, delete-orphan"`
- **aiosqlite** — асинхронный драйвер SQLite
- **Pydantic v2** — валидация входа/выхода (`RecipeCreate`, `RecipeListItem`, `RecipeDetail`)
- **pytest + pytest-asyncio** — тесты
- **ruff** — линтер
- **GitHub Actions** — CI, прогон тестов на каждый push в `main`

## Эндпоинты

| Метод | Путь | Описание |
|---|---|---|
| `GET` | `/recipes` | Список рецептов. Сортировка: просмотры ↓, затем время готовки ↑ |
| `GET` | `/recipes/{id}` | Детали рецепта (ингредиенты, описание). Каждый запрос увеличивает счётчик просмотров |
| `POST` | `/recipes` | Создать рецепт (название, время готовки, описание, список ингредиентов) |

Интерактивная документация — `/docs` (Swagger UI) после запуска сервиса.

### Пример создания рецепта

```json
POST /recipes
{
  "name": "Окрошка",
  "cook_time_minutes": 25,
  "description": "Нарезать. Смешать.",
  "ingredients": [
    { "name": "Огурец" },
    { "name": "Квас" },
    { "name": "Соль" }
  ]
}
```

## Запуск локально

```bash
git clone https://github.com/roivser/fastapi_ci.git
cd fastapi_ci

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
uvicorn main:app --reload
```

Сервис поднимется на `http://127.0.0.1:8000`, документация — на `http://127.0.0.1:8000/docs`.

База — файл SQLite (`recipes.db`), создаётся автоматически при старте.

## Тесты

```bash
pip install -r requirements-dev.txt
pytest
```

Тесты (`tests/test_api.py`) гоняются в изолированном режиме: переменная окружения `TESTING=1` отключает создание боевой SQLite-базы при старте приложения, чтобы тесты не зависели от файла на диске и друг от друга.

## CI

При каждом push в `main` GitHub Actions устанавливает зависимости и прогоняет `pytest` — workflow называется **CI** ([история запусков](https://github.com/roivser/fastapi_ci/actions)).

## Структура проекта

```
fastapi_ci/
├── main.py            # FastAPI-приложение, роуты
├── models.py           # SQLAlchemy-модели (Recipe, Ingredient)
├── schemas.py          # Pydantic-схемы
├── crud.py             # Операции с БД (создание, сортировка, просмотры)
├── database.py         # Подключение к БД, сессии, инициализация таблиц
├── tests/
│   └── test_api.py
├── requirements.txt
├── requirements-dev.txt
└── .github/workflows/  # CI-пайплайн
```

## Возможные доработки

- Пагинация для `GET /recipes`
- Обновление и удаление рецептов (`PATCH` / `DELETE`)
- Переход на PostgreSQL для production-конфигурации
