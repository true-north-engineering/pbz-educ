# Lab 3 - Microservices and backend development:

Open https://github.com/true-north-engineering/pbz-educ-src repository, clone it and open the `lab3` folder in your IDE. Checkout to the branch named `userX`

Familiarize yourself with the code. You will notice that some things are missing. Not to worry, we will add them soon.

## Task 1 - mysql container

Open `compose.yaml` in the repository root. There you will see a service named `mysql-data` which is missing some configuration. Add the configuration by adding the port `3306`, and `MYSQL_DATABASE, MYSQL_USERNAME, MYSQL_PASSOWRD, MYSQL_ROOT_PASSWORD`.

Make sure that database is up. Run `podman-compose up -d`, and next, run `podman-compose ps`. You should see that the pod with the name `lab3-database` is running.

Next, go to `application.properties` and add the following configuration:
```
spring.datasource.url=jdbc:mysql://localhost:3306/{DATABASE_NAME}
spring.datasource.username={USERNAME}
spring.datasource.password={PASSWORD}
```


## Task 2 - pom.xml

Open `pom.xml` file. There you will se that some dependecies are missing. Add the `spring-boot-starter-data-jpa`, `spring-boot-starter-web`, `spring-boot-starter-actuator` dependencies. With them added, you should be able to build the project. Verify that is the case with `mvn clean install` command.

## Task 3 - logging

In `pom.xml` file, add the following dependency:

```
<dependency>
      <groupId>net.logstash.logback</groupId>
      <artifactId>logstash-logback-encoder</artifactId>
      <version>6.6</version>
    </dependency>
```


Open `logback.xml` file. Add two `<springProfile>` tags: `json-logs` and `!json-logs`. The first we will use to configure logging in json format, and the second in a more readable console format. Add the missing configuration to both of them. The pattern for the second one should be:
```%d{yyyy-MM-dd HH:mm:ss} [%X{traceId:-}] [%thread] %-5level %logger{36} - %msg%n```

Run the application in both profiles with following commands and observe the difference in logging formats:
```mvn spring-boot:run -Dspring-boot.run.profiles=json-logs```
```mvn spring-boot:run```

## Task 4 - profiles

Now its time to have some fun with profiles. In `tn.pbz.educ.lab3.service` package you will see a `ProfileService` interface. it contains a `getProfileMessage` method. Your task will be to add two classes that will implement this interface. One should be called `ProfileServiceImpl`, and the other `JsonProfileServiceImpl`. You can implement them both to return any message you like. The difference in implementation between these two classes is that `JsonProfileServiceImpl` should be active when the application is run in `json-logs` profile, and `ProfileServiceImpl` in any other profile.

Once you are done with implementation, go to the `ProfileController` class in `tn.pbz.educ.lab3.controller` package and uncomment the commented lines.
Verify your implementation with following steps:

1) run the app with `mvn spring-boot:run -Dspring-boot.run.profiles=json-logs` and send request to `http://localhost:8080/api/profile`. Did you get the expected message?
2) run the app with `mvn spring-boot:run` and, again, send the request to `http://localhost:8080/api/profile`. Did the message change?


## Task 5 - liveness and healthness probes

Open `application.properties`, and add the following configuration:

```
management.endpoints.web.exposure.include=health,info
management.endpoint.health.probes.enabled=true
management.endpoint.health.show-details=always
```

Run the application with `mvn spring-boot:run` and send the request to `http://localhost:8080/actuator/health`. Inspect what you see there. If everything is okay, the response should be something like:

```
{
    "status": "UP",
    "groups": [
        "liveness",
        "readiness"
    ],
    "components": {
        "db": {
            "status": "UP",
            "details": {
                "database": "MySQL",
                "validationQuery": "isValid()"
            }
        },
        "diskSpace": {
            "status": "UP",
            "details": {
                "total": 499680116736,
                "free": 258922356736,
                "threshold": 10485760,
                "path": "/home/doriankablar/Workspace/pbz-educ/pbz-educ-src/lab3/.",
                "exists": true
            }
        },
        "livenessState": {
            "status": "UP"
        },
        "ping": {
            "status": "UP"
        },
        "readinessState": {
            "status": "UP"
        },
        "ssl": {
            "status": "UP",
            "details": {
                "validChains": [],
                "invalidChains": []
            }
        }
    }
}
```

Everything is running nicely. Let's break it. The easiest way to do that is to stop our database. Do it with `podman-compose down` command. Send the request to `http://localhost:8080/actuator/health` and wait for the response (it might take 30-60 seconds). Do you notice the difference? The database is down, so the `db` component and the overall status should be different.

Run the `podman-compose up -d` command again, wait for database to reestablish its connection and send the request to `http://localhost:8080/actuator/health`. Verify that everything is back to normal.

Now we will write some probes ourselves. Go to the `tn.pbz.educ.lab3.resource` package and create three classes: `AlwaysUpHealthIndicator`, `AlwaysDownHealthIndicator` and `DatabasereadinessIndicator`. They should all implement the `HealthIndicator` interface.

1) `AlwaysUpHealthIndicator` should be implemented in a way that it always returns `"status": "UP"` response
2) `AlwaysDownHealthIndicator` should be implemented in a way that it always returns `"status": "DOWN"` response
3) `DatabseReadinessIndicator` will be our custom database readiness probe implementation. Implement it so that it runs a query `SELECT COUNT(*) FROM person` and that it gives following responses based on the result:
```
{
	"status": "UP",
	"details": {
		"database": "Ready"
	}
}
```
if everything is okay,
```
{
	"status": "UP",
	"details": {
		"database": "Unexpected result"
	}
}
```
if the result it got is not expected and
```
{
	"status": "UP",
	"details": {
		"database": "Not reachable"
	}
}
```
if database is not reacahble.


Once you have implemented these three classes, run the application, send the request to `http://localhost:8080/actuator/health` and check the response. If your `DatabaseReadinessIndicator` is down, don't worry, it should be. We still don't have the `Person` table. Create it with following query:

```
CREATE TABLE IF NOT EXISTS person (
    `id`   BIGINT         NOT NULL AUTO_INCREMENT comment 'id identifier for this table',
    `name` VARCHAR(255)   NOT NULL comment 'the person name',
    `age`  INT            NOT NULL comment 'the person age',
    PRIMARY KEY (`id`)
);
```

Once you create the table, send the `actuator/health` request again and check the response. `DatabaseReadinessIndicator` should be okay. The only problem with the probes should be in `AlwaysDownHealthIndicator`, but we designed it to be that way. We can remove it now by deleting the class.

## Task 6 - application container

Go to `Dockerfile` file, you will see that some lines are missing. Fill those blanks (there are comments that should tell you what is missing). Once you are done, run the `podman-compose build` command to verify.

Go back to `compose.yaml`. In there, create a new service named `spring-lab3-app-json`, have it use `Dockerfile` in build, expose the `8080` port, have it depend on `mysql-data` service on condition that it is healthy. ALso, define the -env_file with name `.env-json`. Create that file and fill it with the values from `.env-example` (make sure that the profile is `json-logs` and that database credentials are wrong, you will see why).

After you are finished, run the `podman-compose up -d` command, and then `podman-compose logs spring-lab3-app-json`. There should be a problem with database connection. Can you tell why?

Change the credentials in `.env-json` and verify that the app is running. You can now use the `PersonController` and the endpoints that it exposes to create some `Person` entries. For example, run the `POST http://localhost:8080/api/person` with the body:

```
{
    "name": "Filip",
    "age": 25
}
```

and then verify that the entity is created with `GET http://localhost:8080/api/person/1` request.

## Task 7 - multiple app instances

Once again, go back to `compose.yaml` and create an new service named `spring-lab3-app`, have it use `Dockerfule` in build, expose the `8081` port, have it depend on `mysql-data` service on condition that it is healthy. ALso, define the `-env_file` with name `.env`. Create that file and fill it with the values from `.env-example` (make sure that the profile is not `json-logs`).

After you are finished, run the `podman-compose up -d` command, and then `podman-compose logs spring-lab3-app` and verify that the app is running. You can now play with `PersonController` and verify that both applications use the same database. You should also notice that logging is different in these two applications.
