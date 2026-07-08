```dockerfile
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