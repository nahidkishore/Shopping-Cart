FROM openjdk:8u151-jdk-alpine3.7

LABEL maintainer="nahid@example.com"

EXPOSE 8070

ENV APP_HOME=/usr/src/app
WORKDIR $APP_HOME

ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
