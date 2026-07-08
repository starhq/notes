```yaml
spring:  
  jackson:  
    serialization:  
      fail-on-empty-beans: true  
  output:  
    ansi:  
      enabled: always  
  
  flyway:  
    baseline-on-migrate: true  
  
star:  
  cors:  
    allowed-origin-patterns:  
      - "http://localhost:3000"  
      - "http://localhost:8080"  
    max-age: 5
```