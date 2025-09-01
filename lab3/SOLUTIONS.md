# Lab 3 - Microservices and backend development:

Open https://github.com/true-north-engineering/pbz-educ-src repository, clone it and open the `lab3` folder in your IDE.

Familiarize yourself with the code.

## Task 1 - mysql container

Open compose.yaml in the repository root. There you will see a service named `mysql-data` which is missing some configuration. Add the configuration by adding the port `3306`, database name `lab3`, root password of your choosing and healthcheck with timeout of 3 seconds and 5 retries.

```
services:
# code omitted ...
  mysql-data:
    image: docker.io/library/mysql:8.0
    container_name: lab3-database
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: labpassword
      MYSQL_DATABASE: lab3
    volumes:
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]
      timeout: 3s
      retries: 5

volumes:
    mysql-data:
```

## Task 2 - pom.xml




## Task 4 - logging

Open `logback.xml` file. Add two `<springProfile>` tags: `json-logs` and `!json-logs`. The first we will use to configure logging in json format, and the second in a more readable console format. Add the missing configuration to both of them. The pattern for the second one should be:
```%d{yyyy-MM-dd HH:mm:ss} [%X{traceId:-}] [%thread] %-5level %logger{36} - %msg%n```

```
<springProfile name="!json-logs">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%X{traceId:-}] [%thread] %-5level %logger{36} - %msg%n</pattern>
      </encoder>
    </appender>
  </springProfile>

  <springProfile name="json-logs">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>
  </springProfile>
```

Run the application in both profiles with following commands and observe the difference in logging formats:
```mvn spring-boot:run -Dspring-boot.run.profiles=json-logs```
```mvn spring-boot:run```


## Task 5 - liveness and healthness probes

## Task 6 - profiles
