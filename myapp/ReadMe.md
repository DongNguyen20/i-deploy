DO:
----


**1.Create** `dockerfile`

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

**2.Build**
```bash
mvn clean package
docker build -t my-app .
```

**3.Run app by docker**
```bash
docker run -p 8080:8080 my-app
```
**4.update** `docker-compose.yml` **file** ->
```bash
docker compose up --build
```