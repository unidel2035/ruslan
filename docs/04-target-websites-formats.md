# Целевые веб-сайты и форматы данных

## Обзор

Данный документ определяет целевые веб-сайты для парсинга и поддерживаемые форматы данных в системе парсинга статей.

## 1. Категории целевых веб-сайтов

### 1.1 Новостные порталы (категория A - высокий приоритет)

Крупные новостные сайты с регулярно обновляемым контентом.

#### Российские новостные сайты

| Сайт | URL | Особенности | Структура | Приоритет |
|------|-----|-------------|-----------|-----------|
| РИА Новости | ria.ru | Schema.org, Open Graph, стандартная разметка | `<article>`, JSON-LD | Высокий |
| ТАСС | tass.ru | Государственное агентство, структурированная разметка | `<article>`, Microdata | Высокий |
| Lenta.ru | lenta.ru | Популярный портал, активное обновление | `<div class="topic-body">` | Высокий |
| Meduza | meduza.io | Независимое СМИ, современная верстка | `<article>`, JSON-LD | Высокий |
| Kommersant | kommersant.ru | Деловые новости, платный контент | `<article>`, paywall | Средний |
| Vedomosti | vedomosti.ru | Деловая пресса | `<article>` | Средний |
| Известия | iz.ru | Общие новости | HTML5 семантика | Средний |

#### Международные новостные сайты (опционально)

| Сайт | URL | Особенности | Язык | Приоритет |
|------|-----|-------------|------|-----------|
| BBC News | bbc.com/news | Международные новости | EN | Низкий |
| Reuters | reuters.com | Агентство новостей | EN | Низкий |
| The Guardian | theguardian.com | Британская газета | EN | Низкий |

### 1.2 Блог-платформы (категория B - средний приоритет)

Платформы для публикации статей с унифицированной структурой.

| Платформа | URL | Особенности | Структура | Приоритет |
|-----------|-----|-------------|-----------|-----------|
| Habr | habr.com | IT-статьи, русскоязычная аудитория | `<article>`, специфичные классы | Высокий |
| VC.ru | vc.ru | Медиа о бизнесе и технологиях | `<div class="content">` | Высокий |
| Dzen (Яндекс.Дзен) | dzen.ru | Персональные блоги | Динамический контент, React | Средний |
| Medium | medium.com | Международная платформа | `<article>`, динамический контент | Средний |
| Dev.to | dev.to | IT-блоги | `<article>`, Markdown-based | Низкий |

### 1.3 Персональные блоги (категория C - низкий приоритет)

Индивидуальные блоги на различных движках.

| Движок | Примеры | Особенности | Определение | Приоритет |
|--------|---------|-------------|-------------|-----------|
| WordPress | *.wordpress.com, множество сайтов | Самый популярный CMS | `<article>`, WP-специфичные классы | Средний |
| Ghost | *.ghost.io | Современная платформа для блогов | `<article>`, JSON-LD | Средний |
| Jekyll | GitHub Pages блоги | Статические сайты | Простой HTML, `<article>` | Низкий |
| Hugo | Различные | Быстрый static site generator | Простой HTML | Низкий |
| Blogger | *.blogspot.com | Google платформа | Blogger-специфичная разметка | Низкий |

### 1.4 Научные и технические публикации (категория D - специализированные)

| Источник | URL | Особенности | Формат | Приоритет |
|----------|-----|-------------|--------|-----------|
| arXiv | arxiv.org | Препринты научных статей | HTML + LaTeX/PDF | Низкий |
| Habr (статьи) | habr.com/ru/articles/ | Технические статьи с кодом | HTML, подсветка кода | Высокий |
| Medium (tech) | medium.com/tag/programming | Технические блоги | HTML, код-блоки | Средний |

## 2. Анализ структуры сайтов

### 2.1 Общие паттерны разметки

#### HTML5 Семантические теги

Большинство современных сайтов используют семантические теги:

```html
<article>
  <header>
    <h1>Заголовок статьи</h1>
    <time datetime="2025-11-01">1 ноября 2025</time>
    <address>
      <a rel="author">Автор</a>
    </address>
  </header>

  <div class="article-body">
    <!-- Контент статьи -->
  </div>

  <footer>
    <!-- Теги, категории, share buttons -->
  </footer>
</article>
```

**Поддержка**: Приоритет 1 - искать `<article>`, `<header>`, `<time>`, `<address>`

#### Open Graph Protocol

Почти все новостные сайты используют Open Graph:

```html
<meta property="og:title" content="Заголовок статьи" />
<meta property="og:description" content="Описание статьи" />
<meta property="og:image" content="https://example.com/image.jpg" />
<meta property="og:url" content="https://example.com/article" />
<meta property="og:type" content="article" />
<meta property="article:published_time" content="2025-11-01T10:00:00Z" />
<meta property="article:author" content="Имя автора" />
```

**Поддержка**: Приоритет 1 - всегда проверять OG теги первыми

#### Twitter Cards

Дополнительная метаинформация:

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Заголовок" />
<meta name="twitter:description" content="Описание" />
<meta name="twitter:image" content="image.jpg" />
```

**Поддержка**: Приоритет 2 - fallback если нет OG

#### Schema.org (JSON-LD)

Структурированные данные:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Заголовок статьи",
  "datePublished": "2025-11-01T10:00:00Z",
  "dateModified": "2025-11-01T12:00:00Z",
  "author": {
    "@type": "Person",
    "name": "Имя автора"
  },
  "image": "https://example.com/image.jpg",
  "articleBody": "Полный текст статьи..."
}
</script>
```

**Поддержка**: Приоритет 1 - парсить JSON-LD как источник истины

### 2.2 Примеры конфигураций для конкретных сайтов

#### Пример 1: РИА Новости (ria.ru)

```yaml
source:
  name: "РИА Новости"
  domain: "ria.ru"
  base_url: "https://ria.ru"
  language: "ru"

parsing_rules:
  # Приоритет 1: JSON-LD
  metadata:
    - type: "json-ld"
      schema: "NewsArticle"

  # Приоритет 2: Open Graph
  fallback:
    - type: "open-graph"

  # Приоритет 3: Селекторы
  selectors:
    title:
      - css: "h1.article__title"
      - css: "h1[itemprop='headline']"

    content:
      - css: "div.article__body"
      - xpath: "//div[@class='article__text']"

    author:
      - css: "a.article__author-name"
      - meta: "article:author"

    published_date:
      - css: "time[datetime]"
        attribute: "datetime"
      - meta: "article:published_time"

    images:
      - css: "div.article__body img"
        attribute: "src"
      - css: "img[itemprop='image']"
        attribute: "src"

    tags:
      - css: "a.article__tag"
        extract: "text"

rate_limiting:
  requests_per_minute: 10
  delay_between_requests: 6  # seconds

validation:
  required_fields:
    - title
    - content
  min_content_length: 100
```

#### Пример 2: Habr (habr.com)

```yaml
source:
  name: "Habr"
  domain: "habr.com"
  base_url: "https://habr.com"
  language: "ru"

parsing_rules:
  metadata:
    - type: "open-graph"

  selectors:
    title:
      - css: "h1.tm-title"
      - css: "h1.post__title"

    content:
      - css: "div.tm-article-body"
      - css: "div.post__text"

    author:
      - css: "a.tm-user-info__username"
      - css: "span.user-info__nickname"

    published_date:
      - css: "time[datetime]"
        attribute: "datetime"

    images:
      - css: "div.tm-article-body img"
        attribute: "data-src"  # Habr uses lazy loading
      - css: "div.tm-article-body img"
        attribute: "src"

    tags:
      - css: "a.tm-tags-list__link"
        extract: "text"

    # Специфично для Habr - рейтинг статьи
    rating:
      - css: "span.tm-votes-meter__value"
        extract: "text"

    # Количество просмотров
    views:
      - css: "span.tm-icon-counter__value"
        extract: "text"

content_processing:
  # Habr имеет код-блоки - нужно сохранить форматирование
  preserve_code_blocks: true
  code_block_selector: "pre code"

  # Удалить элементы
  remove_selectors:
    - "div.tm-article-presenter__footer"
    - "div.tm-article-reading-time"

rate_limiting:
  requests_per_minute: 20
  delay_between_requests: 3
```

#### Пример 3: Medium (medium.com)

```yaml
source:
  name: "Medium"
  domain: "medium.com"
  base_url: "https://medium.com"
  language: "en"

parsing_rules:
  # Medium использует динамический контент, но также предоставляет метаданные
  metadata:
    - type: "open-graph"
    - type: "json-ld"

  selectors:
    title:
      - css: "h1[data-testid='storyTitle']"
      - css: "h1.graf--title"

    content:
      - css: "article section"
      - css: "div.section-content"

    author:
      - css: "a[rel='author']"
      - css: "a[data-action='show-user-card']"

    published_date:
      - css: "time[datetime]"
        attribute: "datetime"
      - meta: "article:published_time"

    images:
      - css: "article figure img"
        attribute: "src"

    tags:
      - css: "a[rel='tag']"
        extract: "text"

content_processing:
  # Medium использует <figure> для изображений с подписями
  preserve_figures: true

  remove_selectors:
    - "div.js-postActionsFooter"
    - "div.metabar"

rate_limiting:
  requests_per_minute: 15
  delay_between_requests: 4

special_handling:
  # Medium может требовать JavaScript для некоторых статей
  javascript_required: false
  # Medium имеет paywall для некоторых статей
  paywall_detection: true
  paywall_selector: "div[data-testid='paywall']"
```

## 3. Форматы входных данных

### 3.1 HTML Форматы

| Формат | Описание | Поддержка | Парсер |
|--------|----------|-----------|--------|
| HTML5 | Современный стандарт с семантическими тегами | Да | lxml + Beautiful Soup |
| HTML4/XHTML | Старые сайты | Да | lxml |
| Невалидный HTML | HTML с ошибками разметки | Да | lxml (режим recovery) |
| AMP HTML | Accelerated Mobile Pages | Опционально | Специальный обработчик |

### 3.2 Метаданные

| Формат | Описание | Приоритет | Пример |
|--------|----------|-----------|--------|
| Open Graph | Facebook Open Graph Protocol | Высокий | `<meta property="og:title">` |
| Twitter Cards | Twitter метаданные | Средний | `<meta name="twitter:title">` |
| JSON-LD | Schema.org в JSON формате | Высокий | `<script type="application/ld+json">` |
| Microdata | Встроенные schema.org данные | Средний | `<div itemscope itemtype="...">` |
| Dublin Core | Метаданные для документов | Низкий | `<meta name="DC.title">` |

### 3.3 Дополнительные форматы

| Формат | Описание | Использование |
|--------|----------|---------------|
| RSS/Atom | Фиды для обнаружения новых статей | Сканирование новых URL |
| Sitemap XML | Карта сайта | Обнаружение всех статей |
| robots.txt | Правила для роботов | Проверка разрешений |

## 4. Форматы выходных данных

### 4.1 Внутреннее хранение (База данных)

**PostgreSQL Schema**:

```sql
-- Таблица статей
CREATE TABLE articles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    url TEXT UNIQUE NOT NULL,
    url_hash VARCHAR(64) NOT NULL,  -- SHA-256 для дедупликации

    -- Основные поля
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    content_html TEXT,  -- Опционально, очищенный HTML
    summary TEXT,

    -- Метаданные
    author TEXT,
    published_at TIMESTAMPTZ,
    updated_at TIMESTAMPTZ,
    parsed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Дополнительные поля
    language VARCHAR(10),  -- ISO 639-1
    word_count INTEGER,
    reading_time INTEGER,  -- В минутах

    -- Связи
    source_id UUID REFERENCES sources(id),

    -- Гибкие метаданные (JSON)
    metadata JSONB DEFAULT '{}',

    -- Статус
    status VARCHAR(20) DEFAULT 'parsed',  -- parsed, validated, failed

    -- Timestamps
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    modified_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Индексы
    CONSTRAINT articles_url_hash_key UNIQUE (url_hash)
);

CREATE INDEX idx_articles_source_id ON articles(source_id);
CREATE INDEX idx_articles_published_at ON articles(published_at);
CREATE INDEX idx_articles_parsed_at ON articles(parsed_at);
CREATE INDEX idx_articles_status ON articles(status);
CREATE INDEX idx_articles_metadata ON articles USING GIN(metadata);

-- Полнотекстовый поиск
CREATE INDEX idx_articles_title_fts ON articles USING GIN(to_tsvector('russian', title));
CREATE INDEX idx_articles_content_fts ON articles USING GIN(to_tsvector('russian', content));

-- Таблица изображений
CREATE TABLE images (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    article_id UUID REFERENCES articles(id) ON DELETE CASCADE,

    url TEXT NOT NULL,
    caption TEXT,
    alt_text TEXT,

    is_main BOOLEAN DEFAULT FALSE,
    position INTEGER,  -- Порядковый номер в статье

    width INTEGER,
    height INTEGER,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_images_article_id ON images(article_id);

-- Таблица тегов
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT UNIQUE NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE article_tags (
    article_id UUID REFERENCES articles(id) ON DELETE CASCADE,
    tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (article_id, tag_id)
);

CREATE INDEX idx_article_tags_article ON article_tags(article_id);
CREATE INDEX idx_article_tags_tag ON article_tags(tag_id);
```

### 4.2 API форматы (JSON)

**GET /api/v1/articles/{id}** - Получение статьи:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "url": "https://example.com/article",
  "title": "Заголовок статьи",
  "content": "Полный текст статьи...",
  "summary": "Краткое описание статьи",
  "author": "Имя Автора",
  "published_at": "2025-11-01T10:00:00Z",
  "updated_at": null,
  "parsed_at": "2025-11-01T11:00:00Z",
  "language": "ru",
  "word_count": 1500,
  "reading_time": 8,
  "source": {
    "id": "660e8400-e29b-41d4-a716-446655440000",
    "name": "Example News",
    "domain": "example.com"
  },
  "images": [
    {
      "id": "770e8400-e29b-41d4-a716-446655440000",
      "url": "https://example.com/image.jpg",
      "caption": "Подпись к изображению",
      "is_main": true,
      "width": 1200,
      "height": 630
    }
  ],
  "tags": ["технологии", "новости", "python"],
  "metadata": {
    "views": 1000,
    "rating": 4.5,
    "comments_count": 25
  },
  "status": "parsed",
  "created_at": "2025-11-01T11:00:00Z"
}
```

**GET /api/v1/articles** - Список статей:

```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "url": "https://example.com/article",
      "title": "Заголовок",
      "summary": "Описание",
      "author": "Автор",
      "published_at": "2025-11-01T10:00:00Z",
      "source": {
        "name": "Example News"
      },
      "tags": ["технологии"]
    }
  ],
  "total": 100,
  "page": 1,
  "page_size": 20,
  "pages": 5
}
```

### 4.3 Экспорт форматов

#### JSON Export

Полный экспорт всех данных:

```json
{
  "export_date": "2025-11-01T12:00:00Z",
  "articles_count": 100,
  "articles": [
    {
      "url": "https://example.com/article",
      "title": "Заголовок",
      "content": "Контент",
      "metadata": { ... }
    }
  ]
}
```

#### CSV Export

Табличное представление:

```csv
id,url,title,author,published_at,word_count,source,tags
550e8400-...,https://example.com/article,"Title","Author",2025-11-01T10:00:00Z,1500,Example News,"tech,news"
```

#### Markdown Export

Текстовое представление статей:

```markdown
# Заголовок статьи

**Автор:** Имя Автора
**Дата:** 2025-11-01
**Источник:** [Example News](https://example.com)
**Теги:** технологии, новости

---

Полный текст статьи в формате Markdown...

![Изображение](https://example.com/image.jpg)
*Подпись к изображению*

---

**Метаданные:**
- URL: https://example.com/article
- Слов: 1500
- Время чтения: 8 минут
```

#### HTML Export

Очищенный HTML для отображения:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Заголовок статьи</title>
    <style>
        /* Минимальные стили */
        body { font-family: sans-serif; max-width: 800px; margin: 0 auto; }
        img { max-width: 100%; }
    </style>
</head>
<body>
    <article>
        <header>
            <h1>Заголовок статьи</h1>
            <p>
                <strong>Автор:</strong> Имя Автора<br>
                <strong>Дата:</strong> 2025-11-01
            </p>
        </header>
        <div class="content">
            <!-- Очищенный HTML контент -->
        </div>
        <footer>
            <p><strong>Источник:</strong> <a href="https://example.com">Example News</a></p>
            <p><strong>Теги:</strong> технологии, новости</p>
        </footer>
    </article>
</body>
</html>
```

## 5. Стратегия определения формата

### 5.1 Алгоритм определения источника

```python
def detect_source_type(url: str, html: str) -> SourceType:
    """
    Определяет тип источника на основе URL и HTML.
    """
    domain = extract_domain(url)

    # 1. Проверка известных доменов (whitelist)
    if domain in KNOWN_SOURCES:
        return KNOWN_SOURCES[domain]

    # 2. Определение по платформе (WordPress, Medium, etc.)
    platform = detect_platform(html)
    if platform:
        return platform

    # 3. Определение по структуре HTML
    structure = analyze_html_structure(html)
    if structure.has_json_ld:
        return SourceType.STRUCTURED
    elif structure.has_open_graph:
        return SourceType.SEMI_STRUCTURED
    else:
        return SourceType.UNSTRUCTURED
```

### 5.2 Приоритетная последовательность извлечения

```
1. JSON-LD (schema.org)
   └─> Если найдено и валидно → использовать как основной источник

2. Open Graph теги
   └─> Если JSON-LD нет или неполный → дополнить из OG

3. Twitter Cards
   └─> Если OG нет → использовать Twitter метаданные

4. HTML5 семантические теги
   └─> Если метаданных нет → искать <article>, <time>, <address>

5. Конфигурируемые селекторы
   └─> Если известен сайт → применить специфичные селекторы

6. Readability алгоритм
   └─> Если ничего не помогло → автоматическое определение контента

7. Эвристики
   └─> Последняя попытка → поиск по паттернам (даты, заголовки)
```

## 6. Валидация извлеченных данных

### 6.1 Обязательные поля

Для успешного парсинга требуются:
- **title**: Не пустой, длина 5-500 символов
- **content**: Не пустой, минимум 100 символов
- **url**: Валидный URL

### 6.2 Опциональные поля

Желательны, но не обязательны:
- **author**: Строка, 2-100 символов
- **published_at**: Валидная дата (не в будущем)
- **images**: Массив валидных URL
- **tags**: Массив строк

### 6.3 Правила валидации

```yaml
validation_rules:
  title:
    required: true
    min_length: 5
    max_length: 500
    not_empty: true

  content:
    required: true
    min_length: 100
    max_length: 1000000
    strip_html: true  # Проверять длину после удаления HTML

  url:
    required: true
    format: "url"
    schemes: ["http", "https"]

  author:
    required: false
    min_length: 2
    max_length: 100

  published_at:
    required: false
    format: "datetime"
    max_date: "now"  # Не в будущем
    min_date: "2000-01-01"  # Разумное прошлое

  images:
    required: false
    type: "array"
    items:
      format: "url"
      schemes: ["http", "https"]

  tags:
    required: false
    type: "array"
    max_items: 20
    items:
      min_length: 1
      max_length: 50
```

## 7. Обработка специальных случаев

### 7.1 Paywall (платный контент)

**Определение**:
- Наличие селекторов: `div.paywall`, `div.subscription-required`
- Проверка метаданных: `<meta name="access" content="subscription">`
- Анализ текста: обрыв контента с предложением подписки

**Обработка**:
- Пометить статью как `paywall: true` в метаданных
- Сохранить доступную часть контента
- Не помечать как ошибку парсинга

### 7.2 JavaScript-rendered контент

**Определение**:
- Пустой `<body>` с `<script>` тегами
- Наличие React/Vue/Angular корневых элементов
- Минимальный HTML с data-атрибутами для JS

**Обработка**:
- Пометить источник как `javascript_required: true`
- Опционально: использовать headless browser (Playwright/Selenium)
- Альтернатива: поиск API endpoints для получения данных

### 7.3 Многостраничные статьи

**Определение**:
- Наличие пагинации: "Страница 1 из 3"
- Ссылки "Следующая страница"

**Обработка**:
- Обнаружить все страницы
- Загрузить и объединить контент
- Сохранить как единую статью

### 7.4 Слайдшоу/Галереи

**Определение**:
- Структура с множественными `<figure>` элементами
- JavaScript галереи

**Обработка**:
- Извлечь все изображения
- Сохранить подписи к каждому
- Сохранить порядок

## Резюме

Документ определяет:

1. **Целевые сайты** по категориям с приоритетами
2. **Примеры конфигураций** для популярных источников
3. **Форматы входных данных** (HTML, метаданные)
4. **Форматы выходных данных** (БД, API, экспорт)
5. **Стратегию определения** формата и извлечения данных
6. **Правила валидации** извлеченных данных
7. **Обработку специальных случаев** (paywall, JS, и т.д.)

Это обеспечивает четкое понимание того, какие сайты система будет парсить и в каких форматах предоставлять данные.

---

**Версия**: 1.0
**Дата**: 2025-11-01
**Статус**: Draft
