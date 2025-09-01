# Lab 1 - Containers

Before starting with lab exercises connect to the linux system box-edu.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@box-edu.tn.hr```

Clone the source repo with the following command:

```git clone https://github.com/true-north-engineering/pbz-educ-src.git```

## Task 1 - Create mysql container

1. Go to public DockerHub repo and find mysql 8 container image. Go through the documentation and find which environment variables are used for database configuration, especially MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.

2. Create internal podman network mynet using bridge driver.

3. Create a directory for mysql data in /home/<your_username>/mysql

4. Create container which has the following properties:
    * Container name is mysql
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE to values of your choosing
    * Container is attached to mynet network
    * Container mounts /home/<your_username>/mysql on host system to /var/lib/mysql folder in the container
    * Container image used is docker.io/mysql:8

5. List the running containers

6. Observe the logs of mysql container

7. Exec into the container, connect to the database specified with environment variables and create the ***Item*** table
```
# Command for connecting to the database
mysql -h localhost -u<mysql_username> -p<mysql_password> <mysql_database>

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

2. Build the Containerfile and name the image todo:latest

3. List and inspect the built image

4. Create container with built image. The container should have the following properties:
    * Container name is todo
    * Container is running in background (detached mode)
    * Set the values of the following environment variables MYSQL_ENV_MYSQL_DATABASE, MYSQL_ENV_MYSQL_USER, MYSQL_ENV_MYSQL_PASSWORD to values you have specified when starting mysql container.
    * Set the values of the environment variables MYSQL_ENV_MYSQL_HOST to the name of mysql container and MYSQL_ENV_MYSQL_PORT to 3306
    * Container is attached to mynet network
    * Container image used is todo:latest
    * Publish container port 30080 on any host port reserved to your user (e.g. for user10, the range is 30**10**0-30**10**9)

5. Find out the port on which the container is listening on the host.

6. Test the app by opening the URL ```http://box-edu.tn.hr:<port>/todo/```. Please note that the last trailing slash (/) is important, so don't omit it.

## Task 3 - Push the image to Nexus

1. Go to https://nexus-edu.tn.hr and login with username user. Password is stored on the ```box-edu.tn.hr``` in file ```/edu/nexus-credentials```. You can view the content of this file with command ```cat /edu/nexus-credentials```

2. Login with podman to docker-nexus-edu.tn.hr.

3. Tag the todo:latest image with podman to docker-nexus-edu.tn.hr/<your_username>/todo:latest

4. Push the image to Nexus.

5. Browse the Nexus through web interface and find your image.

## Task 4 - Build the environment using podman-compose

1. Create compose.yml file for podman-compose in your home folder. The compose.yml shoud:
    * Define mynet network
    * Define mysql service from docker.io/mysql:8 image, connected to mynet network, and defined environment variables with the same names and values like in Task 1.
    * Define todo service from todo:latest image, connected to mynet network, defined environment variables with the same names and values like in Task 2, and exposed container port 30080 to any host port reserved for your user.

Compose file reference -> https://docs.docker.com/reference/compose-file/

2. Start the environment.

3. List the created containers and networks with podman command.

4. Stop the environment.

## Task 5 - Cleanup

1. Cleanup everything by stopping the containers, removing them, removing the images and network mynet.
