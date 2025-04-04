# What is Monica?

Monica is a great open source personal relationship management system to organize the interactions with your loved ones.

%%LOGO%%

## How to use this image

There are two versions of the image you may choose from.

The `apache` tag contains a full Monica installation with an apache webserver. This points to the default `latest` tag too.

The `fpm` tag contains a fastCGI-Process that serves the web pages. This image should be combined with a webserver used as a proxy, like apache or nginx.

### Using the apache image

This image contains a webserver that exposes port 80. Run the container with:

```console
docker run --name some-%%REPO%% -d -p 8080:80 %%IMAGE%%
```

### Using the fpm image

This image serves a fastCGI server that exposes port 9000. You may need an additional web server that can proxy requests to the fpm port 9000 of the container. Run this container with:

```console
docker run --name some-%%REPO%% -d -p 9000:9000 %%IMAGE%%:fpm
```

### Using an external database

You'll need to setup an external database. Monica currently support MySQL/MariaDB database. You can also link a database container, e. g. `--link my-mysql:db`, and then use `db` as the database host on setup. More info is in the Docker Compose section.

### Persistent data storage

To have a persistent storage for your datas, you may want to create volumes for your db, and for monica you will have to save the `/var/www/html/storage` directory.

Run a container with this named volume:

```console
docker run -d \
        -v monica_data:/var/www/html/storage \
        %%IMAGE%%
```

### Run commands inside the container

Like every Laravel application, the `php artisan` command is very usefull for Monica. To run a command inside the container, run

```console
docker exec CONTAINER_ID php artisan COMMAND
```

Or for Docker Compose:

```console
docker compose exec %%REPO%% php artisan COMMAND
```

where `%%REPO%%` is the name of the service in your `compose.yaml` file.

## Configuration using environment variables

The Monica image will use environment variables to setup the application. See [Monica documentation](https://github.com/monicahq/monica/blob/4.x/.env.example) for common used variables you should setup.

## Running the image with Docker Compose

See some examples of Docker Compose possibilities in the [example section](%%GITHUB-REPO%%/blob/main/.examples).

---

### Apache version

This version will use the apache image and add a mysql container. The volumes are set to keep your data persistent. This setup provides **no ssl encryption** and is intended to run behind a proxy.

Make sure to pass in values for `APP_KEY` variable before you run this setup.

1.	Create a `compose.yaml` file

	```yaml
	services:
	  app:
	    image: monica
	    depends_on:
	      - db
	    ports:
	      - 8080:80
	    environment:
	      - APP_KEY= # Generate with `echo -n 'base64:'; openssl rand -base64 32`
	      - DB_HOST=db
	      - DB_USERNAME=monica
	      - DB_PASSWORD=secret
	    volumes:
	      - data:/var/www/html/storage
	    restart: always

	  db:
	    image: mariadb:11
	    environment:
	      - MYSQL_RANDOM_ROOT_PASSWORD=true
	      - MYSQL_DATABASE=monica
	      - MYSQL_USER=monica
	      - MYSQL_PASSWORD=secret
	    volumes:
	      - mysql:/var/lib/mysql
	    restart: always

	volumes:
	  data:
	    name: data
	  mysql:
	    name: mysql
	```

2.	Set a value for `APP_KEY` variable before you run this setup. It should be a random 32-character string. You can for instance copy and paste the output of `echo -n 'base64:'; openssl rand -base64 32`:

3.	Run

	```console
	docker compose up -d
	```

	Wait until all migrations are done and then access Monica at http://localhost:8080/ from your host system. If this looks ok, add your first user account.

4.	Run this command once:

	```console
	docker compose exec app php artisan setup:production
	```

### FPM version

When using FPM image, you will need another container with a webserver to proxy http requests. In this example we use nginx with a basic container to do this.

1.	Download `nginx.conf` and `Dockerfile` file for nginx image. An example can be found on the [`example section`](%%GITHUB-REPO%%/blob/main/.examples/full/fpm/web/)

	```sh
	mkdir web
	curl -sSL https://raw.githubusercontent.com/monicahq/docker/main/.examples/full/web/nginx.conf -o web/nginx.conf
	curl -sSL https://raw.githubusercontent.com/monicahq/docker/main/.examples/full/web/Dockerfile -o web/Dockerfile
	```

	The `web` container image should be pre-build before each deploy with: `docker compose build`.

2.	Create a `compose.yaml` file

	```yaml
	services:
	  app:
	    image: monica:fpm
	    depends_on:
	      - db
	    environment:
	      - APP_KEY= # Generate with `echo -n 'base64:'; openssl rand -base64 32`
	      - DB_HOST=db
	      - DB_USERNAME=monica
	      - DB_PASSWORD=secret
	    volumes:
	      - data:/var/www/html/storage
	    restart: always

	  web:
	    build: ./web
	    ports:
	      - 8080:80
	    depends_on:
	      - app
	    volumes:
	      - data:/var/www/html/storage:ro
	    restart: always

	  db:
	    image: mariadb:11
	    environment:
	      - MYSQL_RANDOM_ROOT_PASSWORD=true
	      - MYSQL_DATABASE=monica
	      - MYSQL_USER=monica
	      - MYSQL_PASSWORD=secret
	    volumes:
	      - mysql:/var/lib/mysql
	    restart: always

	volumes:
	  data:
	    name: data
	  mysql:
	    name: mysql
	```

3.	Set a value for `APP_KEY` variable before you run this setup. It should be a random 32-character string. You can for instance copy and paste the output of `echo -n 'base64:'; openssl rand -base64 32`:

4.	Run

	```console
	docker compose up -d
	```

	Wait until all migrations are done and then access Monica at http://localhost:8080/ from your host system. If this looks ok, add your first user account.

5.	Run this command once:

	```console
	docker compose exec app php artisan setup:production
	```

## Make Monica available from the internet

To expose your Monica instance for the internet, it's important to set environment variable `APP_ENV=production`. In this case `https` mode will be mandatory.

### Using a proxy webserver on the host

One way to expose your Monica instance is to use a proxy webserver from your host with SSL capabilities. This is possible with a reverse proxy.

### Using a proxy webserver container

See some examples of Docker Compose possibilities in the [example section](%%GITHUB-REPO%%/blob/main/.examples) to show how to a proxy webserver with ssl capabilities.
