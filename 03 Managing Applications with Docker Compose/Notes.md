Docker Compose
--------------

Manage multi container environments. This can be even done for containers in different cluster by running in Swarm mode. 

- Compose file
- Command line interface

Commands
--------

Setting and tearing down docker environment

> docker network create create --driver bridge app_network
> docker volume create serviceB_volume
> docker build -f Dockerfile.serviceA .
> docker build -f Dockerfile.serviceB .
> docker run -d -name serviceA --network app_network -p 8080:3000 serviceA
> docker run -d -name serviceB --network app_network --mount source=serviceB_volumne,target=/data serviceB
> docker stop serviceA serviceB
> docker rm serviceA serviceB
> docker rmi serviceA serviceB
> docker volume rm serviceB_volume
> docker network rm app_network

Alternate option to create shell scripts 

spin_up.sh
tear_down.sh

Alternate option to use docker compose to manage multi-containers environments


> docker-compose -f wordpress.yml up 
> docker-compose -f wordpress.yml up -d
> docker-compose -f wordpress.yml down
> docker-compose -f wordpress.yml down --rmi all --volumes --remove-orphans
> docker image prune -a
> docker-compose -f wordpress.yml up -d --force-recreate
> docker-compose -f wordpress.yml up -d --no-deps

> docker-compose -f dev.docker-compose.yml up -d
> docker-compose -f dev.docker-compose.yml up -d --build
> docker-compose -f dev.docker-compose.yml down


YAML
----
-> Used for data serialization
-> .yaml or .yml extension
-> Space sensitive
-> $VARIABLE or ${VARIABLE}
-> Data types
	Integers, Strings, Nulls, Booleans
-> Collections
	mappings 
		key: value
		key1: value1
		key2: value2
	nested mappings
		outerKey:
			innerKey: innerValue
	inline mappings
		outerKey: {innerKey: innerValue}
	sequences
		- item
		- item1
		- item2
	nested sequences
		- 
			- innerValue
	inline sequences
		[value1, value2]
	Combinations of sequences and mappings
		key: 
			- value1
			- value2

		key:
		- value

		- 
			key1: value1
			key2: value2

		- key: value

-> Comments 

Compose file templates
----------------------

version: '3'
services:
	app-cache:
		image: redis:4.0.6
		ports:
		- '6379:6379'
		command: redis-server --appendonly yes
		volumes:
			# Named volume
			- named-volume:/data
			# Compose file relative path
			- ./cache:/tmp/cache
			# Auto created volume
			- /tmp/stuff
volumes:
	named-volume:
	external-volume:
		external: true


build: ./dir
OR
build: 
	context: ./dir
	dockerfile: the.dockerfile
	args: 
		buildno: 1

Docker Compose CLI
------------------

For Mac:
	Docker for Mac & Docker Toolbox

For Winodws:
	Docker for Windows & Docker Toolbox

For Linux:
	sudo (yum|dnf|apt) install docker-compose

Usage
> docker-compose [OPTIONS] [COMMAND] [ARGS]
> build, config, create, events, exec, images, kill, logs, pause, port, ps, pull, push, restart, rm, run, start, stop, top, unpause, version
> up, down
> docker-compose --help
> docker-compose up --help
> docker-compose -f l-extension-fields.yml config


Example
-------

dev.dockerfile

# Node.js version 6 base image
FROM node:6

# use nodemon for development
RUN npm install --global nodemon

# install package.json dependencies
RUN mkdir src
WORKDIR /src
ADD src/package.json /src/package.json
RUN npm install

# Development app runs on port 3000
EXPOSE 3000

# Run server and watch for changes
CMD ["nodemon", "-L", "/src/app/bin/www"]
----------------------------------------------------------

dev.docker-compose.yml

version: '3'
services: 
	app:
		build: 
			context: .
			dockerfile: dev.dockerfile
		image: accumulator
		ports: '3000:3000'
		environment: 
			- NODE_ENV=development
			- DB_HOST=app-db
		volumes:
			- './src:/src/app'
		depends_on:
			- app_db
		networks:
			- backend
	app_db:
		image: mongo:3
		networks:
			- backend
networks:
	backend: 


> docker-compose -f dev.docker-compose.yml up -d



Example with multiple compose files
------------------------------------


dev.dockerfile

# Node.js version 6 base image
FROM node:6

# use nodemon for development
RUN npm install --global nodemon

# install package.json dependencies
RUN mkdir src
WORKDIR /src
ADD src/package.json /src/package.json
RUN npm install

# Development app runs on port 3000
EXPOSE 3000

# Run server and watch for changes
CMD ["nodemon", "-L", "/src/app/bin/www"]

-----------------------------------------------------

prod.dockerfile

# Node.js version 6 base image
FROM node:6

# Production app runs on port 8080
EXPOSE 8080

# Copy source files into container
COPY ./src /app

# INstall production dependencies and build app
RUN npm install --production && npm run build

# Start server in production mode
CMD ["npm", "start"]

----------------------------------------------------


docker-compose.yml

version: '3'
services: 
	app:
		image: your-registry:5000/accumulator
		environment: 
			- DB_HOST=app-db
		volumes:
			- './src:/src/app'
		depends_on:
			- app_db
		networks:
			- backend
	app_db:
		image: mongo:3
		networks:
			- backend
networks:
	backend: 

----------------------------------------------------

dev.docker-compose.yml

version: '3'
services: 
	app:
		build:
			context: .
			dockerfile: dev.dockerfile
		ports:
			- '3000:3000'
		environment:
			- NODE_ENV=development
		volumes:
			- './src:/src/app'

----------------------------------------------------

prod.docker-compose.yml

version: '3'
services: 
	app:
		build:
			context: .
			dockerfile: prod.dockerfile
		ports:
			- '8080'
		environment:
			- NODE_ENV=production
		restart: always
	app-db:
		volumes:
			- db-data:/data/db
		restart: always
volumes: 
	db-data:


> docker-compose -f docker-compose.yml -f dev.docker-compose.yml config
> docker-compose -f docker-compose.yml -f prod.docker-compose.yml config


Support
-------
Twitter @LoganRakai 