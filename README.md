# Olist E-commerce: SQL/PostgreSQL домашнє завдання

Навчальний проєкт з PostgreSQL на датасеті бразильського маркетплейсу Olist: створення raw- і typed-таблиць, DML із PostgreSQL-фічами, JOIN-запити, агрегації, set operations та window functions. Робота виконана в Jupyter/Colab notebook (`.ipynb`).

## Джерело датасету

- **Назва:** Brazilian E-Commerce Public Dataset by Olist
- **Платформа:** Kaggle
- **Ідентифікатор:** `olistbr/brazilian-ecommerce`
- **Посилання:** https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- **Опис:** реальні знеособлені дані про замовлення, клієнтів, продавців, товари та відгуки (2016–2018).

Датасет історичний (останні замовлення датуються 2018 роком), тому recency у сегментації рахується від дати, обчисленої з самого датасету, а не від `CURRENT_DATE`.

## Спосіб отримання файлів

Файли завантажуються через бібліотеку `kagglehub`:

```python
import kagglehub

dataset_dir = kagglehub.dataset_download('olistbr/brazilian-ecommerce')
```

## Використані таблиці

### Raw-таблиці (завантажені з CSV без перетворення типів)

| Raw-таблиця | Файл |
|---|---|
| `olist_customers_raw` | `olist_customers_dataset.csv` |
| `olist_orders_raw` | `olist_orders_dataset.csv` |
| `olist_order_items_raw` | `olist_order_items_dataset.csv` |
| `olist_products_raw` | `olist_products_dataset.csv` |
| `olist_sellers_raw` | `olist_sellers_dataset.csv` |
| `olist_order_reviews_raw` | `olist_order_reviews_dataset.csv` |

### Typed-таблиці (з первинними/зовнішніми ключами та CHECK-обмеженнями)

`olist_customers`, `olist_orders`, `olist_order_items`, `olist_products`, `olist_sellers`, `olist_order_reviews`. Дані переносяться з raw у typed через `INSERT ... SELECT`.

> У вихідному CSV колонки названо з помилкою: `product_name_lenght` і `product_description_lenght`. У typed-схемі вони нормалізовані до `product_name_length` і `product_description_length`, а в `SELECT` з raw-таблиці використовуються фактичні назви з CSV.

### Похідні та допоміжні таблиці

| Таблиця | Призначення |
|---|---|
| `olist_customer_dim` | вимір клієнтів, один рядок на `customer_unique_id` (ціль для FK) |
| `customer_segments` | сегментація клієнтів (VIP / regular / new / inactive) |
| `seller_score` | агрегований рейтинг продавців |
| `seller_alerts` | журнал сповіщень про продавців |

## Структура роботи

1. Створення raw- і typed-таблиць.
2. DML-запити з PostgreSQL-фічами: `INSERT ... RETURNING`, `ON CONFLICT DO UPDATE`, `UPDATE ... FROM`, `DELETE ... RETURNING`.
3. Сегментація клієнтів із `reference_date` з датасету.
4. JOIN-запити з business-interpretation (INNER, LEFT + `COALESCE`, `FULL OUTER`, self join, anti-join).
5. `GROUP BY`, `HAVING` і агрегати (`STRING_AGG`).
6. Set operations, window function та reflection.

## Інструкція запуску ipynb

### Вимоги

- Python 3.9+
- PostgreSQL (локальний або хмарний інстанс)
- Пакети: `kagglehub`, `pandas`, `sqlalchemy`, `psycopg2-binary` (або `psycopg`)

### Кроки

1. Клонуйте репозиторій або завантажте `.ipynb` файл.
2. Запустіть notebook у Google Colab. Для Colab завантажте файл через **File → Upload notebook**.
5. Виконуйте клітинки **послідовно зверху вниз** (Run all). Порядок важливий: typed-таблиці створюються після raw, а `customer_segments` залежить від `olist_customer_dim`.

### Повторний запуск

Клітинки з `DROP TABLE IF EXISTS ... CASCADE` перестворюють таблиці з нуля, а upsert-запити (`ON CONFLICT`) безпечні для повторного виконання й не дублюють рядки.
