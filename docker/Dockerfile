FROM eclipse-temurin:17-jdk-alpine

EXPOSE 8070

ENV APP_HOME=/usr/src/app
WORKDIR $APP_HOME

COPY target/shopping-cart-*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
