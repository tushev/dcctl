# dcctl
A handy tool that simplifies management of `docker-compose` projects.

## Concepts

- "Service" - an application, defined by `docker-compose.yml` (and optionally `.env`) file, located in a separate directory
- (v. 0.1) Each service is located in `/docker` subdir in user's home directory, i.e.: `/home/wordpress/docker`

## Usage

- `dcctl start` Starts 

## dcctl -h
```
Usage: dcctl [-h] [-d subdir] [-x] [-y] command [service_name] [extra_args]

Simplifies management for docker-compose services.
Key feature: automatic discovery of docker-compose.ya?ml directory from SERVICE_NAME.

Available options:

-h, --help      		Print this help and exit
-y				Don't ask for confirmation
-x, --allow-nonexisting		Use . in case $user/$subdir or $path does not contain docker-compose
-d subdir, --subdir subdir	Specify user's subdir for docker-compose (default: docker)

command [extra_args]:
	start			Start SERVICE_NAME			(docker-compose up -d --build)
	stop			Stop SERVICE_NAME			(docker-compose stop && docker-compose rm -f)
	restart			Restart SERVICE_NAME
	status			Show status for SERVICE_NAME		(docker-compose ps)
	logs			Show logs for SERVICE_NAME		(docker-compose logs --tail=100)
	down			Stop and remove	SERVICE_NAME		(docker-compose down)
	enter  CONTAINER	Start the specified container		(docker-compose run --rm CONTAINER)
	sh     CONTAINER	Start sh in container			(docker-compose run --rm CONTAINER sh)
	bash   CONTAINER	Start bash in container			(docker-compose run --rm CONTAINER bash)
	update CONTAINERs	Build CONTAINERs, pull others, up -d	[see examples]
	env			Show .env file for the service		(cat .env)
	env edit		Edit .env file for the service		(EDITOR .env)
	mysql
	prune [args]		Remove unused images, containers etc.	(docker system prune [args])
	list			List all running containers		(docker ps --format "$docker_ps_format" ... [see examples] )

Examples (and their actions):
	- dcctl start wordpress
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: docker-compose up -d --build
	- dcctl stop wordpress
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: docker-compose stop && docker-compose rm -f
	- dcctl logs wordpress
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: docker-compose logs --tail=100
		
	- dcctl -d web/wp-dc start wordpress
		=> find wordpress (most likely in /home/wordpress/web/wp-dc), cd there, run: docker-compose up -d --build
	- dcctl start /data/docker.d/wordpress/docker-compose
		=> cd /data/docker.d/wordpress/docker-compose, run: docker-compose up -d --build
		
	- dcctl enter wordpress php-fpm
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: docker-compose run --rm php-fpm		
	- dcctl bash wordpress php-fpm
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: docker-compose run --rm php-fpm bash
		
	- dcctl update wordpress
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: 
			docker-compose pull
			docker-compose up -d
	- dcctl update wordpress apache-custom redis-custom
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run:
			docker-compose build --no-cache --pull apache-custom
			docker-compose build --no-cache --pull redis-custom
			docker-compose pull
			docker-compose up -d
		
	- dcctl env wordpress
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: cat .env
	- dcctl env wordpress edit
		=> find wordpress (most likely in /home/wordpress/docker), cd there, run: mcedit** .env
		(**: as specified in 'file_editor' variable in /usr/local/bin/dcctl)
		
	- dcctl prune
		=> run: docker system prune
	- dcctl prune --volumes
		=> run: docker system prune --volumes
	- dcctl list
		=> run: docker ps --format "docker_ps_format**" | (read -r; printf "%s\n" "$REPLY"; sort -k 1 )
		(**: as specified in 'docker_ps_format' variable in /usr/local/bin/dcctl)
```
