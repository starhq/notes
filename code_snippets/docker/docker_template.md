# 常用的docker镜像

~~~SEHLL
# portainer
docker pull portainer/portainer-ce:alpine

# prometheus
docker pull prom/prometheus:main-distroless

# grafana
docker pull grafana/grafana-enterprise:nightly-boringcrypto-distroless-slim
~~~

# Dockerfile

## alpine服务器(可以模拟一些物理服务器,比如测试脚本是否可用)

```YAML
FROM alpine:latest

# Set environment variables
ENV LANG=en_US.UTF-8 \
    TZ=Asia/Shanghai

# Install dependencies
RUN set -ex \
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && apk add --no-cache \
           tzdata \
           bash \
           openssh \
           procps \
           curl \
    # generate ssh keys
    && ssh-keygen -A \
    # set root password(not in production env)
    && echo "root:root" | chpasswd \
    # allow root user login
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

#start ssh
CMD ["/usr/sbin/sshd", "-D"]
```

## 打包java镜像

```Dockerfile
# syntax=docker/dockerfile:1.4  
  
# ========================================  
# 阶段1：构建环境  
# ========================================  
FROM gradle:9.5.1-jdk25-alpine AS builder  
WORKDIR /app  
  
# 利用 Docker 缓存: 只有构建脚本或依赖变更时，才会重新下载依赖  
COPY gradle/wrapper gradle/wrapper  
COPY build.gradle settings.gradle gradlew ./  
  
# 利用 Docker 缓存: 只有源码变更时，才会重新编译  
COPY src src  
  
# 构建参数说明:  
# - --mount=type=cache: 挂载 Gradle 缓存目录，极大提升二次构建速度  
# - bootJar: 精准执行打 Fat Jar 任务，避免触发不必要的 jar 或 test 任务  
# - extract: 利用 Spring Boot分层工具提取依赖，为后续镜像分层做准备  
RUN --mount=type=cache,target=/home/gradle/.gradle \  
    --mount=type=cache,target=/app/.gradle \  
    ./gradlew build --no-daemon --parallel --build-cache --exclude-task test && \  
    cp build/libs/java-template.jar app.jar && \  
    java -Djarmode=layertools -jar app.jar extract --destination extracted  
  
# ========================================  
# 阶段2：运行环境  
# ========================================  
FROM eclipse-temurin:25.0.3_9-jre-alpine-3.23  
  
# ------------------------------------------  
# Layer 1: 系统基础配置 (变化频率: 极低)  
# ------------------------------------------  
# 1. 替换为阿里云镜像源，解决国内拉取慢的问题  
# 2. 安装 curl (用于健康检查) 并设置东八区时区  
# 3. 使用虚拟包机制清理 tzdata，减小镜像体积  
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \  
    apk add --no-cache curl && \  
    apk add --no-cache --virtual .build-deps tzdata && \  
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \  
    echo "Asia/Shanghai" > /etc/timezone && \  
    apk del .build-deps  
  
# ------------------------------------------  
# Layer 2: 应用目录与权限 (变化频率: 极低)  
# ------------------------------------------  
# 遵循安全规范: 避免使用 root 用户运行 Java 进程  
RUN addgroup -S appgroup && \  
    adduser -S appuser -G appgroup && \  
    mkdir -p /app && \  
    chown -R appuser:appgroup /app  
  
USER appuser  
WORKDIR /app  
  
# ------------------------------------------  
# Layer 3: 应用分层文件 (变化频率: 由低到高)  
# ------------------------------------------  
# 提取分层文件:  
# dependencies (第三方依赖) > spring-boot-loader (启动器) > snapshot-dependencies (快照依赖) > application (业务代码)  
# 好处: 业务代码变动时，前两层镜像缓存可以复用，极大加快 Docker 构建和推送速度  
COPY --from=builder --chown=appuser:appgroup \  
    /app/extracted/dependencies/ ./ \  
    /app/extracted/spring-boot-loader/ ./ \  
    /app/extracted/snapshot-dependencies/ ./ \  
    /app/extracted/application/ ./  
  
# 健康检查: 探测 Spring Boot Actuator 端点 (确保应用真正就绪)  
# start-period=40s: 给予 Spring Boot 初始化数据库等组件的宽限时间  
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \  
  CMD curl -f http://localhost:8080/actuator/health || exit 1  
  
# JVM 容器化参数:  
# - UseContainerSupport: 让 JVM 感知容器内存限制  
# - MaxRAMPercentage: 动态分配 75% 的可用内存给堆 (避免硬编码 -Xmx)# - ExitOnOutOfMemoryError: 发生 OOM 时立即让容器崩溃退出，便于 K8s/Docker 重启恢复  
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -XX:+ExitOnOutOfMemoryError"  
  
EXPOSE 8080  
  
# 启动命令: 使用提取后的 JarLauncher 加载平铺的 class 文件  
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.launch.JarLauncher"]  
```

# Docker-compose

## prometheus+grafana

~~~YAML
environment:
  HOME: ${HOME}

version: '3.8'

services:
  prometheus:
    image: prom/prometheus:main-distroless
    container_name: star-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ${HOME}/workspace/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    restart: always
    networks: 
      - star-monitor

  grafana:
    image: grafana/grafana-enterprise:nightly-boringcrypto-distroless-slim
    container_name: star-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - ${HOME}/workspace/docker/grafana:/var/lib/grafana
    depends_on:
      - prometheus
    restart: always
    networks: 
      - star-monitor

networks:
  star-monitor:
    driver: bridge
~~~



```YAML
services:
  go-api:
    build:
      context: ../../
      dockerfile: ./deploy/docker/Dockerfile
    container_name: go-api
    restart: on-failure
    ports:
      - "8000:8000"
    healthcheck:
      test: [ "CMD", "nc", "-z", "127.0.0.1", "8000" ]
      interval: 30s
      timeout: 5s
      retries: 3
    networks:
      network:
        ipv4_address: 172.18.0.11
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:latest
    container_name: go-mysql
    restart: on-failure
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: 'example'
      MYSQL_USER: 'qcloud'
      MYSQL_PASSWORD: 'qcloud'
      MYSQL_CHARACTER_SET: utf8mb4
      MYSQL_COLLATION: utf8mb4_unicode_ci
    volumes:
      - nunu-mysql-data:/var/lib/mysql
    networks:
      network:
        ipv4_address: 177.7.0.12
    healthcheck:
      test: [ "CMD", "mysqladmin", "ping", "-h", "localhost" ]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:alpine
    container_name: go-redis
    restart: on-failure
    ports:
      - '6379:6379'
    command: [ "redis-server", "--appendonly", "yes" ]
    volumes:
      - nunu-redis-data:/data
    networks:
      network:
        ipv4_address: 177.7.0.13
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  # data will be stored in host's /var/lib/docker/volumes/nunu-mysql-data/_data directory
  # some give a meaning names, not confuse with other services
  nunu-mysql-data
  nunu-redis-data

networks:
  network:
    # some service not in this docker-compose can link the net too
    attachable: true
    ipam:
      driver: default
      config:
        # 172.18.0.0-172.18.255.255 is the default subnet
        # 172.18.0.1 is gateway
        - subnet: 172.18.0.0/16

# in production env,use docker secret to store sensitive data
```

# 常用镜像启动命令

```SHELL
# portainer
docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/workspace/docker/portainer/data:/data portainer/portainer-ce:alpine
```
