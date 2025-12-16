FROM eclipse-temurin:17-jre
WORKDIR /App
COPY target/*.jar app.jar
EXPOSE 8282
ENTRYPOINT ["java","-jar","app.jar"]
