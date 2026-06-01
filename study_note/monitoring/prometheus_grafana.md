# JAVA端配置

1. 在application.yml添加配置:

~~~YAML
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
      enabled: true  
    health:  
      show-details: always  
      show-components: always  
  health:  
    db:  
      enabled: true  
  metrics:  
    tags:  
      application: ${spring.application.name:my-app}  
      environment: ${spring.profiles.active:dev}
~~~

2. 在build.gradle添加新依赖

~~~gradle
implementation 'io.micrometer:micrometer-registry-prometheus:1.17.0-RC1'
~~~


# DOCKER配置

 1. docker-compose配置

[[docker_template#prometheus+grafana]]

2. Prometheus

#### 配置

~~~YAML
scrape_configs:
  - job_name: 'java-template' # 随便起个名字
    scrape_interval: 5s # 每 5 秒拉取一次
    metrics_path: '/api/actuator/prometheus' # 你的 Spring Boot 暴露的接口路径
    static_configs:
      - targets: ['localhost:8001'] # 你 Spring Boot 的地址
      # - targets: ['host.docker.internal:8001'] 如果是docker部署,使用这个,端口改成后端的ip就行  
~~~

#### 验证
浏览器访问 `http://localhost:9090`，如果能看到 Prometheus 的界面，说明采集器启动成功。

3. Grafana

#### 验证
浏览器访问 `http://localhost:3000`。默认账号密码都是 `admin / admin123`

### 配置

在 Grafana 界面操作：

1. 鼠标悬停左侧边栏的 ⚙️（齿轮图标） -> 点击 **Data Sources**。
2. 点击 **Add data source** -> 选择 **Prometheus**。
3. **关键配置**：在 URL 那一栏填入：`http://localhost:9090` （别加斜杠,如果是docker部署的用http://prometheus:9090）。其他都不用动。
4. 页面底部点击 **Save & test**。如果出现绿色的 `Data source is working`，说明大功告成！
5. 模版id: 12900