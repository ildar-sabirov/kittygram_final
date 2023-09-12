## О проекте:
![workflow](https://github.com/ildar-sabirov/kittygram_final/actions/workflows/main.yml/badge.svg)

Kittygram - социальная сеть для обмена фотографиями любимых питомцев.

## Технологии:

+ python 3.10.12
+ Django 3.2.3
+ djangorestframework 3.12.4
+ JSON Web Token Authentication
+ SQLite
+ gunicorn
+ nginx
+ Docker

## Установка проекта:

Клонируйте репозиторий и перейдите в него в командной строке:
```
git clone git@github.com:ildar-sabirov/kittygram_final.git
```
Заполните .env по примеру .env.example.

## Собираем образы Docker:

В терминале в корне проекта Taski последовательно выполните команды из листинга; замените username на ваш логин на Docker Hub:
```
cd frontend  # В директории frontend...
docker build -t username/taski_frontend .  # ...сбилдить образ, назвать его taski_frontend
cd ../backend  # То же в директории backend...
docker build -t username/taski_backend .
cd ../gateway  # ...то же и в gateway
docker build -t username/taski_gateway . 
```

## Загружаем образы на Docker Hub:

Отправьте собранные образы фронтенда, бэкенда и Nginx на Docker Hub:
```
docker push username/taski_frontend
docker push username/taski_backend
docker push username/taski_gateway
```

## Настраиваем и копируем конфиг для сервера:

В docker-compose.production.yml замените username на ваш логин на Docker Hub.

Скопируйте на сервер в директорию taski/ файл docker-compose.production.yml. 
```
scp -i path_to_SSH/SSH_name docker-compose.production.yml \
    username@server_ip:/home/username/taski/docker-compose.production.yml
```

- `path_to_SSH` — путь к файлу с SSH-ключом;
- `SSH_name` — имя файла с SSH-ключом (без расширения);
- `username` — ваше имя пользователя на сервере;
- `server_ip` — IP вашего сервера.

Скопируйте файл .env на сервер, в директорию kittygram/.

## Устанавливаем Docker Compose на сервер:

Поочерёдно выполните на сервере команды для установки Docker и Docker Compose для Linux.
```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin
```

## Запускаем Docker Compose на сервере:

Для запуска Docker Compose в режиме демона команду docker compose up нужно запустить с флагом -d. Выполните эту команду на сервере в папке kittygram/:
```
sudo docker compose -f docker-compose.production.yml up -d
```
Сразу после запуска всех контейнеров Docker Compose перейдёт в фоновый режим, а вы сможете вводить новые команды в терминал. 

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

## Перенаправляем все запросы в докер.

На сервере в редакторе nano откройте конфиг Nginx: nano /etc/nginx/sites-enabled/default. Измените настройки location в секции server.
```
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
```

Перезагрузите конфиг Nginx:
```
sudo service nginx reload
```

Откройте в браузере страницу админки вашего проекта — https://ваш_домен/ cтраница должна отображаться правильно, со стилевым оформлением. Если же статика всё равно не подгрузилась, очистите кеш браузера и перезагрузите страницу

## Настройка CI/CD:

Workflow уже создан, замените username на ваш логин на Docker Hub по пути:
```
kittygram/.github/workflows/main.yml
```

Осталось настроить переменные c токенами, паролями и другими приватными данными на платформе GitHub Actions.

Перейдите в настройки репозитория — Settings, выберите на панели слева Secrets and Variables → Actions, нажмите New repository secret.

Сохраните переменные `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `SSH_KEY`, `SSH_PASSPHRASE`, `USER`, `HOST`, `TELEGRAM_TO` и `TELEGRAM_TOKEN` с необходимыми значениями: введите имя секрета и его значение, затем нажмите Add secret.

## Автор:
Ильдар Сабиров - [ildar-sabirov](https://github.com/ildar-sabirov)
