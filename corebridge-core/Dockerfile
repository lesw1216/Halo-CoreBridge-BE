FROM eclipse-temurin:17-jdk

WORKDIR /home/core_bridge

# 멀티모듈에서 corebridge-core 모듈 JAR만 복사
COPY build/libs/corebridge-core-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

# WORKDIR 아래에서 실행됨. /home/core_bridge 아래에서 실행
ENTRYPOINT ["java", "-jar", "app.jar"]