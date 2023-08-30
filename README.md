# Kittygram
## Описание 
Проект Kittygram c backend на Django, c контейниразацией Docker и CI/CD. Благодаря этому проекту, можно на сайт [Kittygram](https://kitty-gramm.ddns.net/) добавлять карточки котиков, с их описанием (цвет, год рождения, фотография) и указывать, чем они отличились.
## Технологии 
- Python 3
- Django
- Nginx
- Gunicorn
- React
- Django Rest Framework
- Certbot
- Docker
- PosgreSQL
- GitHub Actions

## Инструкция по запуску

1. Клонирование проекта с GitHub на локальный компьютер
```
git@github.com:ваш_аккаунт/kittygram_final.git
```
2. Создайте файл .env и заполните его. Переменные для работы проекта перечислены в файле .env.example, находящемся в корневой директории проекта.
### Создание Docker-образов и загрузка на Docker Hub
1. В терминале в корне проекта kittygram_final последовательно выполните следующие команды; замените username на ваш логин на Docker Hub.
```
cd frontend
docker build -t username/kittygram_frontend .
cd ../backend  
docker build -t username/kittygram_backend .
cd ../nginx 
docker build -t username/kittygram_gateway . 
```
2. Загрузите образы на Docker Hub
```
docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway 
```
### Деплой на сервер
1. В терминале Git Bash введите:
```
ssh -i путь_до_SSH_ключа/название_файла_с_SSH_ключом_без_расширения login@ip
```
2. Создайте директорию kittygram
```
mkdir kittygram
```
3. Установите Docker Compose на сервер:
```
cd
sudo apt update
sudo apt install curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose
```
4. Скопируйте файлы docker-compose.production.yml и .env в директорию kittygram на сервер. Для этого зайдите на своём компьютере в директорию kittygram_final и выполните в Git Bash команду копирования:
```
scp -i <path_to_SSH>/<SSH_name> docker-compose.production.yml \
    <username>@<server_ip>:/home/<username>/kittygram/docker-compose.production.yml 
```
```
scp -i <path_to_SSH>/<SSH_name> .env \
    <username>@<server_ip>:/home/<username>/kittygram/.env 
```
где:
* path_to_SSH — путь к файлу с SSH-ключом;
* SSH_name — имя файла с SSH-ключом (без расширения);
* username — ваше имя пользователя на сервере;
* server_ip — IP вашего сервера.
5. Для запуска Docker Compose в режиме демона выполните команду:
```
sudo docker compose -f docker-compose.production.yml up -d
```
6. Выполните миграции из директории kittygram
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
```
7. Соберите статику бэкэнда
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
```
8. Скопируйте статику в директорию /backend_static/static/
```
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/ 
```
9. На сервере в редакторе nano откройте конфиг Nginx
```
nano /etc/nginx/sites-enabled/default
```
10. Измените настройки location в секции server
```
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}
```
11. Выполните команду проверки конфигурации:
```
sudo nginx -t 
```
12. Перезапустите Nginx:
```
sudo service nginx reload 
```
### Автоматизация деплоя CI/CD
В папке проекта на локальном компьютере есть файл workflow main.yml
```
kittygram_final/.github/workflows/main.yml
```
При каждом изменении проекта и его загрузке на GitHub, на сервере будет автоматически обноваляться код, перед этим проходя проверку тестами. При положительном результате вам придёт сообщение в телеграм о том, что деплой прошел успешно.
Для использования данного файла необходимо на сайте [GitHub](https://github.com/) перейти в **Settings** проекта -> **Secrets and variables** -> **Actions** и поменять следующие значения на свои
```
DOCKER_USERNAME - имя пользователя в DockerHub
DOCKER_PASSWORD - пароль пользователя в DockerHub
HOST            - IP-адрес сервера
USER            - имя пользователя на сервере
SSH_KEY         - содержимое приватного SSH-ключа
SSH_PASSPHRASE  - PASSPHRASE для SSH-ключа
TELEGRAM_TO     - ID вашего телеграм-аккаунта
TELEGRAM_TOKEN  - токен вашего бота 
```

### Автор 
[Виктор Шустров](https://github.com/shustrov19)

### Контакты
email: shustrov19@gmail.com
