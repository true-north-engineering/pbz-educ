# Lab 3 - Microservices and backend development:

Open https://github.com/true-north-engineering/pbz-educ-src repository, clone it and open the `lab3` folder in your IDE. Checkout to the branch named `userX`

Familiarize yourself with the code. You will notice that some things are missing. Not to worry, we will add them soon.

## Task 1 - mysql container

Open compose.yaml in the repository root. There you will see a service named `mysql-data` which is missing some configuration. Add the configuration by adding the port `3306`, user and password of your choosing and healthcheck with timeout of 3 seconds and 5 retries.

## Task 2 - running the application locally

## Task 3 - application container

## Task 4 - logging

Open `logback.xml` file. Add two `<springProfile>` tags: `json-logs` and `!json-logs`. The first we will use to configure logging in json format, and the second in a more readable console format. Add the missing configuration to both of them. The pattern for the second one should be:
```%d{yyyy-MM-dd HH:mm:ss} [%X{traceId:-}] [%thread] %-5level %logger{36} - %msg%n```


Run the application in both profiles with following commands and observe the difference in logging formats:
```mvn spring-boot:run -Dspring-boot.run.profiles=json-logs```
```mvn spring-boot:run```

## Task 5 - liveness and healthness probes

## Task 6 - profiles
