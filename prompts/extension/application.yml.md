```yaml
spring:  
  config:  
    import: "optional:file:./.env[.properties]"  
  messages:  
    basename: i18n/messages  
    encoding: UTF-8  
    use-code-as-default-message: true  
    cache-duration: 3600s  
  
  mail:  
    host: smtp.qq.com  
    port: 587  
    username: ${MAIL_USERNAME}  
    password: ${MAIL_PASSWORD}  
    properties:  
      mail:  
        smtp:  
          starttls:  
            enable: true  
            required: true  
  flyway:  
    enabled: true  
    table: app_schema_history  
    validate-on-migrate: true  
    clean-disabled: true  
  
  application:  
    name: java-template  
  
  threads:  
    virtual:  
      enabled: true  
  lifecycle:  
    timeout-per-shutdown-phase: 30s  
  
  jackson:  
    time-zone: Asia/Shanghai  
    date-format: yyyy-MM-dd HH:mm:ss  
    default-property-inclusion: NON_EMPTY  
    serialization:  
      fail-on-empty-beans: false # only ture in dev env  
    use-jackson2-defaults: false  
  
  main:  
    log-startup-info: true  
    banner-mode: off  
  
mybatis-plus:  
  type-handlers-package: com.github.starhq.template.config.mybatis.handler  
  mapper-locations: classpath*:/mapper/**/*.xml  
  typeAliasesPackage: com.github.starhq.template.entity  
  global-config:  
    db-config:  
      id-type: ASSIGN_ID  
    banner: false  
  configuration:  
    map-underscore-to-camel-case: true  
    cache-enabled: false  
    call-setters-on-nulls: true  
    jdbc-type-for-null: "NULL"  
  
server:  
  servlet:  
    context-path: /api  
  port: 8001  
  shutdown: graceful  
  forward-headers-strategy: native  
  tomcat:  
    uri-encoding: UTF-8  
  compression:  
    enabled: true  
    mime-types:  
      - "application/json"  
    min-response-size: 1024  
  
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
  
logging:  
  level:  
    com.github.starhq.template.config.security.filter: INFO  
    org.springframework.security.config.annotation.authentication.configuration.InitializeUserDetailsBeanManagerConfigurer: ERROR  
  
star:  
  jwt:  
    # 强烈建议使用环境变量，避免密钥泄露  
    key: ${JWT_KEY}  
    access: 30m  
    refresh: 336h  
  
  white-list:  
    - /auth/login  
    - /auth/captcha  
    - /actuator/**  
    - /error  
       
  cors:  
    allowed-origin-patterns:  
      - "https://your-frontend-domain.com"  
    allowed-methods:  
      - "*"   
    allowed-headers:  
          - "*"   
    allow-credentials: true   
    max-age: 3600  
  
  captcha:  
    type: line  
    width: 120  
    height: 40  
    codeCount: 5  
    interfereCount: 170   
  
  sensitive:  
    fields:  
      - password  
      - oldPassword  
      - newPassword  
      - confirmPassword  
      - token  
      - secret  
      - accessToken  
      - refreshToken  
      - authorization  
      - cookie  
      - set-cookie  
    headers:  
      - Authorization  
      - Content-Type  
      - Accept  
      - User-Agent  
      - X-Device-Fingerprint  
      - Referer  
      - Origin  
      - Host  
    maskValue: "*****"
```