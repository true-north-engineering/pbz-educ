# Lab 1 - Containers

Before starting with lab exercises connect to the linux system box-edu.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@box-edu.tn.hr```

Clone the source repo with the following command:

```git clone https://github.com/true-north-engineering/pbz-educ-src.git```

## Task 1 - Create mysql container

1. Go to public DockerHub repo and find mysql 5.5 container image. Go through the documentation and find which environment variables are used for database configuration, especially MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.

2. Create internal podman network mynet using bridge driver.

3. Create a directory for mysql data in /home/<your_username>/mysql

4. Create container which has the following properties:
    * Container name is mysql
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE to values of your choosing
    * Container is attached to mynet network
    * Container mounts /home/<your_username>/mysql on host system to /var/lib/mysql folder in the container
    * Container image used is mysql:5.5

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
    * Base image should be node:5
    * WORKDIR should be /home/node/app
    * Copy the contents of nodejs folder into WORKDIR
    * Run ```npm install```
    * Set CMD to run ```node app.js```

2. Build the Containerfile and name the image todo:latest

3. List and inspect the builded image

4. Create container with builded image. The container should have the following properties:
    * Container name is todo
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ENV_MYSQL_DATABASE, MYSQL_ENV_MYSQL_USER, MYSQL_ENV_MYSQL_PASSWORD to values you have specified when starting mysql container.
    * Container is attached to mynet network
    * Container image used is todo:latest
    * Publish port 30080

5. Find out the port on which the application is listening.

6. Test the app by opening the URL ```http://box-edu.tn.hr:<port>/todo/```. Please note that the last trailing slash (/) is important, so don't omit him.

## Task 3 - Push the image to Nexus

1. Go to https://nexus-edu.tn.hr and login with username user. Password is stored on the ```box-edu.tn.hr``` in file ```/edu/nexus-credentials```. You can view the content of this file with command ```cat /edu/nexus-credentials```

2. Login with podman to docker-nexus-edu.tn.hr.

3. Tag the todo:latest image with podman to docker-nexus-edu.tn.hr/todo:<your_username>

4. Push the image to Nexus.

5. Browse the Nexus through web interface and find your image.

## Task 4 - Cleanup

1. Cleanup everything by stopping the containers, removing them, removing the images and network mynet.

## Task 5 (Optional) - Build todo app using buildah

1. Build the same todo application from Task 2, Step 1 using Buildah and commit the builded image as todo:latest.

2. List the images with podman.

3. Start the todo app like in task 2.

4. Observe the logs of todo app.

5. Access the application through URL.

6. Stop and cleanup everything.