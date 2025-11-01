# Спецификация технологического стека

## Обзор

Данный документ содержит детальную спецификацию технологического стека для системы парсинга статей с обоснованием выбора каждой технологии.

## 1. Основной язык программирования

### Python 3.11+

**Версия**: 3.11 или выше (рекомендуется 3.12)

**Обоснование**:
- ✅ **Богатая экосистема**: Множество библиотек для веб-скрапинга (Beautiful Soup, Scrapy, lxml)
- ✅ **Async/await**: Нативная поддержка асинхронного программирования для высокой производительности
- ✅ **Type hints**: Статическая типизация для повышения качества кода
- ✅ **Простота**: Быстрая разработка и низкий порог входа
- ✅ **Сообщество**: Большое активное сообщество, хорошая документация
- ✅ **Performance improvements**: Python 3.11+ имеет значительные улучшения производительности

**Альтернативы**:
- ❌ **Node.js**: Хорош для scraping, но менее зрелая экосистема для data processing
- ❌ **Go**: Высокая производительность, но меньше библиотек для HTML parsing
- ❌ **Rust**: Максимальная производительность, но высокий порог входа

**Решение**: Python 3.11+ - оптимальный баланс между производительностью и простотой разработки

## 2. Библиотеки для парсинга и краулинга

### 2.1 HTTP-клиент: httpx

**Версия**: 0.27+

**Обоснование**:
- ✅ **Async поддержка**: Нативная поддержка async/await
- ✅ **HTTP/2**: Поддержка современных протоколов
- ✅ **Совместимость**: API совместим с requests
- ✅ **Активная разработка**: Регулярные обновления и поддержка

**Пример использования**:
```python
import httpx

async def fetch_url(url: str) -> str:
    async with httpx.AsyncClient() as client:
        response = await client.get(
            url,
            headers={"User-Agent": "ArticleParser/1.0"},
            timeout=30.0
        )
        response.raise_for_status()
        return response.text
```

**Альтернативы**:
- **aiohttp**: Чисто async, но менее удобный API
- **requests**: Популярный, но только sync

**Решение**: httpx - современный, async, совместим с requests

### 2.2 HTML-парсер: Beautiful Soup 4 + lxml

**Beautiful Soup**: 4.12+
**lxml**: 5.1+

**Обоснование**:
- ✅ **BS4**: Простой и интуитивный API
- ✅ **lxml**: Самый быстрый парсер (написан на C)
- ✅ **Устойчивость**: Хорошо обрабатывает невалидный HTML
- ✅ **Гибкость**: CSS селекторы и XPath
- ✅ **Mature**: Проверенные временем библиотеки

**Пример использования**:
```python
from bs4 import BeautifulSoup

def parse_html(html: str) -> BeautifulSoup:
    return BeautifulSoup(html, 'lxml')

soup = parse_html(html)
title = soup.select_one('h1.article-title')
content = soup.select_one('div.article-body')
```

**Альтернативы**:
- **Scrapy**: Полноценный framework, избыточен для наших задач
- **pyquery**: jQuery-like синтаксис, менее популярен
- **selectolax**: Очень быстрый, но менее гибкий

**Решение**: Beautiful Soup 4 + lxml - баланс скорости и удобства

### 2.3 Дополнительные библиотеки для парсинга

**html2text**: 2024.2.26+
```python
import html2text

h = html2text.HTML2Text()
h.ignore_links = False
markdown = h.handle(html_content)
```

**bleach**: 6.1+
```python
import bleach

clean_text = bleach.clean(
    dirty_html,
    tags=['p', 'br', 'strong', 'em'],
    strip=True
)
```

**python-dateutil**: 2.8+
```python
from dateutil import parser

date = parser.parse("2025-11-01T10:00:00Z")
```

## 3. База данных

### PostgreSQL 15+

**Версия**: 15 или выше (рекомендуется 16)

**Обоснование**:
- ✅ **Надежность**: ACID транзакции, проверенная временем
- ✅ **Производительность**: Отличная производительность для OLTP
- ✅ **Полнотекстовый поиск**: Встроенная поддержка FTS для русского и английского
- ✅ **JSON поддержка**: JSONB для гибких метаданных
- ✅ **Расширения**: pg_trgm для fuzzy search, pgvector для embeddings (будущее)
- ✅ **Open Source**: Бесплатно, активное сообщество

**Конфигурация**:
```sql
-- Включить расширения
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Полнотекстовый поиск для русского
CREATE TEXT SEARCH CONFIGURATION russian_articles (COPY = russian);
```

**Альтернативы**:
- **MongoDB**: Гибкая схема, но меньше гарантий консистентности
- **SQLite**: Простой, но ограниченный для production
- **MySQL**: Популярный, но менее мощный FTS

**Решение**: PostgreSQL 15+ - оптимальный выбор для структурированных данных с FTS

## 4. ORM и миграции

### SQLAlchemy 2.0+

**Версия**: 2.0+

**Обоснование**:
- ✅ **Async поддержка**: SQLAlchemy 2.0 полностью поддерживает async
- ✅ **Мощный**: Один из самых функциональных ORM
- ✅ **Type hints**: Хорошая поддержка типизации
- ✅ **Гибкость**: От простых моделей до сложных запросов

**Пример использования**:
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import String, Text, DateTime
import uuid

class Base(DeclarativeBase):
    pass

class Article(Base):
    __tablename__ = "articles"

    id: Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
    url: Mapped[str] = mapped_column(String, unique=True, index=True)
    title: Mapped[str] = mapped_column(String)
    content: Mapped[str] = mapped_column(Text)
    published_at: Mapped[DateTime] = mapped_column(DateTime, nullable=True)

# Async usage
engine = create_async_engine("postgresql+asyncpg://...")

async def get_article(article_id: uuid.UUID):
    async with AsyncSession(engine) as session:
        return await session.get(Article, article_id)
```

**Альтернативы**:
- **Tortoise ORM**: Async-first, но менее зрелый
- **Peewee**: Простой, но менее мощный
- **Django ORM**: Отличный, но привязан к Django

**Решение**: SQLAlchemy 2.0+ - стандарт индустрии с async поддержкой

### Alembic 1.13+

**Обоснование**:
- ✅ **Интеграция**: Разработан создателями SQLAlchemy
- ✅ **Миграции**: Автоматическая генерация и применение
- ✅ **Версионирование**: Управление схемой БД

**Пример**:
```bash
# Создать миграцию
alembic revision --autogenerate -m "Add articles table"

# Применить миграции
alembic upgrade head
```

## 5. Кэширование и очереди задач

### Redis 7+

**Версия**: 7.0+

**Обоснование**:
- ✅ **Скорость**: In-memory хранилище, миллисекундная латентность
- ✅ **Универсальность**: Кэш, очередь, pub/sub, rate limiting
- ✅ **Надежность**: Поддержка persistence и репликации
- ✅ **Экосистема**: Множество клиентов и инструментов

**Использование**:
1. **Кэш**: HTTP ответы, robots.txt, конфигурации
2. **Очередь**: Broker для Celery
3. **Rate limiting**: Token bucket counters
4. **Pub/Sub**: Real-time уведомления

**Python клиент**: redis-py 5.0+ (async поддержка)

```python
import redis.asyncio as redis

r = await redis.from_url("redis://localhost")
await r.set("key", "value", ex=3600)  # TTL 1 hour
value = await r.get("key")
```

**Альтернативы**:
- **Memcached**: Только кэш, нет persistence
- **RabbitMQ**: Только очереди, тяжеловеснее

**Решение**: Redis 7+ - универсальное решение для кэша и очередей

## 6. Асинхронные задачи

### Celery 5.3+

**Версия**: 5.3+

**Обоснование**:
- ✅ **Зрелость**: Стандарт для async tasks в Python
- ✅ **Функциональность**: Периодические задачи, retry, routing
- ✅ **Мониторинг**: Flower для визуализации
- ✅ **Масштабируемость**: Легко добавлять воркеры

**Пример использования**:
```python
from celery import Celery

app = Celery('article_parser', broker='redis://localhost:6379/0')

@app.task(bind=True, max_retries=3)
def parse_article(self, url: str):
    try:
        # Parsing logic
        return article_data
    except Exception as exc:
        raise self.retry(exc=exc, countdown=60)

# Запуск задачи
result = parse_article.delay("https://example.com/article")
```

**Celery Beat** для периодических задач:
```python
from celery.schedules import crontab

app.conf.beat_schedule = {
    'parse-news-every-hour': {
        'task': 'tasks.parse_news_sources',
        'schedule': crontab(minute=0),  # Каждый час
    },
}
```

**Альтернативы**:
- **RQ (Redis Queue)**: Проще, но менее функционален
- **Dramatiq**: Современная альтернатива, но меньше экосистема
- **Huey**: Легковесный, но ограниченный

**Решение**: Celery 5.3+ - проверенное решение с богатыми возможностями

## 7. API Framework

### FastAPI 0.109+

**Версия**: 0.109+

**Обоснование**:
- ✅ **Производительность**: Один из самых быстрых Python фреймворков
- ✅ **Async**: Нативная поддержка async/await
- ✅ **Типизация**: Автоматическая валидация через Pydantic
- ✅ **Документация**: Автогенерация OpenAPI/Swagger документации
- ✅ **WebSocket**: Встроенная поддержка
- ✅ **Современность**: Активная разработка, следует best practices

**Пример API**:
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional
import uuid

app = FastAPI(title="Article Parser API", version="1.0.0")

class ArticleResponse(BaseModel):
    id: uuid.UUID
    url: str
    title: str
    content: str
    author: Optional[str] = None

@app.get("/articles/{article_id}", response_model=ArticleResponse)
async def get_article(article_id: uuid.UUID):
    article = await article_service.get_by_id(article_id)
    if not article:
        raise HTTPException(status_code=404, detail="Article not found")
    return article

@app.post("/parse", status_code=202)
async def parse_url(url: str):
    task = parse_article.delay(url)
    return {"task_id": task.id, "status": "accepted"}
```

**Автоматическая документация**: `/docs` (Swagger UI), `/redoc` (ReDoc)

**Альтернативы**:
- **Flask**: Проще, но менее функционален, нет async
- **Django REST Framework**: Мощный, но тяжелый, привязан к Django
- **Starlette**: Легковесный, FastAPI построен на нем

**Решение**: FastAPI 0.109+ - современный, быстрый, с отличным DX

## 8. CLI Framework

### Click 8.1+

**Версия**: 8.1+

**Обоснование**:
- ✅ **Простота**: Интуитивный API через декораторы
- ✅ **Композиция**: Группы команд, подкоманды
- ✅ **Автодокументация**: Автоматическая генерация --help
- ✅ **Валидация**: Встроенная валидация параметров
- ✅ **Широкое использование**: Используется в Flask, pip, и др.

**Пример CLI**:
```python
import click

@click.group()
def cli():
    """Article Parser CLI"""
    pass

@cli.command()
@click.option('--url', required=True, help='Article URL to parse')
@click.option('--format', default='json', type=click.Choice(['json', 'markdown']))
def parse(url: str, format: str):
    """Parse a single article"""
    click.echo(f"Parsing {url}...")
    result = parse_article(url)
    if format == 'json':
        click.echo(json.dumps(result, indent=2))
    else:
        click.echo(result.to_markdown())

@cli.command()
@click.option('--source', required=True, help='Source ID')
def run(source: str):
    """Start parsing for a source"""
    click.echo(f"Starting parser for source {source}")
    # Logic here

if __name__ == '__main__':
    cli()
```

**Использование**:
```bash
python -m article_parser parse --url https://example.com/article
python -m article_parser run --source ria-ru
```

**Альтернативы**:
- **Typer**: Современная альтернатива на основе Click с type hints
- **argparse**: Встроенный в Python, но менее удобный
- **fire**: Автоматическая CLI генерация, но менее контроля

**Решение**: Click 8.1+ - проверенное решение с отличным API

**Альтернатива для рассмотрения**: Typer (более современный, с type hints)

## 9. Валидация и конфигурация

### Pydantic 2.5+

**Версия**: 2.5+

**Обоснование**:
- ✅ **Производительность**: Pydantic v2 значительно быстрее (Rust core)
- ✅ **Валидация**: Мощная валидация данных через type hints
- ✅ **Serialization**: JSON serialization/deserialization
- ✅ **Settings**: Управление конфигурацией через environment variables

**Пример валидации**:
```python
from pydantic import BaseModel, HttpUrl, Field, field_validator
from typing import Optional
from datetime import datetime

class ArticleParseResult(BaseModel):
    url: HttpUrl
    title: str = Field(min_length=5, max_length=500)
    content: str = Field(min_length=100)
    author: Optional[str] = Field(None, max_length=100)
    published_at: Optional[datetime] = None

    @field_validator('title')
    @classmethod
    def title_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('Title cannot be empty')
        return v.strip()

# Использование
article = ArticleParseResult(
    url="https://example.com/article",
    title="Article Title",
    content="Long content here..."
)
```

**Конфигурация через Pydantic Settings**:
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    redis_url: str = "redis://localhost:6379/0"
    api_key: Optional[str] = None

    class Config:
        env_file = ".env"

settings = Settings()
```

**Альтернативы**:
- **Marshmallow**: Старая альтернатива, медленнее
- **attrs + cattrs**: Хороший, но менее популярен
- **dataclasses**: Встроенный, но нет валидации

**Решение**: Pydantic 2.5+ - стандарт для валидации в Python

### PyYAML 6.0+

**Для загрузки конфигураций источников**:

```python
import yaml
from pathlib import Path

def load_source_config(config_path: Path) -> dict:
    with open(config_path, 'r', encoding='utf-8') as f:
        return yaml.safe_load(f)
```

## 10. Тестирование

### pytest 7.4+

**Версия**: 7.4+

**Обоснование**:
- ✅ **Стандарт**: Де-факто стандарт для тестирования в Python
- ✅ **Fixtures**: Мощная система fixtures
- ✅ **Parametrize**: Легко создавать параметризованные тесты
- ✅ **Плагины**: Богатая экосистема плагинов
- ✅ **Async**: Поддержка async тестов через pytest-asyncio

**Плагины**:
- **pytest-asyncio**: Для async тестов
- **pytest-cov**: Coverage reporting
- **pytest-mock**: Удобные моки
- **pytest-httpx**: Моки для httpx

**Пример тестов**:
```python
import pytest
from article_parser.parser import ArticleParser

@pytest.fixture
def sample_html():
    return """
    <article>
        <h1>Test Article</h1>
        <div class="content">Article content here</div>
    </article>
    """

@pytest.mark.asyncio
async def test_parse_article(sample_html):
    parser = ArticleParser()
    result = await parser.parse(sample_html)

    assert result.title == "Test Article"
    assert "Article content" in result.content

@pytest.mark.parametrize("url,expected_domain", [
    ("https://example.com/article", "example.com"),
    ("https://blog.example.com/post", "blog.example.com"),
])
def test_extract_domain(url, expected_domain):
    domain = extract_domain(url)
    assert domain == expected_domain
```

**Альтернативы**:
- **unittest**: Встроенный, но менее удобный
- **nose2**: Устаревший
- **hypothesis**: Для property-based testing (как дополнение)

**Решение**: pytest 7.4+ - лучший выбор для Python

## 11. Качество кода и линтинг

### Ruff 0.1+

**Обоснование**:
- ✅ **Скорость**: Написан на Rust, в 10-100x быстрее других линтеров
- ✅ **Универсальность**: Заменяет Flake8, isort, pyupgrade и др.
- ✅ **Фиксы**: Автоматические исправления многих проблем

**Конфигурация** (`pyproject.toml`):
```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "C",   # flake8-comprehensions
    "B",   # flake8-bugbear
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

**Использование**:
```bash
ruff check .           # Проверка
ruff check --fix .     # Автоисправление
ruff format .          # Форматирование
```

### mypy 1.8+

**Статическая проверка типов**:

```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

```bash
mypy article_parser/
```

**Альтернативы**:
- **Black**: Форматтер, ruff format его заменяет
- **isort**: Сортировка импортов, ruff его заменяет
- **Flake8**: Линтер, ruff его заменяет
- **pyright**: Альтернатива mypy, от Microsoft

**Решение**: Ruff + mypy - современный стек для качества кода

## 12. Логирование и мониторинг

### structlog 24.1+

**Обоснование**:
- ✅ **Структурированность**: JSON логи для парсинга
- ✅ **Контекст**: Автоматический контекст (request_id, user_id)
- ✅ **Производительность**: Lazy evaluation
- ✅ **Интеграция**: Легко интегрируется с ELK, Loki

**Пример**:
```python
import structlog

logger = structlog.get_logger()

logger.info(
    "article_parsed",
    url="https://example.com/article",
    title="Article Title",
    parse_time_ms=150
)

# Output:
# {"event": "article_parsed", "url": "...", "title": "...", "parse_time_ms": 150, "timestamp": "..."}
```

### Prometheus client 0.19+

**Метрики**:
```python
from prometheus_client import Counter, Histogram, Gauge

# Счетчики
articles_parsed = Counter('articles_parsed_total', 'Total articles parsed')
parse_errors = Counter('parse_errors_total', 'Total parsing errors')

# Гистограммы (для latency)
parse_duration = Histogram('parse_duration_seconds', 'Article parsing duration')

# Gauge (текущие значения)
queue_size = Gauge('task_queue_size', 'Current task queue size')

# Использование
with parse_duration.time():
    result = parse_article(url)
    articles_parsed.inc()
```

**Альтернативы**:
- **python-json-logger**: Проще, но менее функционален
- **loguru**: Удобный, но менее структурированный

**Решение**: structlog + prometheus-client - профессиональный стек

## 13. Контейнеризация и оркестрация

### Docker 24+

**Dockerfile пример**:
```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Установка зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода
COPY . .

# Переменные окружения
ENV PYTHONUNBUFFERED=1
ENV WORKERS=4

# Запуск
CMD ["python", "-m", "article_parser.api"]
```

### Docker Compose

**docker-compose.yml**:
```yaml
version: '3.9'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/articles
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  worker:
    build: .
    command: celery -A article_parser.tasks worker -l info
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/articles
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  beat:
    build: .
    command: celery -A article_parser.tasks beat -l info
    environment:
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis

  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=articles
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## 14. CI/CD

### GitHub Actions

**.github/workflows/test.yml**:
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pip install -r requirements-dev.txt

    - name: Lint with ruff
      run: ruff check .

    - name: Type check with mypy
      run: mypy article_parser/

    - name: Test with pytest
      run: |
        pytest --cov=article_parser --cov-report=xml

    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## 15. Дополнительные утилиты

### Управление зависимостями: Poetry 1.7+

**Опционально, альтернатива pip + requirements.txt**

**pyproject.toml**:
```toml
[tool.poetry]
name = "article-parser"
version = "0.1.0"
description = "Web article parsing system"
authors = ["Your Name <your.email@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
httpx = "^0.27.0"
beautifulsoup4 = "^4.12.0"
lxml = "^5.1.0"
fastapi = "^0.109.0"
sqlalchemy = "^2.0.0"
alembic = "^1.13.0"
celery = "^5.3.0"
redis = "^5.0.0"
pydantic = "^2.5.0"
pydantic-settings = "^2.1.0"
click = "^8.1.0"
structlog = "^24.1.0"
prometheus-client = "^0.19.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
pytest-asyncio = "^0.23.0"
pytest-cov = "^4.1.0"
pytest-mock = "^3.12.0"
ruff = "^0.1.0"
mypy = "^1.8.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

### Pre-commit hooks: pre-commit 3.6+

**.pre-commit-config.yaml**:
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
```

## 16. Полный стек - Резюме

### Core Stack

| Категория | Технология | Версия | Назначение |
|-----------|------------|--------|------------|
| Язык | Python | 3.11+ | Основной язык |
| HTTP клиент | httpx | 0.27+ | Асинхронные HTTP запросы |
| HTML парсер | Beautiful Soup 4 | 4.12+ | Парсинг HTML |
| HTML парсер | lxml | 5.1+ | Быстрый XML/HTML парсер |
| База данных | PostgreSQL | 15+ | Хранение статей |
| ORM | SQLAlchemy | 2.0+ | Работа с БД |
| Миграции | Alembic | 1.13+ | Миграции схемы БД |
| Кэш/Очередь | Redis | 7+ | Кэширование и брокер |
| Задачи | Celery | 5.3+ | Асинхронные задачи |
| API | FastAPI | 0.109+ | REST API |
| CLI | Click | 8.1+ | Командная строка |

### Supporting Stack

| Категория | Технология | Версия | Назначение |
|-----------|------------|--------|------------|
| Валидация | Pydantic | 2.5+ | Валидация данных |
| Конфигурация | PyYAML | 6.0+ | YAML конфигурации |
| Тестирование | pytest | 7.4+ | Unit/Integration тесты |
| Линтинг | Ruff | 0.1+ | Код quality |
| Типы | mypy | 1.8+ | Статическая проверка типов |
| Логирование | structlog | 24.1+ | Структурированные логи |
| Метрики | prometheus-client | 0.19+ | Мониторинг метрик |
| Контейнеры | Docker | 24+ | Контейнеризация |
| Оркестрация | Docker Compose | 2.0+ | Локальная разработка |

### Infrastructure

| Категория | Технология | Назначение |
|-----------|------------|------------|
| CI/CD | GitHub Actions | Автоматизация тестов и деплоя |
| Мониторинг | Prometheus + Grafana | Метрики и дашборды |
| Логирование | Loki + Grafana | Агрегация и визуализация логов |
| Package Manager | Poetry (опционально) | Управление зависимостями |
| Pre-commit | pre-commit | Git hooks для качества кода |

## 17. Зависимости проекта

### requirements.txt

```txt
# Core
httpx==0.27.0
beautifulsoup4==4.12.3
lxml==5.1.0

# Database
sqlalchemy[asyncio]==2.0.25
asyncpg==0.29.0  # PostgreSQL async driver
alembic==1.13.1

# Tasks & Cache
celery==5.3.6
redis==5.0.1

# API
fastapi==0.109.0
uvicorn[standard]==0.27.0
pydantic==2.5.3
pydantic-settings==2.1.0

# CLI
click==8.1.7

# Utils
python-dateutil==2.8.2
PyYAML==6.0.1
html2text==2024.2.26
bleach==6.1.0

# Logging & Monitoring
structlog==24.1.0
prometheus-client==0.19.0

# Type stubs
types-PyYAML==6.0.12
types-redis==4.6.0
```

### requirements-dev.txt

```txt
-r requirements.txt

# Testing
pytest==7.4.4
pytest-asyncio==0.23.3
pytest-cov==4.1.0
pytest-mock==3.12.0
pytest-httpx==0.28.0

# Code Quality
ruff==0.1.14
mypy==1.8.0

# Pre-commit
pre-commit==3.6.0
```

## Заключение

Технологический стек построен на современных, проверенных технологиях Python экосистемы:

- **Асинхронность**: httpx, SQLAlchemy 2.0, FastAPI для высокой производительности
- **Типобезопасность**: Type hints, Pydantic, mypy для надежности
- **Качество кода**: Ruff, mypy, pytest для поддерживаемости
- **Наблюдаемость**: structlog, Prometheus для операционной поддержки
- **Простота развертывания**: Docker, Docker Compose для reproducibility

Этот стек обеспечивает баланс между производительностью, надежностью и удобством разработки.

---

**Версия**: 1.0
**Дата**: 2025-11-01
**Статус**: Approved
