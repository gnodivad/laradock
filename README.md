# Laradock

## Setup Local Development

First, we should clone this repo to the directory that is same level with our local repo.

```
git clone https://github.com/gnodivad/laradock.git
```

We will reuse code from [laradock](http://laradock.io/) repo but using our own config.

```
git submodule update --init
```

It will create a directory called `laradock` inside the current directory.

Inside that directory, you will see several development tools that we are familiar such as `php-fpm`, `mysql`, `php-worker`, `nginx`, `workspace` and others. Inside each of them will contains a `Dockerfile` that used by laradock to build the images required.

From here, we will start to copy our own config into laradock config.

```
cp config/env-local laradock/.env
cp config/mysql/docker-entrypoint-initdb.d/createdb.sql laradock/mysql/docker-entrypoint-initdb.d/createdb.sql
```

To browse the website that you built locally in any internet browser, you should copy any `[sites].conf` into `laradock/nginx/sites`. `[sites]` refer to any domain name that you want for local development. For example `[DOMAIN].local`, then you should have a file name `[DOMAIN].local.conf` inside the folder.

This file normally can be found at `ecs/nginx/sites` in each of the project repo. If dont have, you can find the developer that incharge of this repo or get from the other colleague.

Following is the template for `[DOMAIN].local`.

```
server {
    listen 80;
    listen [::]:80;

    server_name [DOMAIN].local;
    root /var/www/[REPO]/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_pass php-upstream;
        fastcgi_index index.php;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    location /.well-known/acme-challenge/ {
        root /var/www/letsencrypt/;
        log_not_found off;
    }

    error_log /var/log/nginx/laravel_error.log;
    access_log /var/log/nginx/laravel_access.log;
}
```

And don't forget add your local domain name to `etc/hosts`, such as

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
#
127.0.0.1       [DOMAIN].local
```

Once we finished that, we can run command below to build and run the laradock.

```
cd laradock
docker-compose up -d --build nginx mysql
```

For who doing the backend development, we often find out that we want to execute certain script or `php artisan` on the container, just like we ssh inside any production server.

To do that, we should

```
docker-compose exec workspace bash
```
