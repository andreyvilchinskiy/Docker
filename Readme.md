Локальный запуск (без Docker)

mvn spring-boot:run

Started FlipCountApplication in 5.5 seconds
Tomcat started on port 8080 (http)

curl http://localhost:8080/api/health

{"status":"OK","service":"Flip Count API","version":"1.0.0"}

Запуск Docker

dockerfile

FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

dockerignore

target/
uploads/
.idea/
*.iml
.git/
README.md

cd Backend
docker build -t flipcount-api:1.0 .

REPOSITORY       TAG       SIZE
flipcount-api    1.0       ~180MB

docker network create flipcount-net

docker run -d --name flipcount-db \
  --network flipcount-net \
  -e POSTGRES_DB=flipcount \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=2507 \
  postgres:16-alpine

docker run -d --name flipcount-api \
  --network flipcount-net \
  -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://flipcount-db:5432/flipcount \
  -e SPRING_DATASOURCE_USERNAME=postgres \
  -e SPRING_DATASOURCE_PASSWORD=2507 \
  flipcount-api:1.0

docker ps

CONTAINER ID   IMAGE                PORTS                    NAMES
def456abc789   flipcount-api:1.0    0.0.0.0:8080->8080/tcp   flipcount-api
abc123def456   postgres:16-alpine   0.0.0.0:5432->5432/tcp   flipcount-db

docker build

$ docker build -t flipcount-api:1.0 .
[+] Building 45.3s (14/14) FINISHED
 => [builder 5/5] RUN mvn clean package -DskipTests
 => [stage-1 2/3] COPY --from=builder /app/target/*.jar app.jar
 => => naming to docker.io/library/flipcount-api:1.0

docker run -p

$ docker run -d --name flipcount-api --network flipcount-net -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://flipcount-db:5432/flipcount \
  -e SPRING_DATASOURCE_USERNAME=postgres \
  -e SPRING_DATASOURCE_PASSWORD=2507 \
  flipcount-api:1.0
abc123def456...

docker ps

CONTAINER ID   IMAGE                COMMAND                  STATUS         PORTS                    NAMES
def456abc789   flipcount-api:1.0    "java -jar app.jar"      Up 30 seconds  0.0.0.0:8080->8080/tcp   flipcount-api
abc123def456   postgres:16-alpine   "docker-entrypoint.s…"   Up 1 minute    0.0.0.0:5432->5432/tcp   flipcount-db

curl

$ curl http://localhost:8080/api/health
{"status":"OK","service":"Flip Count API","version":"1.0.0"}

docker logs

$ docker logs flipcount-api
2026-10-04T14:34:49.103+03:00  INFO 1 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat started on port 8080 (http)
2026-10-04T14:34:49.112+03:00  INFO 1 --- [main] com.flipcount.FlipCountApplication : Started FlipCountApplication in 5.529 seconds