```yaml
spring:
  datasource:
    # connection-fetch: lazy  
    url: ${DATASOURCE_URL:jdbc:postgresql://localhost:5432/db}
    username: ${DATASOURCE_USERNAME:root}
    password: ${DATASOURCE_PASSWORD:root}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 30
      minimum-idle: 15
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      connection-test-query: SELECT 1
      validation-timeout: 5000
      leak-detection-threshold: 60000

  flyway:
    baseline-on-migrate: false
management:
  endpoint:
    health:
      show-details: never
      show-components: never
```