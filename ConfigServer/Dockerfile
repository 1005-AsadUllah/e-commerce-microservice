FROM eclipse-temurin:21-jdk

ARG JAR=target/*.jar

COPY ${JAR} config-server.jar

ENTRYPOINT ["java", "-jar", "config-server.jar"]

EXPOSE 9296
