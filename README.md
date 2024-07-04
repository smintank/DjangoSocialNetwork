# Django Social Network

## Описание
Django Social Network - это социальная сеть на Django и React, где каждый зарегестрированый пользователь может добавить своих котиков для того что бы делиться их фотографиями и достижениями с другими пользователями.


## Установка и настройка бэкенд-приложения:

### 1. Перейдите в директорию `backend` проекта kittygram. Создайте виртуальное окружение и активируйте его.
```
python3 -m venv venv

source venv/bin/activate
```
### 2. Установите зависимости из файла requirements.txt в виртуальное окружение Django:
```
pip install -r requirements.txt
```
### 3. Выполните миграции и создайте суперюзера из директории с файлом manage.py:

```
# Примените миграции.
python3 manage.py migrate

# Создайте суперпользователя.
python3 manage.py createsuperuser
```
### 4. Отредактируйте файл `.env` в директории backend/kittygram_backend следующим образом:
```
# ~/kittygram_backend/.env

SECRET_KEY=your_secret_key
DEBUG=0
ALLOWED_HOSTS="xxx.xxx.xxx.xxx, 127.0.0.1, localhost, ваш_домен"
```

SECRET_KEY - это секретный ключ Django который можно сгенерировать [тут](https://djecrety.ir) или ввести свой.

DEBUG - это параметр режима отладки. `0` - режим отладки выключен (False), `1` - включен (True). Не меняйте если не собираетесь заниматься отладкой бекэнда.

ALLOWED_HOSTS - разрешенные адреса для доступа к бекенду. Для добавления своих IP адресов или присвоенных им доманных имен, добавьте их к существующим локальным адресам `127.0.0.1` и `localhost`.

Пример отредактированного файла:
```
# ~/kittygram_backend/.env

SECRET_KEY=django-insecure-cg6*%6d432vgt#4!r3*$vmxm4)abgjw8mo!4y-q*uq1!4$-89$
DEBUG=0
ALLOWED_HOSTS="123.456.78.90, 127.0.0.1, localhost, yourhostname.com"
```

### 5. Соберите статику бэкенд-приложения:
```
python3 manage.py collectstatic
```
### 6. Скопируйте директорию static_backend/ в директорию /var/www/kittygram/:
```
sudo cp -r infra_sprint1/backend/static_backend /var/www/kittygram
```

### 7. Настройте WSGI Gunicorn:
Отредактируйте файл gunicorn_kittygram.service в директории `infra` следующим образом заменив `<username>` на имя своего пользователя:
```
[Unit]
Description=gunicorn daemon 
After=network.target

[Service]
User=<username> 
WorkingDirectory=/home/<username>/infra_sprint1/backend/
ExecStart=/home/<username>/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target
```

### 8. Скопируйте файл `gunicorn_kittygram.service` в дерикторию /etc/systemd/system/:
```
sudo cp infra_sprint1/infra/gunicorn_kittygram.service /etc/systemd/system/
```

### 9. Запустите Gunicorn и добавьте его в автозагрузку:
```
sudo systemctl start gunicorn_kittygram.service

sudo systemctl enable gunicorn_kittygram.service
```



## Установка и настройка фронтенд-приложения
### 1. Находясь в директории с фронтенд-приложением, установите зависимости для него:
```
npm i
```

### 2. Из директории с фронтенд-приложением выполните команду:
```
npm run build
```

### 3. Скопируйте статику фронтенд-приложения в директорию по умолчанию:
```
sudo cp -r infra_sprint1/frontend/build/. /var/www/kittygram/
```



## Установка и настройка веб- и прокси-сервера Nginx
### 1. Если Nginx ещё не установлен на удалённый сервер, установите его:
```
sudo apt install nginx -y
```

### 2. Запустите Nginx командой:
```
sudo systemctl start nginx
```

### 3. Отредактируйте файл `default` в директории `/etc/nginx/sites-enabled/default` добавив новый сервер:
Впишите свой ip и доменное имя в параметр `server_name`
```
server {
    server_name 123.456.78.90 yourhostname.com;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
        client_max_body_size 20M;
    }

    location /media/ {
        root /var/www/kittygram/;
    }

    location / {
        root   /var/www/kittygram;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
}
```

### 4. Сохраните изменения в файле, закройте его и проверьте на корректность:
```
nginx -t
```

### 5. Перезагрузите конфигурацию Nginx:
```
sudo systemctl reload nginx
```

### 6 Активируйте разрешение принимать запросы только на порты 80, 443 и 22:
```
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
```

### 7. Включите файрвол:
```
sudo ufw enable
```
В терминале выведется запрос на подтверждение операции. Введите `y` и нажмите `Enter`. 

### 8. Проверьте работу файрвола:
```
sudo ufw status
```

## Получение и настройка SSL-сертификата

### 1. Находясь на сервере, установите certbot, если он ещё не установлен:
```
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 2. Запустите certbot и получите SSL-сертификат. Для этого выполните команду:
```
sudo certbot --nginx
```

Далее система попросит вас указать электронную почту и ответить на несколько вопро- сов. Сделайте это.
Следующим шагом укажите имена, для которых вы хотели бы активировать HTTPS

### 3. Проверьте конфигурацию Nginx, и если всё в порядке, перезагрузите её.
```
sudo systemctl reload nginx
```

