
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
