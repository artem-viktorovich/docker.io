<a href="https://www.youtube.com/watch?v=fOQAeP3qkP0&list=PLd2_Os8Cj3t9Ert8mBlNl1UqwllyP1Tm_">Курсы</a>
<span style="color: red;">docker image</span> - docker образ
<span style="color: red;">docker hub</span> - хранилище образов
<span style="color: red;">docker container</span> - docker контейнер
<span style="color: red;">поднять контейнер</span> - запустить
<span style='color: red;'>docker exec -it \имя контейнера\ bash</span> - вход в наш контейнер для выполнения команд внутри него
(для определённого контейнера выполнять - composer install)
<span style="color: red;">docker-compose up -d</span> - запуск контейнера
<span style="color: red;">docker-compose down</span> - остановка контейнера
<span style="color: red;">docker exec \имя контейнера\ nginx -s reload</span> - перезапуск сервиса внутри контейнера
<span style="color: red;">docker builder prune</span> - очистка кэша докера



<h2>Настройки для docker-compose.yml</h2>

Настройка файла

```
version: '3'

  

services:

  nginx: - \\ сервис

    image: nginx:latest - \\ имя образа

    restart: always

    volumes:

      - ./:/var/www - \\ перенос папок

      - ./_docker/nginx/conf.d:/etc/nginx/conf.d - \\ перенос папок

    ports:

      - 80:80 - \\ номер порта

    depends_on:

      - app

    container_name: project_nginx - \\ имя контейнера

  

  app:

    restart: always

    build:

      context: .

      dockerfile: _docker/app/Dockerfile

    volumes:

      - ./:/var/www - \\ открытие портала

    container_name: project_app - \\ имя контейнера

  

  db:

        image: mysql:8.2 - \\ версия базы данных

        restart: always

        ports:

          - 8101:3306 - \\ порт обращения к контейнеру

        command: mysqld --character-set-server=utf8 --collation-server=utf8_unicode_ci

        volumes:

          - ./tmp/db:/var/lib/mysql

        environment:

          MYSQL_DATABASE: database - \\ имя базы данных

          MYSQL_ROOT_PASSWORD: root - \\ права
```


<h2>nginx.conf</h2>

Настройка файла

```
server {

  

    index index.html index.php;

    error_log  /var/log/nginx/error.log;

    access_log /var/log/nginx/access.log;

    root /var/www/public;

  

    location / {

        try_files $uri /index.php; - // открытие файла

        # kill cache

        add_header Last-Modified $date_gmt;

        add_header Cache-Control 'no-store, no-cache';

        if_modified_since off;

        expires off;

        etag off;

    }

  

    location ~ \.php$ {

        try_files $uri =404;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        fastcgi_pass app:9000;

        fastcgi_index index.php;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        fastcgi_param PATH_INFO $fastcgi_path_info;

    }

  

}
```

<h2>php.ini</h2>
Настройка файла

```
cgi.fix_pathinfo=0

max_execution_time = 1000

max_input_time = 1000

memory_limit=4G
```

<h2 style="color:red;">Dockerfile</h2>

Настройка файла

```
FROM php:8.2-fpm

  

RUN apt-get update && apt-get install -y \

      apt-utils \

      libpq-dev \

      libpng-dev \

      libzip-dev \

      zip unzip \

      git && \

      docker-php-ext-install pdo_mysql && \

      docker-php-ext-install bcmath && \

      docker-php-ext-install gd && \

      docker-php-ext-install zip && \

      apt-get clean && \

      rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

  

COPY ./_docker/app/php.ini /usr/local/etc/php/conf.d/php.ini

  

# Install composer

ENV COMPOSER_ALLOW_SUPERUSER=1

RUN curl -sS https://getcomposer.org/installer | php -- \

    --filename=composer \

    --install-dir=/usr/local/bin

  

WORKDIR /var/www
```
