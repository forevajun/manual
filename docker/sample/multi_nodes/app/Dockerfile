FROM openjdk:8-jre-alpine

RUN apt-get update \
    && apt-get install -y ApplicationName

#VOLUME /services/logs/

WORKDIR /services

COPY target/SpringBootApplication-1.0-SNAPSHOT.jar /services/app.jar

EXPOSE 8081

CMD ["java", "-jar", "/services/app.jar"]
