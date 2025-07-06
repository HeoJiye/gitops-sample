# 멀티스테이지 빌드를 사용하여 최적화된 이미지 생성
FROM debian:11 AS builder

# 패키지 업데이트 및 JDK 17 설치
RUN apt-get update && apt-get install -y openjdk-17-jdk curl wget

# 애플리케이션 소스 복사
WORKDIR /app
COPY . .

# Gradle 빌드 실행
RUN ./gradlew build -x test

# 실행 단계
FROM debian:11

# 패키지 업데이트 및 JDK 17 설치
RUN apt-get update && apt-get install -y openjdk-17-jdk curl wget procps

# 시간대 설정
ENV TZ 'Asia/Seoul'
RUN echo $TZ > /etc/timezone && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata

# 환경 변수 설정
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

# 보안을 위한 패키지 제거
RUN apt-mark remove libp11-kit0 || true

# 애플리케이션 사용자 추가
RUN useradd --user-group --create-home --shell /bin/false spring && \
    usermod --shell /sbin/nologin nobody

# 사용자 변경 및 작업 디렉토리 설정
USER spring
WORKDIR /home/spring

# 빌드된 JAR 파일 복사
COPY --from=builder --chown=spring:spring /app/build/libs/*.jar app.jar

# 헬스체크 추가
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# 포트 노출
EXPOSE 8080

# 애플리케이션 실행
ENTRYPOINT ["java", "-jar", "app.jar"] 