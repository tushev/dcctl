# dcctl
A handy tool that simplifies management of `docker-compose` applications on Linux systems.

ℹ (Version 0001): **README is a work in progress.** `dcctl` **itself is functional, except for** `dcctl mysql` **command** (not implemented yet)

## Concepts

- "Service" - an application, defined by `docker-compose.yml` (and optionally `.env`) file + extra files
- Each service is located in a separate directory
- Most likely the service is located in `/docker` subdir in user's home directory, i.e.: `/home/wordpress/docker` (it is expected that each service has its own unix user; not mandatory)
- Services may also be located in other directories. Version 0001 requires to specify full path to dir in this case.
- If there's no service specfied, `dcctl` looks for `docker-compose.yml` in current directory
- `.env` is expected to contain `COMPOSE_PROJECT_NAME` variable (not mandatory)
- Service is "started" by `docker-compose up -d --build` command
- Service is "stopped" by `docker-compose stop && docker-compose rm -f` command

This is example directory structure. It has three services (aka docker-compose projects): `cloud`, `wordpress` and `pihole`. The first two reside in `/home` (and have their corresponding unix users).
```
/home
├── cloud
│   ├── db
│   ├── data
│   ├── docker
│   │   ├── .env
│   │   └── docker-compose.yml
│   └── nextcloud-www
└── wordpress
    ├── db
    ├── docker
    │   ├── .env
    │   └── docker-compose.yml
    └── www

/data/docker.d/pihole
    ├── etc
    └── dc
        └── docker-compose.yml
```

## Installation
```sh
curl -o /usr/local/bin/dcctl https://raw.githubusercontent.com/tushev/dcctl/main/dcctl && chmod +x /usr/local/bin/dcctl
```
Use at your own risk.
```
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Usage

- `dcctl start` Starts service defined by `docker-compose.yml` in current directory
- `dcctl start wordpress` Starts service defined by `/home/wordpress/docker/docker-compose.yml`
- `dcctl start /data/docker.d/pihole/dc` Starts service defined by `/data/docker.d/pihole/dc/docker-compose.yml`
- `dcctl stop wordpress` Stops Wordpress service
- `dcctl restart wordpress` Restarts Wordpress service<br>(equals to `docker-compose stop && docker-compose rm -f && docker-compose up -d --build`)
- `dcctl update wordpress apache-custom redis-custom` Equals to: <br>
```
	cd /home/wordpress/docker/
	docker-compose build --no-cache --pull apache-custom
	docker-compose build --no-cache --pull redis-custom
	docker-compose pull
	docker-compose up -d
```
- `dcctl bash wordpress php-fpm` Equals to `docker-compose run --rm php-fpm bash`
- `dcctl env wordpress` Shows `/home/wordpress/docker/.env`
- `dcctl list` Equals to `docker ps --format "much_better_formatted"`

ℹ This list is incomplete. Please read the following section for more info.

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
