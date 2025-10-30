# Content Factory - n8n Workflow для автоматизації збору веб-контенту

**Content Factory** — автоматизований workflow для збору та структурування веб-контенту через n8n, Apify та Google Sheets

🔗 **Репозиторій:** [https://github.com/sergiyscherbakov/n8n-content-factory](https://github.com/sergiyscherbakov/n8n-content-factory)

---

## 👨‍💻 Автор

**Розробник:** Сергій Щербаков
**Email:** [sergiyscherbakov@ukr.net](mailto:sergiyscherbakov@ukr.net)
**Telegram:** [@s_help_2010](https://t.me/s_help_2010)

### ☕ Підтримати розробку

Якщо цей проєкт допоміг вам зекономити час та автоматизувати збір контенту, ви можете подякувати чашкою кави:

> **💰 Донат на каву:**
>
> **USDT (Binance Smart Chain - BEP20)**
> ```
> 0xDFD0A23d2FEd7c1ab8A0F9A4a1F8386832B6f95A
> ```
>
> Ваша підтримка допомагає створювати більше корисних інструментів! 🙏

---

## 🚀 Швидкий старт

### Крок 1: Імпорт workflow в n8n

> **⚠️ ВАЖЛИВО:** Файл `Урок-11.json` потрібно імпортувати на хмарній платформі [n8n.io](https://n8n.io)

1. Зареєструйтесь або увійдіть на [n8n.io](https://n8n.io)
2. Перейдіть у **Workflows** → **Add workflow** → **Import from File**
3. Виберіть файл `Урок-11.json`
4. Workflow з'явиться у вашому робочому просторі

---

## 📊 Детальний опис workflow - ОСНОВНА ГІЛКА

Ця гілка автоматично збирає контент з веб-сайтів, перелік яких знаходиться у Google Sheets.

### Візуальна схема основної гілки

![Схема workflow](схема.png)

```
[Нода 1] Manual Trigger
    ↓
[Нода 2] Get row(s) in sheet (читає URL з Google Sheets)
    ↓
[Нода 3] Run an Actor (запускає Apify scraper для кожного URL)
    ↓
[Нода 4] Get dataset items (отримує зібрані дані з Apify)
    ↓
[Нода 5] Append row in sheet (зберігає результати в Google Sheets)
```

---

## НОДА 1️⃣: Manual Trigger - "When clicking 'Execute workflow'"

### Що робить
Запускає весь workflow вручну при натисканні кнопки "Execute Workflow" в n8n інтерфейсі.

### Коли спрацьовує
- Коли ви вручну натискаєте кнопку **"Execute Workflow"** у правому верхньому куті n8n

### Вхідні дані
**Немає вхідних даних** - це стартова точка workflow

### Вихідні дані
Просто сигнал для запуску наступної ноди:
```json
[
  {
    "json": {}
  }
]
```

### Як перевірити роботу
1. Відкрийте workflow в n8n
2. Натисніть **"Execute Workflow"**
3. Нода повинна стати зеленою ✅
4. У правій панелі побачите: `1 item`

---

## НОДА 2️⃣: Google Sheets - "Get row(s) in sheet"

### Що робить
Читає всі рядки з вхідної Google таблиці (`cc_input`), де знаходяться URL сайтів для парсингу.

### Коли спрацьовує
- Одразу після спрацювання Manual Trigger
- Виконується **один раз** на початку workflow

### Налаштування ноди

**Document ID:** `18elhWrPQ_v9XgutP4nU23zNrTrOvhRNhgAlgFwkrwiU` (ваша таблиця)
**Sheet Name:** `Лист1`
**Operation:** Read rows (читання рядків)

### Вхідні дані (структура вашої Google таблиці `cc_input`)

Ваша таблиця повинна мати таку структуру:

| Website |
|---------|
| https://example.com/page1 |
| https://techcrunch.com/ai-news |
| https://www.bbc.com/news/article123 |

**Мінімум:** 1 колонка з назвою **Website**

### Вихідні дані

Для кожного рядка таблиці створюється окремий об'єкт:

```json
[
  {
    "json": {
      "Website": "https://example.com/page1"
    }
  },
  {
    "json": {
      "Website": "https://techcrunch.com/ai-news"
    }
  },
  {
    "json": {
      "Website": "https://www.bbc.com/news/article123"
    }
  }
]
```

### Як перевірити роботу покроково

1. **Створіть тестову таблицю:**
   - Відкрийте [Google Sheets](https://sheets.google.com)
   - Створіть таблицю з назвою `cc_input_test`
   - У комірку A1 напишіть: `Website`
   - У комірку A2 напишіть: `https://example.com`
   - У комірку A3 напишіть: `https://techcrunch.com`

2. **Підключіть таблицю до ноди:**
   - Клікніть на ноду "Get row(s) in sheet"
   - У полі "Credential" → "Create New Credential"
   - Авторизуйтесь через Google
   - У полі "Document" виберіть вашу таблицю `cc_input_test`
   - Sheet Name: `Лист1`

3. **Запустіть тільки цю ноду:**
   - Клікніть на ноду "Get row(s) in sheet"
   - Натисніть **"Execute node"** (не Execute Workflow!)
   - Нода стане зеленою ✅

4. **Перевірте результат:**
   - Натисніть на ноду
   - У правій панелі побачите: `2 items` (якщо додали 2 URL)
   - Клікніть **"Table"** → побачите таблицю з даними:
     ```
     Website
     https://example.com
     https://techcrunch.com
     ```
   - Клікніть **"JSON"** → побачите JSON структуру

### Можливі помилки

❌ **"Insufficient permissions"**
- **Причина:** Немає доступу до таблиці
- **Рішення:** Переконайтесь, що ви залогінені під тим самим Google акаунтом, що й власник таблиці

❌ **"Document not found"**
- **Причина:** Неправильний Document ID
- **Рішення:** Скопіюйте ID з URL таблиці (частина між `/d/` та `/edit`)

❌ **"Sheet not found"**
- **Причина:** Неправильна назва аркуша
- **Рішення:** Перевірте назву аркуша внизу таблиці (за замовчуванням "Лист1")

---

## НОДА 3️⃣: Apify - "Run an Actor"

### Що робить
Для **кожного URL** з попередньої ноди запускає Apify Actor "Website Content Crawler", який:
- Відкриває веб-сторінку
- Витягує текстовий контент
- Видаляє непотрібні елементи (навігація, футер, реклама)
- Конвертує HTML у Markdown
- Зберігає результат у Apify Dataset

### Коли спрацьовує
- Після того, як нода "Get row(s) in sheet" успішно отримала список URL
- Виконується **для кожного URL окремо** (якщо 3 URL → 3 запуски Actor)

### Налаштування ноди

**Actor ID:** `aYG0l9s7dbB7j3gbS` (Website Content Crawler)
**Memory:** `4096 MB` (4GB RAM для Actor)

### Конфігурація Actor (customBody)

```json
{
  "crawlerType": "playwright:adaptive",
  "blockMedia": true,
  "saveMarkdown": true,
  "removeCookieWarnings": true,
  "expandIframes": true,
  "readableTextCharThreshold": 100,
  "removeElementsCssSelector": "nav, footer, script, style, noscript, svg, img[src^='data:'], [role=\"alert\"], [role=\"banner\"], [role=\"dialog\"]",
  "proxyConfiguration": {
    "useApifyProxy": true
  },
  "startUrls": [
    {
      "url": "{{ $json.Website }}",
      "method": "GET"
    }
  ]
}
```

**Пояснення параметрів:**

| Параметр | Значення | Що робить |
|----------|----------|-----------|
| `crawlerType` | `playwright:adaptive` | Використовує браузер (підтримка JS сайтів) |
| `blockMedia` | `true` | Не завантажує зображення/відео (швидше + дешевше) |
| `saveMarkdown` | `true` | Зберігає контент у Markdown форматі |
| `removeCookieWarnings` | `true` | Видаляє спливаючі вікна про cookies |
| `readableTextCharThreshold` | `100` | Мінімум 100 символів для тексту |
| `removeElementsCssSelector` | CSS селектор | Видаляє nav, footer, scripts |
| `startUrls[].url` | `{{ $json.Website }}` | Підставляє URL з попередньої ноди |

### Вхідні дані

Нода отримує дані з "Get row(s) in sheet":

**Приклад для ОДНОГО URL:**
```json
{
  "Website": "https://techcrunch.com/ai-news"
}
```

Ця нода **виконується для КОЖНОГО елемента окремо**. Якщо у вас 3 URL → Actor запуститься 3 рази.

### Вихідні дані

Після успішного запуску Actor повертає метадані:

```json
{
  "id": "xKg7dP2mN4vR8sL1",
  "actId": "aYG0l9s7dbB7j3gbS",
  "userId": "ваш_user_id",
  "startedAt": "2024-09-16T12:30:00.000Z",
  "finishedAt": "2024-09-16T12:32:15.000Z",
  "status": "SUCCEEDED",
  "defaultDatasetId": "AB12CD34EF56GH78",
  "stats": {
    "netRxBytes": 2458632,
    "netTxBytes": 125896
  }
}
```

**Найважливіше поле:** `defaultDatasetId` - це ID датасету, де зберігся зпарсений контент!

### Як перевірити роботу покроково

1. **Підготовка Apify облікового запису:**
   - Зареєструйтесь на [apify.com](https://apify.com)
   - Отримайте $5 безкоштовних кредитів
   - Перейдіть у [Settings → Integrations](https://console.apify.com/account/integrations)
   - Скопіюйте **API Token**

2. **Підключення Apify до n8n:**
   - Клікніть на ноду "Run an Actor"
   - У полі "Credential" → "Create New Credential"
   - Виберіть "Apify OAuth2 API"
   - Вставте API Token
   - Натисніть "Save"

3. **Тестовий запуск з одним URL:**
   - Відключіть зв'язок з попередньою нодою (видаліть стрілку)
   - Клікніть на ноду "Run an Actor"
   - Натисніть **"Execute node"**
   - В полі `startUrls[].url` вручну вставте тестовий URL: `https://example.com`

4. **Очікування виконання:**
   - Actor починає працювати (побачите статус "RUNNING")
   - Зачекайте 30-60 секунд
   - Нода стане зеленою ✅

5. **Перевірте результат:**
   - Клікніть на ноду після виконання
   - У правій панелі виберіть **"JSON"**
   - Знайдіть поле `defaultDatasetId` - це ID вашого датасету!
   - Скопіюйте це значення (наприклад, `AB12CD34EF56GH78`)

6. **Перевірте контент у Apify консолі:**
   - Відкрийте [https://console.apify.com/storage/datasets](https://console.apify.com/storage/datasets)
   - Знайдіть датасет з ID `AB12CD34EF56GH78`
   - Клікніть на нього
   - Побачите зпарсений контент у форматі JSON з полями `text`, `markdown`, `crawl` і т.д.

### Можливі помилки

❌ **"Actor run failed"**
- **Причина:** Недостатньо кредитів або Actor не зміг зпарсити сайт
- **Рішення:**
  1. Перевірте баланс на [console.apify.com](https://console.apify.com)
  2. Спробуйте простіший сайт (наприклад, `https://example.com`)

❌ **"Timeout"**
- **Причина:** Сайт завантажується занадто довго
- **Рішення:** Збільшіть timeout або зменшіть `memory` до 2048

❌ **"Empty dataset"**
- **Причина:** Actor не знайшов контент на сторінці
- **Рішення:**
  1. Зменшіть `readableTextCharThreshold` до 50
  2. Перевірте `removeElementsCssSelector` - можливо, він видаляє весь контент

---

## НОДА 4️⃣: Apify - "Get dataset items"

### Що робить
Отримує зпарсені дані з Apify Dataset за ID, який повернула попередня нода.

### Коли спрацьовує
- Одразу після успішного завершення "Run an Actor"
- Для **кожного Dataset ID** окремо

### Налаштування ноди

**Resource:** `Datasets`
**Dataset ID:** `={{ $json.defaultDatasetId }}` (динамічно з попередньої ноди)
**Limit:** `500` (максимум 500 елементів з одного датасету)

### Вхідні дані

Нода отримує `defaultDatasetId` з попередньої ноди:

```json
{
  "defaultDatasetId": "AB12CD34EF56GH78",
  "status": "SUCCEEDED",
  "id": "xKg7dP2mN4vR8sL1"
}
```

Нода витягує `AB12CD34EF56GH78` та робить запит до Apify API.

### Вихідні дані

Масив об'єктів з зпарсеним контентом:

**Приклад для ОДНОГО зпарсеного URL:**

```json
[
  {
    "url": "https://techcrunch.com/ai-news",
    "crawl": {
      "loadedUrl": "https://techcrunch.com/ai-news",
      "loadedTime": "2024-09-16T12:31:45.123Z",
      "httpStatusCode": 200
    },
    "metadata": {
      "title": "AI News - Latest Updates",
      "description": "Breaking news in artificial intelligence",
      "languageCode": "en"
    },
    "text": "# AI News\n\nLatest developments in artificial intelligence...\n\n## Key Highlights\n\n- New GPT model released\n- AI regulation updates\n- Industry partnerships announced",
    "markdown": "# AI News\n\nLatest developments in artificial intelligence...",
    "screenshotUrl": null
  }
]
```

**Структура даних:**

| Поле | Тип | Опис |
|------|-----|------|
| `url` | String | Оригінальний URL |
| `crawl.loadedUrl` | String | Фактичний URL (після редіректів) |
| `crawl.loadedTime` | DateTime | Час завантаження сторінки |
| `metadata.title` | String | Заголовок сторінки |
| `metadata.description` | String | Meta опис |
| `text` | String | **Основний контент у Markdown** |
| `markdown` | String | Альтернативний Markdown |

### Як перевірити роботу покроково

1. **Запустіть попередню ноду:**
   - Спочатку виконайте ноду "Run an Actor"
   - Дочекайтесь зеленої позначки ✅
   - Скопіюйте `defaultDatasetId` з результату

2. **Тестовий запуск з фіксованим Dataset ID:**
   - Відключіть зв'язок з попередньою нодою
   - Клікніть на ноду "Get dataset items"
   - Змініть параметр `datasetId` з `={{ $json.defaultDatasetId }}` на конкретне значення:
     ```
     AB12CD34EF56GH78
     ```
   - Натисніть **"Execute node"**

3. **Перевірте результат:**
   - Нода стане зеленою ✅
   - У правій панелі побачите кількість елементів (наприклад, `1 item`)
   - Клікніть **"Table"** → побачите таблицю з колонками `url`, `text`, `metadata`
   - Клікніть **"JSON"** → побачите повний JSON

4. **Перевірте поле `text`:**
   - У JSON знайдіть поле `text`
   - Там буде зпарсений контент у форматі Markdown
   - Якщо поле порожнє - перевірте налаштування Actor у попередній ноді

### Можливі помилки

❌ **"Dataset not found"**
- **Причина:** Неправильний Dataset ID або датасет видалено
- **Рішення:**
  1. Перевірте, що попередня нода успішно завершилась
  2. Відкрийте [Apify Datasets](https://console.apify.com/storage/datasets) та перевірте існування датасету

❌ **"Empty result"**
- **Причина:** Датасет існує, але порожній
- **Рішення:**
  1. Перевірте налаштування Actor у ноді "Run an Actor"
  2. Можливо, сайт заблокував скрейпінг

❌ **"Rate limit exceeded"**
- **Причина:** Занадто багато запитів до Apify API
- **Рішення:** Додайте ноду "Wait" на 2-3 секунди між запитами

---

## НОДА 5️⃣: Google Sheets - "Append row in sheet"

### Що робить
Додає нові рядки у вихідну Google таблицю (`cc_output`) з зпарсеними даними.

### Коли спрацьовує
- Після того, як нода "Get dataset items" успішно отримала дані
- Для **кожного елемента датасету окремо** створюється новий рядок

### Налаштування ноди

**Document ID:** `1tlNjUavcyCaNoehcOOdZUqt4ZHjcNjvdF-IYm_VJ0Xk` (ваша вихідна таблиця)
**Sheet Name:** `Лист1`
**Operation:** `Append` (додавання рядків у кінець таблиці)

### Mapping колонок

```json
{
  "loadedTime": "={{ $json.crawl.loadedTime }}",
  "loadedUrl": "={{ $json.crawl.loadedUrl }}",
  "text": "={{ $json.text }}"
}
```

**Пояснення:**
- `loadedTime` ← Час завантаження з `crawl.loadedTime`
- `loadedUrl` ← Фактичний URL з `crawl.loadedUrl`
- `text` ← Зпарсений контент з `text`

### Вхідні дані

Дані з попередньої ноди "Get dataset items":

```json
{
  "crawl": {
    "loadedUrl": "https://techcrunch.com/ai-news",
    "loadedTime": "2024-09-16T12:31:45.123Z"
  },
  "text": "# AI News\n\nLatest developments in AI..."
}
```

### Вихідні дані

Після успішного додавання:

```json
{
  "success": true,
  "spreadsheetId": "1tlNjUavcyCaNoehcOOdZUqt4ZHjcNjvdF-IYm_VJ0Xk",
  "updatedRange": "Лист1!A2:C2",
  "updatedRows": 1
}
```

### Як виглядає вихідна таблиця (cc_output)

Після виконання workflow ваша таблиця матиме такий вигляд:

| loadedTime | loadedUrl | text |
|------------|-----------|------|
| 2024-09-16T12:31:45.123Z | https://techcrunch.com/ai-news | # AI News<br><br>Latest developments in AI... |
| 2024-09-16T12:33:12.456Z | https://example.com/page1 | # Example Domain<br><br>This domain is for use in examples... |
| 2024-09-16T12:34:50.789Z | https://bbc.com/news/article123 | # Breaking News<br><br>Top story today... |

### Як перевірити роботу покроково

1. **Створіть вихідну таблицю:**
   - Відкрийте [Google Sheets](https://sheets.google.com)
   - Створіть таблицю з назвою `cc_output_test`
   - У перший рядок додайте заголовки:
     - A1: `loadedTime`
     - B1: `loadedUrl`
     - C1: `text`

2. **Підключіть таблицю до ноди:**
   - Клікніть на ноду "Append row in sheet"
   - У полі "Credential" використайте той самий Google Sheets credential
   - У полі "Document" виберіть `cc_output_test`
   - Sheet Name: `Лист1`

3. **Перевірте mapping:**
   - У розділі "Columns" переконайтесь, що mapping правильний:
     ```
     loadedTime = {{ $json.crawl.loadedTime }}
     loadedUrl = {{ $json.crawl.loadedUrl }}
     text = {{ $json.text }}
     ```

4. **Тестовий запуск:**
   - Запустіть весь workflow від початку (Execute Workflow)
   - Або запустіть тільки цю ноду з тестовими даними

5. **Перевірте таблицю:**
   - Відкрийте вашу таблицю `cc_output_test`
   - У другому рядку (A2:C2) повинні з'явитись дані
   - Колонка `text` буде містити Markdown контент

### Можливі помилки

❌ **"Column header not found"**
- **Причина:** У таблиці немає колонки `loadedTime`, `loadedUrl` або `text`
- **Рішення:**
  1. Відкрийте вихідну таблицю
  2. Перевірте, що у першому рядку є headers: `loadedTime`, `loadedUrl`, `text`
  3. Назви мають точно збігатись (з урахуванням регістру)

❌ **"Insufficient permissions"**
- **Причина:** Немає прав на редагування таблиці
- **Рішення:** Переконайтесь, що ви є власником таблиці або маєте права Editor

❌ **"Data not appearing in sheet"**
- **Причина:** Mapping неправильний або дані порожні
- **Рішення:**
  1. Клікніть на ноду після виконання
  2. Перевірте вхідні дані у розділі "Input"
  3. Переконайтесь, що поля `$json.crawl.loadedTime`, `$json.crawl.loadedUrl`, `$json.text` існують

---

## 📊 АЛЬТЕРНАТИВНА ГІЛКА - Пряме читання з датасету

Ця гілка дозволяє отримати дані з **існуючого** Apify датасету без повторного запуску скрейпера.

```
[Нода 6] Get dataset items1 (прямий доступ)
    ↓
[Нода 7] Append row in sheet1 (збереження)
```

---

## НОДА 6️⃣: Apify - "Get dataset items1"

### Що робить
Отримує дані з **фіксованого** Apify Dataset за ID `CauuiPzhZfhEKj4FJ`.

### Коли використовувати
- Коли у вас вже є готовий датасет з попереднього запуску
- Коли потрібно повторно імпортувати дані без повторного скрейпінгу
- Коли потрібно протестувати workflow без витрат на Apify кредити

### Налаштування ноди

**Resource:** `Datasets`
**Dataset ID:** `CauuiPzhZfhEKj4FJ` (фіксоване значення)
**Limit:** `500`

### Вхідні дані
**Немає вхідних даних** - нода працює незалежно

### Вихідні дані
Аналогічно ноді "Get dataset items" (нода 4):

```json
[
  {
    "url": "https://example.com",
    "crawl": {
      "loadedUrl": "https://example.com",
      "loadedTime": "2024-09-16T10:00:00.000Z"
    },
    "text": "# Example Domain\n\nThis domain is for use in examples..."
  }
]
```

### Як перевірити роботу

1. **Знайдіть Dataset ID:**
   - Відкрийте [Apify Datasets](https://console.apify.com/storage/datasets)
   - Виберіть будь-який існуючий датасет
   - Скопіюйте його ID (наприклад, `CauuiPzhZfhEKj4FJ`)

2. **Вставте ID у ноду:**
   - Клікніть на ноду "Get dataset items1"
   - У полі "Dataset ID" вставте ID вашого датасету
   - Натисніть **"Execute node"**

3. **Перевірте результат:**
   - Нода стане зеленою ✅
   - У правій панелі побачите кількість елементів
   - Перевірте дані у форматі Table або JSON

---

## НОДА 7️⃣: Google Sheets - "Append row in sheet1"

### Що робить
Ідентична ноді 5 - додає рядки у вихідну таблицю `cc_output`.

### Налаштування
Повністю аналогічні ноді "Append row in sheet" (нода 5).

---

## 🧪 Повний тестовий сценарій - Перевірка всього workflow

### Сценарій 1: Мінімальний тест з одним URL

1. **Підготовка:**
   - Створіть `cc_input` з одним URL: `https://example.com`
   - Створіть `cc_output` з headers: `loadedTime`, `loadedUrl`, `text`
   - Підключіть Google Sheets credentials
   - Підключіть Apify credentials

2. **Запуск:**
   - Натисніть **"Execute Workflow"**
   - Дочекайтесь завершення (всі ноди зелені)

3. **Перевірка:**
   - Відкрийте `cc_output`
   - У рядку 2 повинен з'явитись контент з https://example.com
   - Колонка `text` містить Markdown контент

4. **Очікуваний час виконання:** 30-60 секунд

### Сценарій 2: Тест з 3 URL

1. **Підготовка:**
   - Додайте у `cc_input` три URL:
     - https://example.com
     - https://example.org
     - https://example.net

2. **Запуск:** Execute Workflow

3. **Перевірка:**
   - У `cc_output` повинно з'явитись 3 рядки
   - Кожен рядок з контентом відповідного сайту

4. **Очікуваний час:** 90-180 секунд (30-60 сек на кожен URL)

### Сценарій 3: Покрокова перевірка кожної ноди

**Крок 1: Перевірка ноди 2**
- Відключіть всі зв'язки
- Виконайте тільки "Get row(s) in sheet"
- Переконайтесь, що отримали список URL

**Крок 2: Перевірка ноди 3**
- Підключіть ноду 2 → ноду 3
- Виконайте до ноди 3
- Перевірте, що отримали `defaultDatasetId`

**Крок 3: Перевірка ноди 4**
- Підключіть ноду 3 → ноду 4
- Виконайте до ноди 4
- Перевірте, що отримали поле `text` з контентом

**Крок 4: Перевірка ноди 5**
- Підключіть ноду 4 → ноду 5
- Виконайте до ноди 5
- Перевірте таблицю `cc_output`

---

## 🔍 Debugging - Як знайти проблему

### Якщо workflow не працює:

1. **Перевірте кожну ноду окремо:**
   - Клікайте на ноди по черзі
   - Натискайте "Execute node"
   - Дивіться на результат у правій панелі

2. **Перевірте вхідні дані:**
   - Клікніть на ноду після виконання
   - Виберіть вкладку "Input"
   - Переконайтесь, що дані правильні

3. **Перевірте вихідні дані:**
   - Виберіть вкладку "Output"
   - Перевірте, чи є очікувані поля

4. **Перевірте expressions:**
   - У mapping переконайтесь, що `{{ $json.crawl.loadedTime }}` правильний
   - Клікніть на поле → "Expression" → побачите попередній перегляд

5. **Перевірте execution log:**
   - Внизу n8n натисніть "Executions"
   - Виберіть останнє виконання
   - Подивіться детальний лог кожної ноди

---

## 💰 Вартість використання

### Apify Free Tier
- **$5 кредитів/місяць**
- **1 запуск Website Content Crawler ≈ $0.01-0.05** (залежить від складності сайту)
- **Приблизно 100-500 URL/місяць безкоштовно**

### Порада: як економити кредити
1. Використовуйте `blockMedia: true` (вже налаштовано)
2. Зменшіть `memory` до 2048 для простих сайтів
3. Використовуйте `crawlerType: cheerio` для статичних сайтів (замість `playwright`)

---

## 📚 Додаткові ресурси

- [Офіційна документація n8n](https://docs.n8n.io/)
- [Apify Website Content Crawler](https://apify.com/apify/website-content-crawler)
- [Google Sheets API](https://developers.google.com/sheets/api)

---

## 🤝 Контакти

- 📧 **Email:** [sergiyscherbakov@ukr.net](mailto:sergiyscherbakov@ukr.net)
- 💬 **Telegram:** [@s_help_2010](https://t.me/s_help_2010)

---

**🇺🇦 Зроблено в Україні**
