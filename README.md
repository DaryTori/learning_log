# Learning Log

Учебный проект по курсу «Python & Django» (https://практика.ркси.рф), выполнен по разделу
«Проект learning_log». Learning Log — веб-журнал обучения: пользователи регистрируются,
создают темы для изучения и добавляют записи по каждой теме.

- **Python** 3.12
- **Django** 5.2 LTS
- **СУБД** SQLite
- **Оформление** Bootstrap 4 (пакет django-bootstrap4)
- **Продакшен-контейнер** Gunicorn + WhiteNoise (Docker Compose)

## Возможности

- Регистрация, вход и выход пользователей (приложение `users`)
- Каждый пользователь видит только собственные темы и записи (`@login_required` + проверка владельца)
- Создание тем и записей, редактирование записей через формы Django
- Административный сайт Django
- Автоматические тесты доступа (4 теста, `learning_logs/tests.py`)

## Демо-доступ

| Что | Значение |
|---|---|
| Суперпользователь | `admin` |
| Пароль | `admin123` |
| Админ-сайт | http://127.0.0.1:8000/admin/ |
| Демо-данные в базе | темы Chess (2 записи) и Rock Climbing (1 запись) |

## Структура проекта

```
learning_log/              ← папка проекта
├── manage.py
├── requirements.txt       ← зависимости
├── Dockerfile             ← образ для деплоя
├── compose.yaml           ← запуск через Docker Compose
├── .gitignore / .dockerignore
├── db.sqlite3             ← база с демо-данными
├── learning_log/          ← настройки проекта
│   ├── settings.py
│   └── urls.py
├── learning_logs/         ← приложение «журнал»
│   ├── models.py          ← модели Topic и Entry
│   ├── forms.py           ← TopicForm и EntryForm
│   ├── views.py           ← index, topics, topic, new_topic, new_entry, edit_entry
│   ├── urls.py
│   ├── tests.py           ← 4 автотеста
│   └── templates/learning_logs/
│       ├── base.html      ← общий шаблон с Bootstrap-навигацией
│       ├── index.html, topics.html, topic.html
│       └── new_topic.html, new_entry.html, edit_entry.html
└── users/                 ← приложение пользователей
    ├── views.py           ← register
    ├── urls.py            ← login/logout + register
    └── templates/registration/
        ├── login.html, logged_out.html, register.html
```

Виртуальное окружение `ll_env` в репозиторий не входит.

## Запуск на локальном компьютере (Windows 10, PowerShell)

Требуется установленный Python 3.12 (при установке отметьте «Add Python to PATH»).

```powershell
# 1. Перейти в папку проекта
cd C:\Users\Дарья\Documents\borovkova\learning_log

# 2. Создать виртуальное окружение (один раз)
py -3.12 -m venv ll_env

# 3. Активировать окружение
.\ll_env\Scripts\activate

# 4. Установить зависимости
python -m pip install -r requirements.txt

# 5. Запустить сервер разработки
python manage.py runserver
```

Откройте http://127.0.0.1:8000/ и войдите как `admin / admin123`
(или зарегистрируйте нового пользователя по ссылке Register).

При следующем запуске достаточно шагов 1 и 3, затем `python manage.py runserver`.

### Полезные команды

```powershell
python manage.py runserver          # запуск сервера разработки
python manage.py migrate            # применить миграции
python manage.py makemigrations learning_logs   # создать миграции
python manage.py createsuperuser    # новый суперпользователь
python manage.py shell              # интерактивная оболочка Django
python manage.py test               # автотесты (4 теста)
```

### Запуск автотестов

```powershell
python manage.py test
```

Ожидаемый результат: `Ran 4 tests ... OK`. Тесты проверяют: гость перенаправляется
на страницу входа; владелец видит свою тему; чужой пользователь получает 404;
GET-запрос на logout не разлогинивает (в Django 5.2 выход выполняется только POST).

## Развёртывание на сервере через Docker Compose

Требования: Docker и Docker Compose на сервере, порт выдан преподавателем.

### 1. Скопировать проект на сервер

```bash
git clone https://github.com/DaryTori/learning_log learning_log
cd learning_log
```

### 2. Создать файл `.env` в корне проекта

```env
DJANGO_SECRET_KEY=
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=192.168.1.11,localhost,127.0.0.1
DJANGO_CSRF_TRUSTED_ORIGINS=http://192.168.1.11:20010
DJANGO_USE_HTTPS=False
SQLITE_PATH=/app/data/db.sqlite3
APP_BIND_IP=0.0.0.0
APP_PORT=20240
```

### 3. Запустить

```bash
docker compose up -d --build
```

Контейнер автоматически применит миграции, соберёт статические файлы и запустит
Gunicorn. Приложение будет доступно по адресу `http://<IP сервера>:<порт>/`.

Файл `.env` содержит настройки конкретного сервера и **не загружается в GitHub**
(он уже добавлен в `.gitignore` и `.dockerignore`).

## Переменные окружения

| Переменная | Назначение | Значение по умолчанию |
|---|---|---|
| `DJANGO_SECRET_KEY` | Секретный ключ Django | ключ для локальной разработки |
| `DJANGO_DEBUG` | Режим отладки (`True`/`False`) | `True` |
| `DJANGO_ALLOWED_HOSTS` | Разрешённые хосты через запятую | `localhost,127.0.0.1` |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | Доверенные источники для CSRF | пусто |
| `DJANGO_USE_HTTPS` | Безопасные cookie при работе через HTTPS | `False` |
| `SQLITE_PATH` | Путь к файлу базы данных | `db.sqlite3` в корне проекта |
| `APP_BIND_IP` | Адрес, на котором публикуется порт (compose) | `127.0.0.1` |
| `APP_PORT` | Внешний порт приложения (compose) | `8000` |

## Восстановление базы с нуля

Если нужна пустая база (например, для проверки с нуля):

```powershell
del db.sqlite3
python manage.py migrate
python manage.py createsuperuser
```

На сервере база хранится в Docker-томе `learning_log_data` и переживает
пересоздание контейнера. Локальные пользователи и темы на сервер автоматически
не переносятся — создайте пробные данные заново.

## Что сдавать

По требованиям курса передаются две ссылки:

1. Публичный репозиторий проекта в GitHub (без `ll_env`, `db.sqlite3`, `.env` и `staticfiles/` — они в `.gitignore`).
2. Развёрнутое приложение: IP учебного сервера и выделенный порт.

---

Проект выполнен по учебнику Каламбета В.Б. (https://практика.ркси.рф).
