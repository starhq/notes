```xml
<?xml version="1.0" encoding="utf-8"?>  
  
<!--<configuration scan="true" scanPeriod="60 seconds">-->  
<configuration>  
  <!-- ============================== -->  
  <!-- Context and Properties -->  <!-- ============================== -->  <!-- Spring Properties -->  
  <springProperty scope="context" name="mailHost" source="spring.mail.host" defaultValue="unknown-host"/>  
  <springProperty scope="context" name="mailPort" source="spring.mail.port" defaultValue="587"/>  
  <springProperty scope="context" name="mailUsername" source="spring.mail.username" defaultValue="unknown-user"/>  
  <springProperty scope="context" name="mailPassword" source="spring.mail.password" defaultValue="unknown-pwd"/>  
  <springProperty scope="context" name="mailTo" source="spring.mail.to" defaultValue="starimba@outlook.com"/>  
  <springProperty scope="context" name="appName" source="spring.application.name" defaultValue="java-template"/>  
  <!-- Name used to identify this logging context -->  
  <contextName>${appName}</contextName>  
  <!-- Log Path Configuration -->  
  <property name="LOG_BASE_PATH" value="${LOG_PATH:-${user.home}/workspace/java/log}"/>  
  <!-- Log Patterns -->  
  <property name="FILE_LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %X{traceId:-} %logger{40} - %msg%n"/>  
  <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%-5level) %clr([%thread]){faint} %clr(%X{traceId:-}){magenta} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %highlight(%msg)\n%wEx"/>  
  <!-- Rolling Policy Configuration -->  
  <property name="MAX_FILE_SIZE" value="100MB"/>  
  <!-- Recommended: 100MB to 250MB -->  
  <property name="MAX_HISTORY" value="30"/>  
  <!-- Recommended: 7 to 30 days -->  
  <property name="TOTAL_SIZE_CAP" value="5GB"/>  
  <!-- Recommended: 5GB to 10GB+ based on disk space -->  
  <!-- Color Conversion Rules -->  
  <conversionRule conversionWord="clr" class="org.springframework.boot.logging.logback.ColorConverter"/>  
  <conversionRule conversionWord="wEx" class="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>  
  <!-- Disable Logback Status Messages -->  
  <statusListener class="ch.qos.logback.core.status.NopStatusListener"/>  
  <!-- ============================== -->  
  <!-- Console Appender -->  <!-- ============================== -->  
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder>  
      <pattern>${CONSOLE_LOG_PATTERN}</pattern>  
      <charset>UTF-8</charset>  
    </encoder>  
  </appender>  
  <!-- ============================== -->  
  <!-- Logger Configuration -->  <!-- ============================== -->  <!-- Framework Loggers -->  
  <logger name="org.springframework" level="WARN"/>  
  <logger name="org.springframework.boot" level="WARN"/>  
  <logger name="org.springframework.security" level="WARN"/>  
  <logger name="org.redisson" level="WARN"/>  
  <logger name="io.netty" level="WARN"/>  
  <logger name="org.mybatis" level="WARN"/>  
  <logger name="org.flywaydb" level="WARN"/>  
  <logger name="io.undertow" level="WARN"/>  
  <logger name="org.xnio" level="WARN"/>  
  <logger name="org.apache" level="WARN"/>  
  <logger name="org.hibernate" level="WARN"/>  
  <logger name="org.jboss" level="WARN"/>  
  <logger name="com.alibaba" level="WARN"/>  
  <logger name="io.springfox" level="WARN"/>  
  <logger name="org.quartz" level="WARN"/>  
  <logger name="org.thymeleaf" level="WARN"/>  
  <logger name="com.zaxxer.hikari" level="WARN"/>  
  <logger name="com.baomidou" level="ERROR"/>  
  <logger name="com.github.pagehelper" level="WARN"/>  
  <logger name="ch.qos.logback" level="ERROR"/>  
  <!-- <logger name="com.github.starhq.app.mapper" level="INFO"/>  
    <logger name="com.github.starhq.app.config.spring.security.filter" level="INFO"/> -->  <!-- ============================== -->  <!-- Profile-Specific Configuration -->  <!-- ============================== -->  <!-- Production Profile -->  
    <springProfile name="prod">  
    <!-- ============================== -->  
    <!-- Email Appender -->    <!-- ============================== -->    
    <appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">  
      <smtpHost>${mailHost}</smtpHost>  
      <smtpPort>${mailPort}</smtpPort>  
      <username>${mailUsername}</username>  
      <password>${mailPassword}</password>  
      <STARTTLS>true</STARTTLS>  
      <asynchronousSending>false</asynchronousSending>  
      <to>${mailTo}</to>  
      <from>${mailUsername}</from>  
      <subject>[${appName}] ERROR: %logger{20} - %msg{50}</subject>  
      <layout class="ch.qos.logback.classic.html.HTMLLayout">  
        <pattern>%d{yyyy-MM-dd HH:mm:ss}%level%thread%X{traceId}%logger{36}%msg</pattern>  
      </layout>  
      <cyclicBufferTracker class="ch.qos.logback.core.spi.CyclicBufferTracker">  
        <bufferSize>10</bufferSize>  
      </cyclicBufferTracker>  
      <evaluator class="ch.qos.logback.classic.boolex.OnErrorEvaluator"/>  
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">  
        <level>ERROR</level>  
      </filter>  
    </appender>  
    <!-- ============================== -->  
    <!-- File Appenders -->    <!-- ============================== -->    <!-- Main Application Log (INFO and above, excluding ERROR) -->    
    <appender name="FILE_APP" class="ch.qos.logback.core.rolling.RollingFileAppender">  
      <file>${LOG_BASE_PATH}/${appName}/${appName}.log</file>  
      <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
        <fileNamePattern>${LOG_BASE_PATH}/${appName}/archived/${appName}-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>  
        <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>  
        <maxHistory>${MAX_HISTORY}</maxHistory>  
        <totalSizeCap>${TOTAL_SIZE_CAP}</totalSizeCap>  
        <cleanHistoryOnStart>false</cleanHistoryOnStart>  
      </rollingPolicy>  
      <encoder>        <pattern>${FILE_LOG_PATTERN}</pattern>  
        <charset>UTF-8</charset>  
      </encoder>  
      <!-- Exclude ERROR level -->  
      <!--<filter class="ch.qos.logback.classic.filter.LevelFilter">        <level>ERROR</level>        <onMatch>DENY</onMatch>        <onMismatch>NEUTRAL</onMismatch>      </filter>-->    
      </appender>  
    <!-- Error Log (ERROR level only) -->  
    <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">  
      <file>${LOG_BASE_PATH}/${appName}/${appName}-error.log</file>  
      <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
        <fileNamePattern>${LOG_BASE_PATH}/${appName}/errors/${appName}-error-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>  
        <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>  
        <maxHistory>${MAX_HISTORY}</maxHistory>  
        <totalSizeCap>${TOTAL_SIZE_CAP}</totalSizeCap>  
      </rollingPolicy>  
      <encoder>        <pattern>${FILE_LOG_PATTERN}</pattern>  
        <charset>UTF-8</charset>  
      </encoder>  
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">  
        <level>ERROR</level>  
      </filter>  
    </appender>  
    <!-- ============================== -->  
    <!-- Async Appenders -->    <!-- ============================== -->    
    <appender name="ASYNC_FILE_APP" class="ch.qos.logback.classic.AsyncAppender">  
      <appender-ref ref="FILE_APP"/>  
      <queueSize>1000</queueSize>  
      <discardingThreshold>20</discardingThreshold>  
      <includeCallerData>false</includeCallerData>  
      <neverBlock>true</neverBlock>  
    </appender>  
    <appender name="ASYNC_FILE_ERROR" class="ch.qos.logback.classic.AsyncAppender">  
      <appender-ref ref="FILE_ERROR"/>  
      <queueSize>256</queueSize>  
      <discardingThreshold>0</discardingThreshold>  
      <includeCallerData>true</includeCallerData>  
      <neverBlock>false</neverBlock>  
    </appender>  
    <appender name="ASYNC_EMAIL" class="ch.qos.logback.classic.AsyncAppender">  
      <appender-ref ref="EMAIL"/>  
      <queueSize>50</queueSize>  
      <discardingThreshold>0</discardingThreshold>  
      <includeCallerData>true</includeCallerData>  
      <neverBlock>true</neverBlock>  
    </appender>  
    <root level="INFO">  
      <appender-ref ref="CONSOLE"/>  
      <appender-ref ref="ASYNC_FILE_APP"/>  
      <appender-ref ref="ASYNC_FILE_ERROR"/>  
      <appender-ref ref="ASYNC_EMAIL"/>  
    </root>  
    <jmxConfigurator/>  </springProfile>  
  <!-- Development Profile -->  
  <springProfile name="dev">  
    <root level="INFO">  
      <appender-ref ref="CONSOLE"/>  
    </root>  
    <!-- 业务代码开启 DEBUG，直接输出到控制台 -->  
    <logger name="com.github.starhq" level="DEBUG" additivity="false">  
      <appender-ref ref="CONSOLE"/>  
    </logger>  
  </springProfile>  
  <!-- Test Profile -->  
  <springProfile name="test">  
    <root level="INFO">  
      <appender-ref ref="CONSOLE"/>  
    </root>  
    <logger name="com.github.starhq" level="DEBUG" additivity="false">  
      <appender-ref ref="CONSOLE"/>  
    </logger>  
  </springProfile>  
</configuration>
```