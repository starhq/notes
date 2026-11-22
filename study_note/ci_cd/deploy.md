# 1. 部署forgejo

```yml
networks:
  forgejo:

services:
  forgejo:
    image: codeberg.org/forgejo/forgejo:16
    container_name: forgejo
    environment:
      - USER_UID=1000
      - USER_GID=1000
      # 域名	
      - FORGEJO__server__ROOT_URL=https://forgejo.forgejo.orb.local/
      - FORGEJO__server__DOMAIN=forgejo.forgejo.orb.local
      - FORGEJO__server__SSH_DOMAIN=forgejo.forgejo.orb.local
      
      # action's log  
      - FORGEJO__actions__LOG_RETENTION_DAYS=30
      - FORGEJO__actions__ARTIFACT_RETENTION_DAYS=30
      
      - FORGEJO__database__DB_TYPE=postgres
      - FORGEJO__database__HOST=db:5432
      - FORGEJO__database__NAME=forgejo
      - FORGEJO__database__USER=root
      # 非安全环境下用下面命令修改数据库密码
      # docker compose exec db psql -U root -d forgejo -c "ALTER USER root PASSWORD '新密码';"
      - FORGEJO__database__PASSWD=root
    restart: unless-stopped
    networks:
      - forgejo
    volumes:
      # ${HOME}环境变量,按情况设置 
      - ${HOME}/forgejo/host:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "22:22"
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:18-alpine
    container_name: forgejo-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=forgejo
    networks:
      - forgejo
    volumes:
      # ${HOME}环境变量,按情况设置
      - ${HOME}/postgres:/var/lib/postgresql
    # ports:
      # - "5432:5432"
    shm_size: "256mb"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U forgejo -d forgejo"]
      interval: 5s
      timeout: 5s
      retries: 10
```

# 2. 部署RUNNER

## 2.1 在 Forgejo 上创建 runner，拿到凭据

## 2.2 服务器 B 准备目录并生成配置

```shell
mkdir -p /opt/forgejo-runner/data/.cache
chown -R 1001:1001 /opt/forgejo-runner/data
chmod 775 /opt/forgejo-runner/data/.cache
chmod g+s /opt/forgejo-runner/data/.cache
cd /opt/forgejo-runner

# 用官方镜像生成默认配置（镜像内以非 root 运行）
sh -c 'docker run --rm data.forgejo.org/forgejo/runner:13 forgejo-runner generate-config > data/runner-config.yml'
```

## 2.3 编辑 `data/runner-config.yml`(按需修改)

```yml
# =============================================================================
# MERGE these keys into your existing data/runner-config.yml.
# Do not paste the file over it and do not duplicate sections: the generated file
# already has `runner:`, `container:` and `cache:` blocks - edit those in place.
# Keep your existing `server.connections` (url / uuid / token) untouched.
# =============================================================================

runner:
  # Concurrent jobs. Each Gradle job may use ~2 GB + the JVM overhead, so make sure
  # capacity x per-job memory fits inside the docker-in-docker mem_limit.
  capacity: 2

  # Environment variables injected into EVERY job container.
  # Don't put secrets here - every repo that can use this runner can read them.
  envs:
    TZ: Asia/Shanghai
    # ---- Caches: everything lives in the persistent 'ci-cache' volume at /cache ----
    GRADLE_USER_HOME: /cache/gradle
    MAVEN_OPTS: "-Dmaven.repo.local=/cache/m2"
    npm_config_cache: /cache/npm
    npm_config_store_dir: /cache/pnpm
    GOMODCACHE: /cache/go/mod
    GOCACHE: /cache/go/build
    PIP_CACHE_DIR: /cache/pip
    UV_CACHE_DIR: /cache/uv

    # ---- Memory / CI behaviour ----
    GRADLE_OPTS: "-Xmx2048m -Dorg.gradle.daemon=false"
    NODE_OPTIONS: "--max-old-space-size=1536"

    # ---- Mainland-China mirrors (delete this block if you don't need it) ----
    GOPROXY: "https://goproxy.cn,direct"
    npm_config_registry: "https://registry.npmmirror.com"
    PIP_INDEX_URL: "https://pypi.tuna.tsinghua.edu.cn/simple"
    
  insecure: true

  # Which `runs-on:` values this runner answers to, and the default image for each.
  # The ci-* images are built locally by build-ci-images.sh (Node.js is added to
  # them because most actions, e.g. actions/checkout, need Node inside the job image).
  labels:
    - docker:docker://node:24-bookworm
    - ubuntu-latest:docker://node:24-bookworm
    - node:docker://node:24-bookworm
    - java:docker://ci-java:25
    - go:docker://ci-go:1.25
    - python:docker://ci-python:3.15

container:
  # Mount the shared cache volume into every job container.
  options: "-v ci-cache:/cache"
  # Volumes jobs are allowed to mount (an empty list means none are allowed).
  valid_volumes:
    - ci-cache

# `cache:` can stay at its defaults. With the /cache volume above you don't need
# actions/cache for Gradle, Maven, npm/pnpm, Go or pip.
```

```yml
server:
  connections:
    forgejo:          # 键名随意，多个连接时保持唯一即可
      url: https://forgejo.forgejo.orb.local/    # 换成你的 Forgejo 地址，runner 容器必须可达
      uuid: 33834eef-e758-48c4-a676-1745426747aa # 步骤2.1生成的
      token: d4fe2db46a4c6bdc434a9ce3378d9a1489c1b30e
```

## 2.4 编写 `docker-compose.yml`

```yml
services:
  docker-in-docker:
    image: docker:dind
    container_name: docker_dind
    privileged: true
    command:
      - dockerd
      - -H 
      - unix:///var/run/docker.sock
      - -H
      - tcp://0.0.0.0:2375
      - --tls=false
      - --registry-mirror=https://docker.m.daocloud.io
      - --registry-mirror=https://docker.1ms.run
      - --registry-mirror=https://docker.xuanyuan.me
    volumes:
      - dind-data:/var/lib/docker
    stop_grace_period: 1m 
    healthcheck:
      test: ["CMD", "docker", "info"]
      interval: 5s
      timeout: 5s
      retries: 20
      start_period: 10s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    restart: unless-stopped

  runner:
    image: data.forgejo.org/forgejo/runner:13
    container_name: runner
    depends_on:
      docker-in-docker:
        condition: service_healthy           
    environment:
      DOCKER_HOST: tcp://docker-in-docker:2375
    user: 1001:1001
    volumes:
      - ./data:/data
    restart: unless-stopped
    command: "forgejo-runner daemon --config runner-config.yml"
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  dind-data:
```

## 2.5. job模版

### Frontend (React / Vue) — label `node`

```yaml
on: [push]
jobs:
  build:
    runs-on: node
    steps:
      - uses: actions/checkout@v4
      - run: corepack enable
      - run: pnpm install --frozen-lockfile    # or: npm ci
      - run: pnpm lint
      - run: pnpm test --run
      - run: pnpm build
```

### Java (Maven) — label `java`

```yaml
on: [push]
jobs:
  build:
    runs-on: java
    steps:
      - uses: actions/checkout@v4
      - run: mvn -B -ntp verify
```

### Java (Gradle wrapper) — label `java`

```yaml
on: [push]
jobs:
  build:
    runs-on: java
    steps:
      - uses: actions/checkout@v4
      - run: ./gradlew --no-daemon build
```

### Go — label `go`

```yaml
on: [push]
jobs:
  build:
    runs-on: go
    steps:
      - uses: actions/checkout@v4
      - run: go vet ./...
      - run: go test ./...
      - run: CGO_ENABLED=0 go build -o bin/app ./cmd/app
```

### Python — label `python`

```yaml
on: [push]
jobs:
  test:
    runs-on: python
    steps:
      - uses: actions/checkout@v4
      - run: python -m venv .venv && . .venv/bin/activate && pip install -r requirements.txt
      - run: . .venv/bin/activate && pytest -q
```

### Needs a database or Redis for tests?

Use `services:` in the job; it starts the containers on the same job network:

```yaml
jobs:
  test:
    runs-on: java
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
    steps:
      - uses: actions/checkout@v4
      - run: mvn -B -ntp verify
        env:
          DB_URL: jdbc:postgresql://postgres:5432/postgres
```

## Quick check that the shared cache works

Run any job twice: the second run should be much faster. You can also inspect the volume directly from the runner server:

```bash
docker compose exec docker-in-docker \
  docker run --rm -v ci-cache:/cache alpine du -sh /cache/*
```

# 3. 辅助

```shell
#!/usr/bin/env bash
# =============================================================================
# Builds the CI images referenced by the labels in runner-config.yml:
#   ci-java:21   ci-go:1.25   ci-python:3.13
# Each is an official toolchain image + Node.js (+ git/curl), because most
# actions (actions/checkout, ...) run on Node.js inside the job container.
#
# The images are built INSIDE the docker-in-docker daemon, so the runner finds
# them locally - no registry needed. Run from the directory with docker-compose.yml:
#
#     bash build-ci-images.sh
#
# Re-run it periodically (e.g. weekly via cron) to pick up security updates:
# the runner never re-pulls an image it already has.
# =============================================================================
set -euo pipefail
cd "$(dirname "$0")"

# Idempotent: start dind if it is down and wait until its healthcheck passes
docker compose up -d --wait docker-in-docker

# Dockerfile is passed on stdin, so there is no build context to send.
read -r -d '' DOCKERFILE <<'EOF' || true
ARG BASE
FROM node:24-bookworm-slim AS node
FROM ${BASE}
# Merge Node.js + npm/npx/corepack into the toolchain image
COPY --from=node /usr/local/bin/ /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
# git/curl/CA certs are needed by checkout and most build tools
RUN set -eux; \
    find /etc/apt -name '*.sources' -o -name 'sources.list' | \
      xargs -r sed -i 's|deb.debian.org|mirrors.aliyun.com|g; s|archive.ubuntu.com|mirrors.aliyun.com|g' || true; \
    apt-get update \
 && apt-get install -y --no-install-recommends git ca-certificates curl \
 && rm -rf /var/lib/apt/lists/* \
 && node --version && npm --version && corepack --version && git --version
EOF

build() {
  local tag="$1" base="$2"
  echo "==> Building ${tag} from ${base}"
  printf '%s\n' "$DOCKERFILE" |
    timeout 900 docker compose exec -T docker-in-docker \
      docker build --pull -t "$tag" --build-arg "BASE=${base}" -
}

declare -a FAILED=()
#     tag               base image (pick the versions you need)
build ci-java:25        eclipse-temurin:25-jdk-noble || FAILED+=("ci-java:25")
build ci-go:1.25        golang:1.25-bookworm         || FAILED+=("ci-go:1.25-bookworm")
build ci-python:3.15    python:3.15.2-bookworm       || FAILED+=("ci-python:3.15.2-bookworm")

docker compose exec -T docker-in-docker docker image prune -f

echo "==> Done. Images available to the runner:"
docker compose exec -T docker-in-docker docker images 'ci-*'
echo "==> ci-* tags referenced by runner-config.yml (keep in sync!):"
grep -o 'ci-[a-z]*:[0-9.]*' data/runner-config.yml | sort -u || true

if ((${#FAILED[@]} > 0)); then
  echo "!! Build FAILED for: ${FAILED[*]}" >&2
  exit 1
fi
```



