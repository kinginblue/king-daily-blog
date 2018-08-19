# Spring XXX Security 依赖关系图

| 项目名 | 顶级包名 |
|:-|:-|
| spring-cloud-starter-oauth2 | org.springframework.cloud |
| ---- spring-cloud-starter-security | org.springframework.cloud |
| ---- spring-security-oauth2 | org.springframework.security.oauth |
| ---- spring-security-jwt | org.springframework.security |
| . |  |
| spring-cloud-starter-security | org.springframework.cloud |
| ---- spring-cloud-security | org.springframework.cloud |
| . |  |
| spring-cloud-security | org.springframework.cloud |
| ---- spring-boot-starter-security | org.springframework.boot |
| ---- spring-security-oauth2(optional) | org.springframework.security.oauth |
| . |  |
| spring-boot-starter-security | org.springframework.boot |
| ---- spring-security-config | org.springframework.security |
| ---- spring-security-web | org.springframework.security |
| - |  |
| spring-security-oauth2 | org.springframework.security.oauth |
| ---- spring-security-core | org.springframework.security |
| ---- spring-security-config | org.springframework.security |
| ---- spring-security-jwt(optional) | org.springframework.security |
| ---- spring-security-web | org.springframework.security |
| . |  |
| spring-security-web | org.springframework.security |
| ---- spring-security-core | org.springframework.security |
| . |  |
| spring-security-config | org.springframework.security |
| ---- spring-security-core | org.springframework.security |
| ---- spring-security-ldap(optional) | org.springframework.security |
| ---- spring-security-messaging(optional) | org.springframework.security |
| ---- spring-security-openid(optional) | org.springframework.security |
| ---- spring-security-web(optional) | org.springframework.security |
| ---- spring-security-aspects | org.springframework.security |
| ---- spring-security-cas | org.springframework.security |
