# Контент Завод для n8n

## Опис

**Контент Завод (Content Factory)** — це автоматизована система для збору контенту з веб-ресурсів через Apify платформу. Workflow витягує інформацію з новинних сайтів, YouTube, TikTok, Instagram, Telegram та інших джерел, структурує її та зберігає у Google Sheets для подальшої обробки та генерації цільового контенту.

## Призначення системи

Контент Завод — це перша частина повного конвеєра контент-маркетингу:

1. ✅ **Збір інформації** з необхідних веб-ресурсів
2. ✅ **Структурування і збереження** в таблицю
3. 🔄 Ранжування, сортування і аналіз на відповідність контенту
4. 🔄 Передача на агентів/мультиагентів для генерації контенту
5. 🔄 Генерація зображень, відео або аудіо під контент
6. 🔄 Публікація в соціальні мережі або на сайт

**Поточний workflow** реалізує перші два етапи — автоматичний збір та структурування даних.

## Архітектура системи

### Основний workflow (Запуск скрейпера)

```
┌─────────────────────────┐
│ When clicking           │
│ 'Execute workflow'      │
│ (Manual Trigger)        │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Get row(s) in sheet     │
│ (Read URLs from         │
│  cc_input)              │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Run an Actor            │
│ (Apify Website          │
│  Content Crawler)       │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Get dataset items       │
│ (Fetch crawled data)    │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Append row in sheet     │
│ (Save to cc_output)     │
└─────────────────────────┘
```

### Альтернативний workflow (Витягування з датасету)

```
┌─────────────────────────┐
│ Get dataset items1      │
│ (Direct dataset read)   │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Append row in sheet1    │
│ (Save to cc_output)     │
└─────────────────────────┘
```

## Опис вузлів

### 1. When clicking 'Execute workflow'
**Тип:** Manual Trigger
**Призначення:** Ручний запуск workflow для початку процесу збору контенту.

**Вихід:**
- Сигнал запуску workflow

### 2. Get row(s) in sheet
**Тип:** Google Sheets
**Призначення:** Читає список URL-адрес з вхідної таблиці Google Sheets.

**Параметри:**
- Document ID: `18elhWrPQ_v9XgutP4nU23zNrTrOvhRNhgAlgFwkrwiU`
- Sheet Name: `Лист1`
- Table Name: `cc_input`

**Структура вхідної таблиці:**
```
| Website                          |
|----------------------------------|
| https://example.com/page1        |
| https://example.com/page2        |
| ...                              |
```

**Вхід:**
- Trigger від Manual Trigger

**Вихід:**
- Масив рядків з URL-адресами веб-сайтів

### 3. Run an Actor
**Тип:** Apify
**Призначення:** Запускає Apify Actor для парсингу веб-сайтів та витягування контенту.

**Параметри:**
- Actor ID: `aYG0l9s7dbB7j3gbS` (Website Content Crawler)
- Memory: `4096 MB`

**Конфігурація Actor:**
```json
{
  "aggressivePrune": false,
  "blockMedia": true,
  "crawlerType": "playwright:adaptive",
  "expandIframes": true,
  "saveMarkdown": true,
  "startUrls": [
    {
      "url": "{{ $json.Website }}",
      "method": "GET"
    }
  ],
  "removeCookieWarnings": true,
  "removeElementsCssSelector": "nav, footer, script, style, noscript, svg...",
  "readableTextCharThreshold": 100
}
```

**Основні налаштування:**
- `blockMedia: true` — блокує завантаження медіа-файлів для швидшості
- `saveMarkdown: true` — зберігає контент у форматі Markdown
- `crawlerType: "playwright:adaptive"` — використовує адаптивний краулер з Playwright
- `removeCookieWarnings: true` — видаляє попередження про cookies
- `readableTextCharThreshold: 100` — мінімальна кількість символів для читабельного тексту

**Вхід:**
- URL веб-сайту з Google Sheets

**Вихід:**
- `defaultDatasetId` — ID датасету з зібраними даними
- Метадані про виконання Actor

### 4. Get dataset items
**Тип:** Apify (Datasets)
**Призначення:** Витягує зібрані дані з Apify Dataset.

**Параметри:**
- Resource: `Datasets`
- Dataset ID: `{{ $json.defaultDatasetId }}` (динамічно з попереднього кроку)
- Limit: `500` елементів

**Вхід:**
- Dataset ID від Run an Actor

**Вихід:**
- Масив елементів з даними:
  ```json
  {
    "crawl": {
      "loadedTime": "2024-09-16T12:30:00.000Z",
      "loadedUrl": "https://example.com/page1"
    },
    "text": "Extracted content in markdown format...",
    "metadata": { ... }
  }
  ```

### 5. Append row in sheet
**Тип:** Google Sheets
**Призначення:** Зберігає витягнуті дані у вихідну таблицю Google Sheets.

**Параметри:**
- Document ID: `1tlNjUavcyCaNoehcOOdZUqt4ZHjcNjvdF-IYm_VJ0Xk`
- Sheet Name: `Лист1`
- Table Name: `cc_output`
- Operation: `append`

**Mapping колонок:**
```json
{
  "loadedTime": "{{ $json.crawl.loadedTime }}",
  "loadedUrl": "{{ $json.crawl.loadedUrl }}",
  "text": "{{ $json.text }}"
}
```

**Структура вихідної таблиці:**
```
| loadedTime              | loadedUrl                    | text                          |
|-------------------------|------------------------------|-------------------------------|
| 2024-09-16T12:30:00.000Z| https://example.com/page1    | Markdown content here...      |
| 2024-09-16T12:31:00.000Z| https://example.com/page2    | More content...               |
```

**Вхід:**
- Дані від Get dataset items

**Вихід:**
- Підтвердження успішного додавання рядків

### 6. Get dataset items1 (Альтернативний варіант)
**Тип:** Apify (Datasets)
**Призначення:** Прямий доступ до конкретного датасету без запуску Actor.

**Параметри:**
- Resource: `Datasets`
- Dataset ID: `CauuiPzhZfhEKj4FJ` (фіксований ID)
- Limit: `500` елементів

**Використання:** Цей вузол корисний для витягування даних з раніше створеного датасету без повторного запуску краулера.

### 7. Append row in sheet1
**Тип:** Google Sheets
**Призначення:** Зберігає дані з альтернативного workflow у ту саму вихідну таблицю.

**Параметри:** Ідентичні до **Append row in sheet**

## Налаштування workflow

### 1. Імпорт у n8n

1. Відкрийте n8n
2. Перейдіть до **Import from File**
3. Завантажте файл `Урок-11.json`

### 2. Створення вхідної таблиці Google Sheets

1. Створіть нову Google Sheets таблицю з назвою `cc_input`
2. Додайте заголовок стовпця: `Website`
3. Заповніть стовпець URL-адресами веб-сайтів, які потрібно парсити:
   ```
   Website
   https://example.com/article1
   https://example.com/article2
   https://news.site.com/news/123
   ```
4. Скопіюйте Document ID з URL таблиці

### 3. Створення вихідної таблиці Google Sheets

1. Створіть нову Google Sheets таблицю з назвою `cc_output`
2. Додайте заголовки стовпців:
   ```
   loadedTime | loadedUrl | text
   ```
3. Скопіюйте Document ID з URL таблиці

### 4. Підключення Google Sheets

1. Відкрийте вузол **Get row(s) in sheet**
2. Натисніть на **Credentials**
3. Додайте **Google Sheets OAuth2 credentials**
4. Вкажіть Document ID вхідної таблиці (`cc_input`)
5. Повторіть для вузла **Append row in sheet** з Document ID вихідної таблиці (`cc_output`)

### 5. Підключення Apify

1. Створіть акаунт на https://apify.com/ (є безкоштовний план)
2. Отримайте API токен в розділі **Settings → Integrations**
3. Відкрийте вузол **Run an Actor**
4. Натисніть на **Credentials**
5. Додайте **Apify OAuth2 API credentials**
6. Вставте API токен

### 6. Налаштування Actor (опціонально)

Ви можете змінити параметри Website Content Crawler:

**Для швидшого парсингу:**
```json
{
  "blockMedia": true,
  "saveScreenshots": false,
  "saveHtml": false
}
```

**Для більш глибокого сканування:**
```json
{
  "maxCrawlDepth": 3,
  "useSitemaps": true
}
```

**Для парсингу JavaScript-сайтів:**
```json
{
  "crawlerType": "playwright:adaptive",
  "waitForSelector": ".content-loaded"
}
```

## Приклади використання

### Приклад 1: Збір новин з кількох сайтів

**Вхідна таблиця (cc_input):**
```
Website
https://techcrunch.com/
https://www.theverge.com/
https://www.wired.com/
```

**Результат:**
Система зпарсить головні сторінки цих сайтів та збереже контент у форматі Markdown у вихідну таблицю.

### Приклад 2: Моніторинг блогу конкурентів

**Вхідна таблиця (cc_input):**
```
Website
https://competitor.com/blog/post-1
https://competitor.com/blog/post-2
https://competitor.com/blog/post-3
```

**Результат:**
Отримаєте структурований текст статей конкурентів для аналізу та генерації власного контенту.

### Приклад 3: Збір даних для навчання AI

**Використання:** Зберіть великий обсяг текстових даних для:
- Навчання власних AI-моделей
- Створення векторної бази знань
- Формування контент-стратегії

## Інтеграція з подальшими етапами

Після збору даних у `cc_output` таблицю, ви можете:

### Етап 3: Ранжування та аналіз
- Додайте AI Agent для аналізу релевантності контенту
- Використайте sentiment analysis для оцінки тональності
- Створіть систему тегування за темами

### Етап 4: Генерація контенту
- Підключіть Multi-Agent System для створення статей
- Використайте промпт-інженерію для адаптації стилю
- Додайте перевірку на унікальність

### Етап 5: Генерація медіа
- Інтегруйте Fal.AI або DALL-E для зображень
- Додайте text-to-speech для аудіо
- Створіть відео-контент через Synthesia або подібні сервіси

### Етап 6: Публікація
- Підключіть API соціальних мереж (Twitter, Facebook, LinkedIn)
- Налаштуйте WordPress API для публікації на сайт
- Додайте Telegram Bot для розсилки

## Troubleshooting

### Помилка: "Actor run failed"

**Проблема:** Apify Actor не може запуститися або падає під час виконання.

**Рішення:**
1. Перевірте баланс Apify акаунту (достатньо коштів або free tier)
2. Зменште параметр `memory` з 4096 до 2048 MB
3. Перевірте, чи доступні URL-адреси (не заблоковані, не 404)
4. Спробуйте змінити `crawlerType` з `playwright:adaptive` на `cheerio`

### Помилка: "Failed to get dataset items"

**Проблема:** Не вдається отримати дані з датасету.

**Рішення:**
1. Перевірте, що Actor успішно завершився
2. Переконайтесь, що Dataset ID правильний
3. Збільште затримку між запуском Actor та читанням датасету (додайте Wait node)

### Помилка: "Google Sheets: Insufficient permissions"

**Проблема:** Немає прав доступу до Google Sheets.

**Рішення:**
1. Перевірте OAuth2 credentials для Google Sheets
2. Переконайтесь, що додали правильний Document ID
3. Перевірте права доступу до таблиці (потрібен Editor або Owner)

### Дані не зберігаються у Google Sheets

**Проблема:** Workflow виконується, але рядки не додаються.

**Рішення:**
1. Перевірте mapping колонок у вузлі **Append row in sheet**
2. Переконайтесь, що структура таблиці відповідає mapping
3. Перевірте, чи отримує вузол дані (подивіться на execution log)
4. Спробуйте змінити `operation` з `append` на `appendOrUpdate`

### Контент витягується не повністю

**Проблема:** У таблиці з'являється неповний або порожній текст.

**Рішення:**
1. Збільште `readableTextCharThreshold` до 50
2. Налаштуйте `removeElementsCssSelector` для конкретного сайту
3. Додайте `waitForSelector` для динамічних сайтів
4. Увімкніть `debugMode: true` для діагностики
5. Перевірте, чи використовується правильний `crawlerType`

## Обмеження та рекомендації

### Обмеження Apify Free Tier:
- $5 безкоштовних кредитів на місяць
- Обмежена кількість concurrent runs
- Максимальна тривалість run: 300 секунд

### Рекомендації:
1. **Batch processing:** Групуйте URL-адреси для економії запусків
2. **Incremental crawling:** Збирайте тільки нові сторінки
3. **Rate limiting:** Додайте затримки між запитами
4. **Monitoring:** Налаштуйте сповіщення про помилки через Email або Telegram
5. **Backup:** Регулярно експортуйте дані з Google Sheets

## Автор

**Розробник:** Сергій Щербаков
**Email:** sergiyscherbakov@ukr.net
**Telegram:** @s_help_2010

## Ліцензія

Цей шаблон надається "як є" для освітніх та комерційних цілей.
