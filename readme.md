![Readme Art](https://raw.githubusercontent.com/codecasts/ambientum/master/ambientum-for-wordpress-cover.png)

# Ambientum for Wordpress

This is a fork from Ambientum to adapt the environment slightly for Wordpress development. Some of the differences are the inclusion of Wordpress specific tools and optimizations.

No more "it worked on my machine"!

## What is does?
Ambientum for Wordpress can help you doing some amazing things, the 3 main scenarios are listed above:

- **Run WordPress in Development.**
- **Run WordPress in Production (Continuous Integration included).** // TODO
- **Replace local dependencies with Docker commands.** // TODO

## What do I need to know before getting started?

Before staging with Ambientum for Wordpress, a few pieces of knowledge must be in place:

#### For replacing local commands:
- The basics of Docker and Containers.
- Being comfortable around the command line.

#### For running a development Wordpress environment:
- Know how to operate `docker-compose`.


## Notice
Docker compose shipped with Docker is usually very old.
Please have the latest version installed from Github at https://github.com/docker/compose/releases.

## Images
If you are already comfortable with the tools and have played around Ambientum for Wordpress, here are the set of images available for usage,
so you can start building your environment with the tools that you may want. // TODO

> PHP 7.2 is available but yet without xDebug support, considering this, it will not receive the latest tag until xDebug stable is compatible with PHP 7.2

|Repository                 | Images/Tags                   | Description                                        |
|---------------------------|-------------------------------|----------------------------------------------------|
| ambientum/**php**         | `7.2`, `latest`               | PHP v7.2 for command line and queues               |
|                           | `7.2-nginx`, `latest-nginx`   | PHP v7.2 with Nginx webserver                      |
|                           | `7.1`                         | PHP v7.1 for command line and queues               |
|                           | `7.1-nginx`, `latest-nginx`   | PHP v7.1 with Nginx webserver                      |
| ambientum/**node**        | `9`, `latest`, `current`      | Node.js v9.x                                       |
|                           | `8`, `lts`                    | Node.js v8.x                                       |

## Quick usage guide

### Replacing local commands:
One of the features of ambientum is allowing you to replace commands (normally painful to install) with their docker
based version.

There are a set of aliases that helps you with that task, and before proceeding you will need to install and
activate those aliases:

#### For Bash and ZSH Users:
```
curl -L https://github.com/codecasts/ambientum/raw/master/commands.bash -o ~/.ambientum_rc
source ~/.ambientum_rc
```

#### For Fish users:
```
curl -L https://github.com/codecasts/ambientum/raw/master/commands.fish -o ~/.ambientum_rc
source ~/.ambientum_rc
```

#### For Windows users (via Git Bash):

For others Windows consoles (like PowerShell or Cmder), check this [link](https://github.com/julihermes/ambientum-commands-for-windows).

```
curl https://github.com/codecasts/ambientum/raw/master/commands.git-bash -o ~/.ambientum_rc
source ~/.ambientum_rc
```

Once you have those commands activated, following the instructions above, you can start using them right away.

#### Notice: First execution may take some time since it will download the images

Everything Node.JS related can be executed by prefixing the `n` command. For example, let's say we need to install Gulp
```
n npm install -g gulp
n gulp --version
```

Everything PHP related can be executed by prefixing the `p` command. For example, lets say we want to run composer:

```
p composer global require some/package-here
```

Or even run php against a single file:
```
p php test.php
```

Another example would be starting a new Laravel project:

```
p composer create laravel/laravel my-project-name-here
```

### I do have a project, and I want to run it using docker
Well, that's the whole point of the project, the commands there was designed for quick usage of stand alone commands, so we have a great alternative when we have a project already, we can define a docker-compose.yml file that will expose and run the services we need.

> **Understanding the docker-compose compose tool is appreciated in order to use the following configuration files.**

#### Laravel docker-compose.yml


```yml
####
# ATENTION:
# Replace all occurences of sandbox with your project's name
####

# v2 syntax
version: '2'

# Named volumes
volumes:
  # Postgres Data
  sandbox-postgres-data:
    driver: local

  # MySQL Data
  sandbox-mysql-data:
    driver: local

  # Redis Data
  sandbox-redis-data:
    driver: local

services:
  # Postgres (10.3)
  postgres:
    image: postgres:10.3
    container_name: sandbox-postgres
    volumes:
      - sandbox-postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=sandbox
      - POSTGRES_DB=sandbox
      - POSTGRES_USER=sandbox

  # MySQL (5.7)
  mysql:
    image: mysql:5.7
    container_name: sandbox-mysql
    volumes:
      - sandbox-mysql-data:/var/lib/mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=sandbox
      - MYSQL_DATABASE=sandbox
      - MYSQL_USER=sandbox
      - MYSQL_PASSWORD=sandbox

  # Redis
  cache:
    image: redis:4.0
    container_name: sandbox-redis
    command: --appendonly yes
    volumes:
      - sandbox-redis-data:/data
    ports:
      - "6379:6379"

  # PHP (with Nginx)
  # you can change from nginx to apache, just change session 'image'
  app:
    image: ambientum/php:7.2-nginx
    container_name: sandbox-app
    volumes:
      - .:/var/www/app
    ports:
      - "80:8080"
    links:
      - postgres
      - mysql
      - cache

  # Laravel Queues
  queue:
    image: ambientum/php:7.2
    container_name: sandbox-queue
    command: php artisan queue:listen
    volumes:
      - .:/var/www/app
    links:
      - mysql
      - cache
```
