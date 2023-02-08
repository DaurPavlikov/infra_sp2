# Развёртывание API YaMDB внутри Docker

**Описание**

Проект YaMDb является базой данных по сбору отзывов пользователей на произведения.
Произведения делятся на категории, такие как «Книги», «Фильмы», «Музыка». Список категорий может быть расширен, например, можно добавить категорию «Изобразительное искусство» или «Ювелирка», произведению может быть присвоен жанр из списка предустановленных, например, «Сказка», «Рок» или «Артхаус».
Добавлять произведения, категории и жанры может только администратор. Пользователи оставляют к произведениям текстовые отзывы и ставят произведению оценку в диапазоне от одного до десяти, из пользовательских оценок формируется усреднённая оценка произведения — рейтинг. На одно произведение пользователь может оставить только один отзыв.Добавлять отзывы, комментарии и ставить оценки могут только аутентифицированные пользователи.

Документация доступна по эндпойнту: http://localhost/redoc/

**Версии:**  
- [Python 3.7.5](https://www.python.org/doc/) 
- [Django 2.2.16](https://docs.djangoproject.com/en/4.1/releases/2.2.16/)
- [Django REST framework 3.12.4](https://www.django-rest-framework.org/)
- [DRF Simple JWT 5.2](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/)
- [Django Filter 21.1](https://django-filter.readthedocs.io/en/main/)
- [Docker 23.0](https://docs.docker.com/)
- [Docker Compose 1.29.2](https://docs.docker.com/compose/gettingstarted/)
- [PostgreSQL 14](https://www.postgresql.org/)
- [Gunicorn 20.0.4](https://gunicorn.org/)
- [nginx 1.22](https://nginx.org/en/)

**Как запустить проект:**

Клонируйте проект из репозитория:
```bash
git clone https://github.com/Koloyojik/infra_sp2
```
Перейдите в директорию *api_yamdb* внутри папки проекта:
```bash
cd infra_sp2/api_yamdb/
```

Первым делом необходимо создать виртуальное окружение и установить зависимости из *requirements.txt*:
```bash
python3 -m venv venv
source /venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

После этого преходим в папку с файлом *docker-compose.yaml* по пути:
```bash
cd infra_sp2/infra
```

Создаём и запускаем контейнеры *infra_db_1, infra_web_1, infra_nginx_1*:
```bash
docker-compose up -d --build
```
Выполняем миграции для базы данных и собираем статику в контейнер:
```bash
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py collectstatic --no-input
```
Загружаем данные в базу из резервной копии, если она есть, *fixtures.json*:
```bash
sudo docker cp ./fixtures.json <container_id>:/app/fixtures.json
docker-compose exec web python manage.py loaddata fixtures.json
```
Или создаем суперпользователя для заполнения новой базы данных:
```bash
docker-compose exec web python manage.py createsuperuser
```

Перед остановкой контейнера необходимо создать резервную копию баз данных командой:
```bash
docker-compose exec web python manage.py dumpdata > fixtures.json
```

После выполнения всех операций останавливаем контейнеры:
```bash
docker-compose down -v
```

**Шаблон .env-файла, который необходимо разместить по пути infra/.env**

Представлены значения по умолчанию
```
DB_ENGINE=django.db.backends.postgresql # Тип используемой БД
DB_NAME=postgres # Название БД
POSTGRES_USER=postgres # Имя PostgreSQL-пользователя
POSTGRES_PASSWORD=postgres # Пароль для доступа к указанной БД
DB_HOST=db
DB_PORT=5432
```

Автор: [**Даур Павликов**](https://github.com/Koloyojik)