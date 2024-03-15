# <a name="laravel-docker-compose-mysql-phpmyadmin">Laravel Docker, Compose, MySQL, phpMyAdmin</a>

<div align="center" >
<!-- link to project -->
    <a href='-URL TO DEMO GOES HERE-'>
    <!-- link to local image -->
        <img src="https://picsum.photos/500/300" alt="" height="100%"/>
    </a>
</div>

<br />

<div align="center">
    <img src="https://img.shields.io/badge/-Laravel-black?style=for-the-badge&logoColor=white&logo=laravel&color=f5382e" alt="laravel" />
    <img src="https://img.shields.io/badge/-Docker-black?style=for-the-badge&logoColor=white&logo=docker&color=2496ED" alt="docker" />
    <img src="https://img.shields.io/badge/-MySQL-black?style=for-the-badge&logoColor=white&logo=mysql&color=e57b16" alt="mysql" />
</div>
<br />
<div align="center">
    <img src="https://img.shields.io/badge/-phpMyAdmin-black?style=for-the-badge&logoColor=white&logo=phpmyadmin&color=717cb2" alt="phpmyadmin" />
    <img src="https://img.shields.io/badge/-nginx-black?style=for-the-badge&logoColor=white&logo=nginx&color=009639" alt="nginx" />
    <img src="https://img.shields.io/badge/-npm-black?style=for-the-badge&logoColor=white&logo=npm&color=fa5c00" alt="npm" />
</div>

<div align="center">
  <h3 align="center">Laravel Docker Container</h3>
  <p>
     Setting Up a Laravel 10 Development Environment with Docker
     written by <b>Vitalii Shloda</b> on Medium. This article was used as a reference along with <b> Docker Hub</b> documentation to bring up to date.
  </p>
  <p align="center">
    <a href="https://github.com/vshloda/docker-laravel" target="_blank">
    <img src="https://img.shields.io/badge/-Repo-lightgrey?style=plastic&labelColor=black&for-the-badge&logo=github"/>
    </a>
    <a href="https://vshloda.medium.com/setting-up-a-laravel-10-development-environment-with-docker-3977a292c8dd" target="_blank">
    <img src="https://img.shields.io/badge/-Medium-blue?style=plastic&labelColor=black&for-the-badge&logo=googlechrome&logoColor=white&"/>
    </a>
  </p>
</div>

<br />

## <a name="table-of-contents">Table of Contents</a>

- [Laravel Docker, Compose, MySQL, phpMyAdmin](#laravel-docker-compose-mysql-phpmyadmin)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step 1: Setup PHP Docker Environment](#step-1-setup-php-docker-environment)
    - [Code Snippet](#code-snippet)
  - [Step 2: Create a Docker Compose File](#step-2-create-a-docker-compose-file)
    - [Code Snippet](#code-snippet-1)
  - [Step 3: Configure Nginx](#step-3-configure-nginx)
    - [Code Snippet](#code-snippet-2)
  - [Step 4: Build and Run the Docker Containers](#step-4-build-and-run-the-docker-containers)
  - [Step 5: Create a New Laravel 10 Project](#step-5-create-a-new-laravel-10-project)

<br />

## <a name="introduction">Introduction</a>

With the growing trend of containerization in web development, Docker has become an indispensable tool for developers. Docker allows for seamless management of dependencies and environments, enabling developers to focus on coding. In this article, we’ll guide you through setting up a Laravel 10 development environment using Docker.

## <a name="prerequisites">Prerequisites</a>

Before starting, make sure the following are installed on your system:

- Docker: Visit Docker’s official website to download and install Docker Desktop.
- Docker Compose: Usually included with Docker Desktop, install separately if it’s not.
- Composer: PHP’s dependency manager, necessary to create a new Laravel project.

## <a name="step-1-setup-php-docker-environment">Step 1: Setup PHP Docker Environment</a>

In the root of your project, create a file named `Dockerfile`. This file defines the configuration for our Docker container. Add the following content to your `Dockerfile`:

### <a name="code-snippets">Code Snippet</a>

<details>
<summary><code>laravel Dockerfile</code></summary>

```dockerfile
  FROM php:8.0-fpm-alpine

  ADD ./php/www.conf /usr/local/etc/php-fpm.d/www.conf

  RUN addgroup -g 1000 laravel && \
      adduser -G laravel -g laravel -s /bin/sh -D laravel && \
      mkdir -p /var/www/html && \
      ADD ./src/ /var/www/html && \
      docker-php-ext-install pdo pdo_mysql && \
      chown -R laravel:laravel /var/www/html

```

</details>

<br />

## <a name="step-2-create-a-docker-compose-file">Step 2: Create a Docker Compose File</a>

In the root directory of your Laravel project, create a `docker-compose.yml` file. This file allows us to define and manage multi-container Docker applications. Insert the following content:

### <a name="code-snippets">Code Snippet</a>

<details>
<summary><code>laravel-docker/compose.yaml</code></summary>

```yaml
version: "3.9"

networks:
  laravel:
    name: laravel

services:
  nginx:
    build:
      context: .
      dockerfile: nginx.dockerfile
    depends_on:
      - php
      - mysql
    container_name: nginx
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./src:/var/www/html
    networks:
      - laravel

  php:
    build:
      context: .
      dockerfile: php.dockerfile
    container_name: php
    volumes:
      - ./src:/var/www/html
    networks:
      - laravel

  mysql:
    image: mysql:8.0.27
    container_name: mysql
    ports:
      - 3306:3306
    volumes:
      - ./mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: laraveldb
      MYSQL_USER: laravel
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: secret
    networks:
      - laravel

  composer:
    image: composer:latest
    container_name: composer
    volumes:
      - ./src:/var/www/html
    working_dir: /var/www/html
    networks:
      - laravel

  artisan:
    build:
      context: .
      dockerfile: php.dockerfile
    container_name: artisan
    volumes:
      - ./src:/var/www/html
    working_dir: /var/www/html
    entrypoint: ["php", "artisan"]
    networks:
      - laravel

  npm:
    image: node:current-alpine
    container_name: npm
    volumes:
      - ./src:/var/www/html
    working_dir: /var/www/html
    entrypoint: ["npm"]
    networks:
      - laravel
```

</details>

<br />

This `docker-compose.yml` file creates two services: one for PHP and one for Nginx. It also defines a Docker network that the two services use to communicate.

## <a name="step-3-configure-nginx">Step 3: Configure Nginx</a>

Create an `nginx` directory at the root of your Laravel project and within it, create a `conf.d` directory. Inside `conf.d`, create an `app.conf` file and insert the following Nginx server configuration:

### <a name="code-snippets">Code Snippet</a>

<details>
<summary><code>nginx</code></summary>

```nginx
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri = 404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}

```

</details />

## <a name="step-4-build-and-run-the-docker-containers">Step 4: Build and Run the Docker Containers</a>

Now, we’re ready to build and run our Docker containers. From the root directory of your Laravel project, run the following command:

```bash
docker-compose up -d --build
```

## <a name="step-5-create-a-new-laravel-10-project">Step 5: Create a New Laravel 10 Project</a>

We need to create a new Laravel 10 project. Navigate to the directory where you wish to create your project and run the following command:

<br />

```bash
docker-compose run --rm composer create-project laravel/laravel .
```

Congratulations! You’ve now successfully set up a Laravel 10 development environment with Docker and PHP 8.2. Your Laravel application is now accessible at [http://localhost:80](http://localhost:80).
