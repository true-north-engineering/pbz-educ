# Lab 1 - Containers

Be aware that these solutions are for ***user50***. Please adjust all paths for your username!

## Task 1 - Create mysql container

1. Go to public DockerHub repo and find mysql 8 container image. Go through the documentation and find which environment variables are used for database configuration, especially MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.

2. Create internal podman network mynet using bridge driver.

```podman network create mynet```

3. Create a directory for mysql data in /home/<your_username>/mysql

```mkdir /home/user50/mysql```

4. Create container which has the following properties:
    * Container name is mysql
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE to values of your choosing
    * Container is attached to mynet network
    * Container mounts /home/<your_username>/mysql on host system to /var/lib/mysql folder in the container
    * Container image used is docker.io/mysql:8

```podman run -d --name mysql -e MYSQL_ROOT_PASSWORD=r00tpassw0rd -e MYSQL_USER=todo -e MYSQL_PASSWORD=todopassw0rd -e MYSQL_DATABASE=todo --network mynet -v /home/user50/mysql:/var/lib/mysql:Z docker.io/mysql:8```

5. List the running containers

```podman ps```

6. Observe the logs of mysql container

```podman logs mysql```

7. Exec into the container, connect to the database specified with environment variables and create the ***Item*** table
```
podman exec -it mysql bash

# Command for connecting to the database
mysql -h localhost -utodo -ptodopassw0rd todo

# Create the table
create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));

# Check if table was created
show tables;

# Check if select works
select * from Item;
```

## Task 2 - Write a Containerfile for nodejs application and run the application

1. The Containerfile should satisfy the following requirements
    * The location of Containerfile should be ~/pbz-educ-src/todo-single/Containerfile
    * Base image should be node:12
    * WORKDIR should be /home/node/app
    * Copy the contents of nodejs folder into WORKDIR
    * Run ```npm install```
    * Set CMD to run ```node app.js```

Containerfile / Dockerfile reference -> https://docs.docker.com/reference/dockerfile/

```
cd ~/pbz-educ-src/todo-single
vi Containerfile
```

Content of the Containerfile:

```
FROM node:12

WORKDIR /home/node/app
COPY . /home/node/app
RUN npm install

CMD ["node", "app.js"]
```

2. Build the Containerfile and name the image todo:latest

```podman build -t todo:latest .```

3. List and inspect the builded image

```
podman images
podman inspect todo:latest
```

4. Create container with builded image. The container should have the following properties:
    * Container name is todo
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ENV_MYSQL_DATABASE, MYSQL_ENV_MYSQL_USER, MYSQL_ENV_MYSQL_PASSWORD to values you have specified when starting mysql container.
    * Set the values od the environment variables MYSQL_ENV_MYSQL_HOST to the name of mysql container and MYSQL_ENV_MYSQL_PORT to 3306
    * Container is attached to mynet network
    * Container image used is todo:latest
    * Publish container port 30080 on any host port reserved to your user (e.g. for user10, the range is 30**10**0-30**10**9)

```podman run -d -p 30080:30501 --network mynet -e MYSQL_ENV_MYSQL_DATABASE=todo -e MYSQL_ENV_MYSQL_USER=todo -e MYSQL_ENV_MYSQL_PASSWORD=todopassw0rd -e MYSQL_ENV_MYSQL_HOST=mysql -e MYSQL_ENV_MYSQL_PORT=3306 --name todo todo:latest```

5. Find out the port on which the container is listening on the host.

```podman ps```

Note the PORTS column of todo container. It has a value like ```0.0.0.0:30501->30080/tcp```. This means that the host port 30501 is redirected to the container's port 30080. In this case the app is available outside on port 30501.

6. Test the app by opening the URL ```http://box-edu.tn.hr:<port>/todo/```. Please note that the last trailing slash (/) is important, so don't omit him.

Open URL http://box-edu.tn.hr:30501/todo/

## Task 3 - Push the image to Nexus

1. Go to https://nexus-edu.tn.hr and login with username user. Password is stored on the ```box-edu.tn.hr``` in file ```/edu/nexus-credentials```. You can view the content of this file with command ```cat /edu/nexus-credentials```

2. Login with podman to docker-nexus-edu.tn.hr.

```podman login docker-nexus-edu.tn.hr```

3. Tag the todo:latest image with podman to docker-nexus-edu.tn.hr/user50/todo:latest

```podman tag todo:latest docker-nexus-edu.tn.hr/user50/todo:latest```

4. Push the image to Nexus.

```podman push docker-nexus-edu.tn.hr/user50/todo:latest```

5. Browse the Nexus through web interface and find your image.

## Task 4 - Build the environment using podman-compose

1. Create compose.yml file for podman-compose in your home folder. The compose.yml shoud:
    * Define mynet network
    * Define mysql service from mysql:8 image, connected to mynet network, and defined environment variables with the same names and values like in Task 1.
    * Define todo service from todo:latest image, connected to mynet network, defined environment variables with the same names and values like in Task 2, and exposed container port 30080 to any host port reserved for your user.

Compose file reference -> https://docs.docker.com/reference/compose-file/

```
cd ~
vi compose.yml
```

Contents of compose.yml

```
services:
  mysql:
    image: docker.io/mysql:8
    environment:
      MYSQL_DATABASE: todo
      MYSQL_USER: todo
      MYSQL_ROOT_PASSWORD: todopassw0rd
    networks:
      - mynet
  todo:
    image: todo:latest
    environment:
      MYSQL_ENV_MYSQL_DATABASE: todo
      MYSQL_ENV_MYSQL_USER: todo
      MYSQL_ENV_MYSQL_PASSWORD: todopassw0rd
      MYSQL_ENV_MYSQL_HOST: todopassw0rd
      MYSQL_ENV_MYSQL_PORT: 3306
    networks:
      - mynet
    ports:
      - 30080:30501

networks:
  mynet:
```

2. Start the environment.

```podman-compose up -d```

3. List the created containers and networks with podman command.

```
podman ps
podman network ls
podman logs user0_mysql_1
podman logs user0_todo_1
```

4. Stop the environment.

```podman-compose down```

## Task 5 - Cleanup

1. Cleanup everything by stopping the containers, removing them, removing the images and network mynet.

```
podman ps
podman stop todo
podman stop mysql
podman ps
podman ps -a
podman rm -a

podman images
podman rmi -a
podman images -a

podman network ls
podman network rm mynet
podman network ls
```
