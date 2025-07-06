# 멀티스테이지 빌드를 사용하여 최적화된 이미지 생성
FROM openjdk:17-jdk-alpine AS builder

# 애플리케이션 소스 복사
WORKDIR /app
COPY . .

# Gradle 빌드 실행
RUN ./gradlew build -x test

# 실행 단계
FROM openjdk:17-jre-alpine

# 애플리케이션 사용자 추가
RUN addgroup -g 1001 -S spring && \
    adduser -S spring -u 1001

# 작업 디렉토리 설정
WORKDIR /app

# 빌드된 JAR 파일 복사
COPY --from=builder /app/build/libs/*.jar app.jar

# 사용자 권한 변경
RUN chown -R spring:spring /app
USER spring

# 헬스체크 추가
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# 포트 노출
EXPOSE 8080

# 애플리케이션 실행
ENTRYPOINT ["java", "-jar", "app.jar"] 