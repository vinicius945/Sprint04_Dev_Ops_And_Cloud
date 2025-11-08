# ESTÁGIO 1: Build (Compilar o .jar com Maven)
# Usamos a imagem do Maven com Java 17, como no seu pom.xml
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
# Roda o build e pula os testes (pois a pipeline já vai testar)
RUN mvn -B -U clean package -DskipTests

# ESTÁGIO 2: Run (Imagem final leve, só com Java)
# Usamos a imagem JRE (Java Runtime) que é menor
FROM eclipse-temurin:17-jre
WORKDIR /app
# Copia apenas o .jar que foi criado no Estágio 1
COPY --from=build /app/target/*.jar ./app.jar
EXPOSE 8080
# A aplicação vai iniciar e pegar as variáveis (SPRING_...) do Azure Web App
ENTRYPOINT ["java", "-jar", "/app/app.jar"]