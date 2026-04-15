FROM maven:3.9.6-eclipse-temurin-11 AS build

WORKDIR /app
COPY . .

RUN mvn clean package -DskipTests

FROM openjdk:11.0.10-jre

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 9999

CMD ["java","-jar","app.jar"]