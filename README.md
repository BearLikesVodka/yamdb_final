# Docker build for YaMDb project

[![YaMDb workflow](https://github.com/BearLikesVodka/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)](https://github.com/BearLikesVodka/yamdb_final/actions)

Проект YaMDb собирает отзывы пользователей на различные произведения.

Сборка `docker` представляет из собой три образа: `web`, `db`, `nginx`. При создании контейнеров из образов, будет создано два тома для работы со статикой и медиа файлами. Названия томов: `static_value`, `media_value`.

- `web` образ содержит сам проект, все зависимости, в том числе `gunicorn server` для корректной работы, документацию к API проекта YaMDb (v1.0);
- `nginx` содержит веб-сервер;
- `db` содержит базу данных проекта (PostgreSQL)

Оркестрацией контейнеров занимается `docker-compose`.

## Оглавление

1. [Описание сборки](#описание-сборки);
2. [Описание проекта](#описание-сборки);
3. [Стек технологий](#стек-технологий);
4. [Запуск проекта](#запуск-проекта);
5. [Документация API](#документация-api);

## Описание сборки

Сборка представляет из собой три образа: `web`, `db`, `nginx`.

- Конфигурация `web` образа описана в файле  `./api_yamdb/Dockerfile`, там же лежит файл с зависимостями `requirements.txt`;
- `db` образ включает образ с **docker-hub** — `postgres:13.0-alpine`,
- `nginx` включает образ с **docker-hub** `nginx:1.21.3-alpine`.

Инфраструктура проекта лежит в папке `./infra`:

- `docket-compose.yaml` # файл конфигурации `docker-compose`;
- `./nginx/default.conf` # файл настройки `nginx`;
- `fixture.json` # дамп базы данных с тестовыми публикациями.

При запуске контейнеров создаются два тома для хранения данных:

- `static_value`;
- `media_value`.

## Описание проекта

Проект YaMDb собирает отзывы пользователей на произведения. Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Произведению может быть присвоен жанр. Новые жанры может создавать только администратор. Читатели оставляют к произведениям текстовые отзывы и выставляют произведению рейтинг (оценку в диапазоне от одного до десяти). Из множества оценок автоматически высчитывается средняя оценка произведения.

## Стек технологий

- Docker
- Docker-compose
- проект написан на Python с использованием Django REST Framework
- библиотека Simple JWT - работа с JWT-токеном
- библиотека django-filter - фильтрация запросов
- база данных - PostgreSQL
- система управления версиями - git

## Запуск проекта

Создать и заполнить .env файл (шаблон для переменных окружения):

```env
DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД
```

Проверить, установлен ли Docker:

```bash
docker -v
docker-compose -v
```

Собрать образы и запустить проект на локальной машине:

```bash
docker-compose up -d --build
```

Выполнить миграции:

```bash
docker-compose exec web python manage.py migrate
```

Создать суперпользователя:

```bash
docker-compose exec web python manage.py createsuperuser
```

Собрать статику:

```bash
docker-compose exec web python manage.py collectstatic
```

Проект будет доступен по адресу [http://localhost/](http://localhost/)

## Документация API

Ознакомиться с полной документацией можно по адресу [http://localhost/redoc/](http://localhost/redoc/)

### Алгоритм получения токена

- Получить токен подтверждения;

В теле передать JSON POST-запрос на эндпоинт [http://localhost/api/v1/auth/signup/](http://localhost/api/v1/auth/signup/):

```json
{ "username": "имя_пользователя", "email": "адрес эл. почты" }
```

Затем, в папке `sent_emails` найти письмо и скопировать полученный код подтверждения.

- Получить токен

В теле передать JSON POST-запрос на эндпоинт [http://localhost/api/v1/auth/token/](http://localhost/api/v1/auth/token/)

```json
{ "username": "имя_пользователя", "confirmation_code": "код_подтвержения" }
```

Токен использовать для авторизации.
