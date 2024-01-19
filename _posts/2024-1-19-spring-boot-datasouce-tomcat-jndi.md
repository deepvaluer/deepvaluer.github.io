---
layout: post
title: Configuring Datasource with Spring Boot Embedded Tomcat JNDI
categories: [TOMCAT,SPRINGBOOT,JNDI]
excerpt: Spring Boot 내장 톰캣 JNDI를 이용한 Datasource 설정
---

Spring Boot 경우 대게 프로퍼티 설정을 통해 Datasource를 설정 하지만 내장 톰캣의 JNDI를 이용해 Datasource를 설정하는 방법을 공유합니다.

### 적용 환경
`Spring Boot 2.7.9`  

### Maven Dependency 추가  
``` xml
<dependency>
  <groupId>org.apache.tomcat</groupId>
  <artifactId>tomcat-jdbc</artifactId>
</dependency>        
```

### Tomcat configuration  
``` java
import org.apache.catalina.Container;
import org.apache.catalina.core.NamingContextListener;
import org.apache.catalina.core.StandardHost;
import org.apache.tomcat.util.descriptor.web.ContextResource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class TomcatConfig implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    @Value("${spring.datasource.hikari.username}")
    private String username;

    @Value("${spring.datasource.hikari.password}")
    private String password;

    @Value("${spring.datasource.hikari.jdbc-url}")
    private String jdbcUrl;

    @Value("${spring.datasource.driver-class-name}")
    private String driverClassName;

    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.addContextCustomizers(context -> {
            ContextResource resource = new ContextResource();
            resource.setName("jndiName");
            resource.setAuth("Container");
            resource.setType("org.apache.tomcat.jdbc.pool.DataSource");
            resource.setProperty("factory", "org.apache.tomcat.jdbc.pool.DataSourceFactory");
            resource.setProperty("driverClassName", driverClassName);
            resource.setProperty("url", jdbcUrl);
            resource.setProperty("username", username);
            resource.setProperty("password", password);

            resource.setProperty("validationQuery", "SELECT 1 FROM DUAL");
            resource.setProperty("validationInterval", "60000");
            resource.setProperty("testWhileIdle", "true");
            resource.setProperty("testOnBorrow", "true");
            resource.setProperty("testOnReturn", "false");

            context.getNamingResources().addResource(resource);
        });

        enableNaming(factory);
    }

    private void enableNaming(TomcatServletWebServerFactory server) {
        server.addContextLifecycleListeners(new NamingContextListener());

        // The following code is copied from Tomcat
        System.setProperty("catalina.useNaming", "true");
        String value = "org.apache.naming";
        String oldValue = System.getProperty("java.naming.factory.url.pkgs");
        if (oldValue != null) {
            if (oldValue.contains(value)) {
                value = oldValue;
            } else {
                value = value + ":" + oldValue;
            }
        }

        System.setProperty("java.naming.factory.url.pkgs", value);
        value = System.getProperty("java.naming.factory.initial");
        if (value == null) {
            System.setProperty("java.naming.factory.initial", "org.apache.naming.java.javaURLContextFactory");
        }
    }
}
```
