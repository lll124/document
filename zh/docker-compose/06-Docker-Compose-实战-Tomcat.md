# Docker Compose 实战 Tomcat
```
version: '3.1'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - /usr/local/docker/tomcat/webapps/test:/usr/local/tomcat/webapps/test
    environment:
      TZ: Asia/Shanghai
```


```
version: '3.1'
services:
  tomcat:
    restart: always
    image: tomcat:8.5.64-jdk8
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - /usr/local/docker/postgresql_scws/web/vvdd:/usr/local/tomcat/webapps/ROOT
    environment:
      TZ: Asia/Shanghai
```