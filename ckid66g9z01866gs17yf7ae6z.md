## How To Setup A PHP Laravel Development Container With Docker

Recently, I've learned to use Docker for my project development environment setup.

A Docker container allows you to contain all the necessary development software and environment needs into one complete package for easy distribution.

By using a development container, anyone in your development team will be able to clone and have the exact same setup ready for development with just one command.

In this article, we will walk through the steps in creating a Docker container for Laravel Development that will include an **NGINX** webserver, **PHP** with extra extensions, as well as **MySQL** database.

In addition, we will throw in ** [adminer](https://www.adminer.org/) ** for easy database management.

## 🛠 Prerequisites

### Docker

You need to install Docker on your machine. If you are new to Docker, I recommend you to download Docker Desktop from the following site:

- 🔗 https://www.docker.com/products/docker-desktop

### Git

Make sure your machine has Git installed.

### Project Folder

Before we begin, please create a new folder for your project. All files will be created in this folder.

We will call this folder `my-project`.

## 1. Clone or setup a Laravel project

Make sure you are in the `my-project` directory.

```
~$ cd my-project
```

Clone an existing Laravel codebase or set up a new Laravel project.

```
my-project$ git clone https://github.com/laravel/laravel.git laravel
```

Note that it clones the project files into a folder named `laravel`.

## 2. Install Composer

Composer is usually required when developing with Laravel.

Instead of forcing your developers to install Composer globally on their machine, we can use Docker’s composer image to mount the directories that are needed by your Laravel project.

Move into the `./laravel` directory and run the command: 

```
my-project$ cd laravel

laravel$ docker run --rm -v $PWD:/app composer install
```

The `-v` and `--rm` flags are used to create a short-lived container that will be bind-mounted to the `/laravel` directory before being removed. This ensures that the `/vendor` folder that Composer creates inside the container is copied to your current directory.

## 3. Customize a new PHP Docker image

To be able to run Laravel and Composer, and to connect with MySQL database, we will need to install a few PHP extensions.

We will refer to the official Laravel's requirements here: 

- 🔗 https://laravel.com/docs/8.x#server-requirements

The official Docker PHP image provides only the bare minimum for most PHP applications. Therefore, to fulfil our requirements, we will have to build our own Docker image.

Move back to `my-project` folder and create a new folder `php-fpm`:

```
laravel$ cd ..

my-project$ mkdir php-fpm
```

Create a new file with the filename `Dockerfile` inside the new `./php-fpm` folder:

```Dockerfile
# Use the PHP-FPM 7.4 image
FROM php:7.4-fpm

# Install the necessary software and libraries
RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libzip-dev \
    git \
    zip \
    unzip \
    curl

# Install the required PHP extensions
RUN docker-php-ext-install bcmath mysqli pdo_mysql zip
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install -j$(nproc) gd

# Install xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
```

When saving the file, make sure the `Dockerfile` filename does not contain any file extension like `.txt`.

## 4. Setup NGINX site configuration

In order for our Laravel application to be served by NGINX webserver, we need to create an NGINX site `.conf` file.

In `\my-project` folder, create a new folder `nginx` and a subdirectory `conf.d` within the new folder:

```
my-project$ mkdir -p nginx/conf.d
```

The `-p` flag will allow the creation of subdirectories when the parent directory is not present.

Create a new configuration file `site.conf` in `./nginx/conf.d` directory:

```
server {
    listen      80;
    server_name localhost;
    root        /var/www/laravel/public;

    add_header  X-Frame-Options "SAMEORIGIN";
    add_header  X-XSS-Protection "1; mode=block";
    add_header  X-Content-Type-Options "nosniff";

    index       index.php;

    charset     utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    # pass the PHP scripts to FastCGI server listening on [server]:9000
    location ~ \.php$ {
        fastcgi_pass    app:9000;
        fastcgi_param   SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include         fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

The above configuration is based upon the recommended site configuration by Laravel here:

- 🔗 https://laravel.com/docs/8.x/deployment#nginx

Please note the following:

i) The `root        /var/www/laravel/public;` is where we will store our Laravel application codebase.

ii) `fastcgi_pass    app:9000;` is the Docker network DNS name for the PHP-FPM container, which we will see in the next step.

## 5. Create a `docker-compose.yml` configuration

Instead of running each container service one by one, a `docker-compose.yml` file will define each container image and setup details that can be called by Docker Compose with a single command.

Let's start by defining the services we need. Create a new `docker-compose.yml` configuration file in `/my-project` folder:

```yml
version: "3.7"

services: 
  web:
    image: nginx:latest
    container_name: webserver

  app:
    build: 
      context: ./php-fpm
      dockerfile: Dockerfile
    image: php-fpm-laravel
    container_name: app
    working_dir: /var/www/laravel

  db:
    image: mysql:8.0
    container_name: db

  adminer:
    image: adminer
    container_name: adminer
```

Here, we have defined four services in our configuration. As you can guess from the `image:` field,  each service will serve a different purpose.

This is the recommended practice when setting up container services: Each container should do one thing and do it well.

A few things to take note of the configuration settings:

i) Under the **app** service, we are building a custom PHP Docker image using the build commands in `./php-fpm/Dockerfile` defined previously.

ii) `container_name` is important for containers to identify each other within a Docker network.

iii) The `working_dir: /var/www/laravel` under **app** service will be the location where the Laravel application codebase is stored in the container. This is important when we need to execute PHP Artisan & Composer commands in the container later.

## 6. Mapping Access Port

To access the Laravel **app** & **adminer** from our localhost machine, we need to define each port that maps to the **web** & **adminer** service container.

i) In the `docker-compose.yml` file, add the following port configuration under the **web** service:

```yml
...
services: 
  web:
    image: nginx:latest
    container_name: webserver
    ports:
      - "8000:80"
...
```

This means that we will access our Laravel app via http://localhost:8000

ii) Add the following port configuration under the **adminer** service:

```yml
...
  adminer:
    image: nginx:latest
    container_name: webserver
    ports:
      - "8888:8080"
```

This will allow us to manage our MySQL database using **adminer** from http://localhost:8888

## 7. Set a MySQL Database & Password

Let's set a database and password for MySQL database container.

i) In `docker-compose.yml` file, add the following `environment:` configuration under the **db** service:

```yml
...
db:
    image: mysql:8.0
    container_name: db
    environment: 
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: secret
...
```

ii) Make a copy of `./laravel/.env.example` file and name the new file `./laravel/.env`. Then, open the newly copied file `./laravel/.env` and edit it with the following MySQL Database information:

```
...
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=secret
...
```

Note that the `DB_HOST=db` must be equivalent to the **db** service `container_name: db` from `docker-compose.yml` file.

> ❗️It's not a good practice to use the `root` account for database access, but it's acceptable for development purposes in a container deployed on your local machine.

## 8. Setting up Docker Volumes

At this point, your project directory structure should look like this:
```
my-project
    \_ laravel
        \_ app
        \_ ...
        \_ public
        \_ vendor
        \_ .env
        \_ ...
    \_ nginx
        \_ conf.d
            \_ site.conf
    \_ php-fpm
        \_ Dockerfile
    \_ docker-compose.yml
```

Next, we are going to create Docker Volumes for several purposes:

i) First, the Laravel application codebase in our local machine will be mounted to the containers so that any code changes made to our local files will be reflected in the containers without having to reload our containers.

This will speed up the work of your development process.

ii) Second, to mount the NGINX site configuration from **Step 4** before that will override the default site configuration in the NGINX container.

This exposes the site configurations to your development team so that everyone understands how the webserver is being setup.

This also allows us to change our site configuration conveniently at any time without having to do it in the container.

iii) Thirdly, to persist the data stored in MySQL Database so that all data created inside the container database is retained even when the containers are being brought down.

This is useful when we need to develop a huge feature that spans several days and needed the data to persists for development continuation purposes.

Here's the complete `docker-compose.yml` configuration after adding the volume mounts for each container:

```yaml
version: "3.7"

services: 
  web:
    image: nginx:latest
    container_name: webserver
    ports:
      - "8000:80"
    volumes: 
      - ./laravel:/var/www/laravel
      - ./nginx/conf.d/:/etc/nginx/conf.d/

  app:
    build: 
      context: ./php-fpm
      dockerfile: Dockerfile
    image: php-fpm-laravel
    container_name: app
    working_dir: /var/www/laravel
    volumes: 
      - ./laravel:/var/www/laravel

  db:
    image: mysql:8.0
    container_name: db
    environment: 
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - laravel-mysql-data:/var/lib/mysql

  adminer:
    image: adminer
    container_name: adminer
    restart: always
    ports:
      - "8888:8080"

volumes: 
  laravel-mysql-data:
```

Note the last line `laravel-mysql-data:` is where the database data will be stored persistently in a Docker volume.

## 9. Run Docker Compose

All the necessary files and configuration is in place. Now let's spin up our containers using Docker Compose.

Make sure you are in the `my-project` directory where the `docker-compose.yml` file is located and run the following command:

```
my-project$ docker-compose up -d
```

The `-d` flag is to instruct Docker to run the containers in detached mode.

Please be patient as the build process will take some time to complete. Once it's completed, you will see the following response in your terminal:

```
Creating webserver ... done
Creating db        ... done
Creating adminer   ... done
Creating app       ... done
my-project$
```

If you have installed Docker Desktop, open the Dashboard to view your new containers:

![Screenshot 2020-12-06 at 5.08.44 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1607245791680/ZUyg2sAbM.png)

## 10. Execute PHP Artisan & Composer commands

Normally, you do not include the `.env` file and `./vendor/` folder in your code repository. Therefore, you need to include in your repository `README` file with the following instructions for your development team on how to run `php artisan` & `composer` commands in the containers.

So, before we can access the Laravel application from the browser, we need to run a few commands to set up the necessary data required by Laravel.

i) First, let's generate the application encryption key:

```
$ docker exec app php artisan key:generate
```

Let's break down the command:

- `docker exec`: the Docker command to execute commands in a container
- `app`: the service `container_name` defined previously in `docker-compose.yml` file
- `php artisan key:generate`: the actual command you intended to run in the container

You will see `Application key set successfully.` returned for the above successful command.

ii) Next, let's install all composer's dependencies:

```
$ docker exec app composer install
```

iii) To clean up composer cache:

```
$ docker exec app composer dump-autoload
```

Now, you can access your application via http://localhost:8000 on a web browser where you will see your Laravel application being loaded.

![Screenshot 2020-12-06 at 8.41.16 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1607258502511/9vdray-gL.png)

## 11. (Optional) Database migration and **adminer** access

Next, I'll show you how to install new composer dependencies, run database migration, and view them in **adminer**.

Let's assume your application needs to use the Laravel Passport to handle authentication. You can install the dependencies with the following command:

```
$ docker exec app composer require laravel/passport
```

To create the tables necessary for Laravel Passport, you may run the following command:

```
$ docker exec app php artisan migrate
```

And lastly, to create the encryption keys needed to generate secure access tokens in Laravel Passport:

```
$ docker exec app php artisan passport:install
```

Now, let's inspect the tables and data in MySQL database via **adminer** by going to http://localhost:8888/ using your web browser:

![Screenshot 2020-12-06 at 5.58.08 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1607248739964/AN-h99fDU.png)

Login with the following credentials:

- Server: *db*
- Username: *root*
- Password: *secret*
- Database: *laravel*

You should see the tables and their data created from the migration and installation processes earlier:

![Screenshot 2020-12-06 at 6.25.32 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1607250394568/7U2fJwGti.png)

## 12. Shutting down containers

Finally, we can shut down our containers once we are done working on our project.

```
$ docker-compose down
```

You will see the following response messages in your terminal after the process has completed:

```
Stopping webserver ... done
Stopping app       ... done
Stopping adminer   ... done
Stopping db        ... done
Removing webserver ... done
Removing app       ... done
Removing adminer   ... done
Removing db        ... done
Removing network my-project_default
my-project$
```

Alternatively, take a look at the Docker Desktop Dashboard. You will find that your project containers are gone from the list.

<hr>

## 🏁 Summary

🎉 That's all for today! We have come to an end to the walkthrough of setting up a Laravel Development Container using Docker.

With this method, we can say goodbye to setting up a different development environment manually on your machine for every project. Your development team will no longer have to spend the whole day setting up their machine before they can start working on the project. Not to mention having to make sure all software and environments must match the project system requirements.

Hopefully, by learning how to use containers as a development environment, you will no longer hear your developers saying "It works on my machine!". 😉

🙏 Thank you for reading! If you find this article useful, please share it with your followers.

💬 If you have any questions, feel free to leave a comment below.

## 📚 References

- https://www.docker.com/101-tutorial
- https://www.digitalocean.com/community/tutorials/how-to-set-up-laravel-nginx-and-mysql-with-docker-compose
- https://hub.docker.com/_/composer
- https://hub.docker.com/_/php
- https://laravel.com/docs/8.x/installation#server-requirements
- https://laravel.com/docs/8.x/deployment#nginx