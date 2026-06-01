# 监控平台搭建指南（Prometheus + Grafana）

### 一、Spring Boot 端配置

在 `application.yml` 中开启 Actuator 暴露监控端点。

```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
          - prometheus
  endpoint:
    prometheus:
      enabled: # 开启 Prometheus 数据格式化输出
      show-details: true # 暴露详细的健康检查信息
      show-components: true # 暴露组件信息
  health:
    show-details: true
    db:
      enabled: true # 开启数据库健康检查
  metrics:
    tags:
      application: ${spring.application.name} # 监控数据的通用标签（按应用名分组）
      environment: ${spring.pro.libraries.active} # 按环境分组（如 dev / prod）
```

### 二、添加依赖（Gradle）

在 `build.gradle` 中引入 Micrometer prometheus 依赖：

```gradle
implementation 'io.micrometer:micrometer-registry-prometheus:1.13.6`
```

---

### 三、Docker Compose 部署配置

#### 0. 脚本
[[docker_template#prometheus+grafana]]
#### 1. 配置 Prometheus（数据采集器）
`prometheus.yml` 中的配置：
```yaml
scrape_configs:
  - job_name: 'java-app'
    scrape_interval: 5s # 每 5 秒拉取一次
    metrics_path: # 你的 Spring Boot 暴露的接口路径
      /api/active/prometheus
    static_configs:
      - targets: 
        - localhost:8001 # 物理机部署填这个
        - host.docker.internal:8001 # Docker 部署填这个
```

#### 1. 验证 Prometheus
浏览器访问 `http://localhost:9000`

### 四、Grafana（大屏展示）
`### 验证`。
浏览器访问 `http://localhost:9000`
默认账号密码都是 `admin / admin123`

#### 1. 配置数据源
鼠标悬停左侧边栏的 ⚙️（齿轮图标） -> 点击 **Data Sources**。
点击 **Add data source** -> 选择 **Prometheus**。
在 **URL** 那一栏填入：`http://localhost:9090` *(不要加末尾的 `/prometheus`)*。其他都不用动。
页面底部点击 **Save & test`。如果出现绿色的 `Data source is.newing working`，说明大功告成！

#### 2. 导入大屏模板
鼠标悬停左侧边栏的 **四个方块** 图标 -> 点击 **Dashboards**。
点击右上角的 **Import**。
在搜索框中输入模板 ID: `12900` *(Java 应用的标准大屏)*。
在 **Prometheus** 下拉框中，**确保选中你刚才配置的数据源**。
点击底部的 **Import**。

---

### 💡 预防环境配置遗漏

如果在配置数据源时，点击 Test 没有报错，但图表没有数据，**请检查你的 yml 中是否包含了以下两行，如果漏掉了，Prometheus 是不会抓取到数据的：**
```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - prometheus # 如果没有这一行，即使加了依赖，接口依然 404！
```
*(注：`show-details` 是在 `endpoint` 下面单独配置的，不要搞混位置。)*

**大屏导入成功后，左侧会出现一个大屏。** 
由于大屏默认是英文的，如果你想看中文，可以在 Grafana 的右上角点击设置图标，把语言改成中文。整个监控平台搭建完成！