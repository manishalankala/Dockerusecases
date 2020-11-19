


Dockerfile


```

FROM openjdk:8

RUN apt-get update && apt-get install -y maven
COPY . /project
RUN  cd /project && mvn package

#run the spring boot application
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom", "-Dblabla", "-jar","/project/target/demo-0.0.1-SNAPSHOT.jar"]


```


docker-compose.yml


```
version: "3"

networks:
  test:

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "18080:8080"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - test

  db:
    image: opavlova/db-mysql:5.7-test
    container_name: db
    ports:
      - "13306:3306"
    healthcheck:
          test: ["CMD", "mysql", "-h", "localhost", "-P", "3306", "-u", "root", "--password=root", "-e", "select 1", "DOCKERDB"]
          interval: 1s
          timeout: 3s
          retries: 30
    networks:
      - test


```
