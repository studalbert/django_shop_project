# Интернет-магазин на Django

Простой онлайн-магазин: каталог товаров, корзина, регистрация пользователей, оформление заказов и оплата через **Stripe**.

## Технологии

- **Python 3** (локально в проекте используется виртуальное окружение с Python 3.12)
- **Django 6**
- **PostgreSQL**
- **Stripe** — приём платежей и вебхуки
- **Gunicorn** — сервер приложения в Docker
- **Docker Compose** — PostgreSQL, веб-приложение и **Nginx** (порты 80/443)

## Структура приложений

| Приложение | Назначение |
|------------|------------|
| `main` | Главная, каталог по категориям, карточка товара |
| `cart` | Корзина (сессия + middleware) |
| `users` | Пользователи (кастомная модель `CustomUser`) |
| `orders` | Заказы |
| `payment` | Stripe: успех/отмена оплаты, вебхук |

Админка Django: `/admin/`.

## Локальный запуск (без Docker)

1. Установите **PostgreSQL** и создайте базу данных.

2. Создайте виртуальное окружение и установите зависимости:

   ```bash
   python -m venv venv
   source venv/bin/activate   # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. В корне проекта создайте файл **`.env`** (скопируйте из примера ниже) и подставьте свои значения.

4. Примените миграции и запустите сервер разработки:

   ```bash
   python manage.py migrate
   python manage.py createsuperuser   # по желанию, для входа в админку
   python manage.py runserver
   ```

Сайт откроется по адресу `http://127.0.0.1:8000/`.

### Переменные окружения (`.env`)

Минимальный набор:

```env
SECRET_KEY=ваш-секретный-ключ-django

DB_NAME=имя_базы
DB_USER=пользователь_postgres
DB_PASSWORD=пароль
DB_HOST=localhost
DB_PORT=5432

STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

Для продакшена и Stripe вебхуков URL вебхука должен указывать на ваш домен, например:  
`https://ваш-домен/payment/stripe/webhook/`.

В `enf/settings.py` заданы `ALLOWED_HOSTS` и `CSRF_TRUSTED_ORIGINS` — при смене домена их нужно обновить.

## Запуск через Docker Compose

Из корня проекта:

```bash
docker compose up --build
```

Убедитесь, что в **`.env`** для контейнера веб-приложения указано подключение к базе в Docker-сети, например:

```env
DB_HOST=postgres
```

В `docker-compose.yml` сервис **web** поднимает Gunicorn на порту **8000**, **nginx** проксирует трафик на порты **80** и **443**. Для HTTPS в compose смонтирован каталог Let’s Encrypt — при локальной разработке это можно не использовать.

**Замечание:** в `docker-compose.yml` указан файл образа `Dockerfile`. На Linux регистр имени файла важен: если у вас файл называется `dockerfile`, переименуйте его в `Dockerfile` или поправьте путь в `docker-compose.yml`.

## Полезные команды Django

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic   # перед деплоем, если раздаёте static через веб-сервер
```

---

Проект создавался как учебный/рабочий каркас магазина; перед выкладкой в прод проверьте `DEBUG`, секреты, HTTPS и настройки безопасности в `settings.py`.
