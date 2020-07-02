# SonarQube-docker-compose
```yaml

version: "2"

services:
  sonarqube:
    image: sonarqube
    restart: always
    ports:
      - "9000:9000"
    depends_on:
      - db
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.username=sonar
      - sonar.jdbc.password=sonar
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonarqube
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=sonar
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    volumes:
      - /srv/docker/sonarqube/sonarqube_conf:/opt/sonarqube/conf
      - /srv/docker/sonarqube/sonarqube_data:/opt/sonarqube/data
      - /srv/docker/sonarqube/sonarqube_extensions:/opt/sonarqube/extension
    ulimits:
      nofile:
        soft: "65536"
        hard: "65536"

  db:
    image: postgres
    restart: always
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonarqube
    volumes:
      - /srv/docker/sonarqube/postgresql:/var/lib/postgresql
      - /srv/docker/sonarqube/postgresql_data:/var/lib/postgresql/data

networks:
  sonarnet:
    driver: bridge
```
說明：

*在`db`中需要明確定義Postgres數據庫的用戶名、密碼和數據庫名

*在`sonarqube`中的數據庫的用戶名、密碼和數據庫名要和db中的定義保持一致

*由于`sonarqube`镜像有bug，需要同时用`sonar.jdbc.xxx`和S`ONARQUBE_JDBC_XXX`指定数据库的用户名、密码和数据库名（否则会出现仍然使用默认的H2数据库的问题，或者打开SonarQube后发现Rules和Quality Profile为空的问题）

*由於新版Sonarqube似乎有max virtual memory areas跟max file descriptors問題，錯誤訊息如下
    
    sonarqube_1  | ERROR: [1] bootstrap checks failed
    sonarqube_1  | [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    sonarqube_1  | [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

virtual memory解決方法：

```
sudo sysctl -w vm.max_map_count=262144
```

file descriptorsr解決方法：
在yaml加上`ulimits:`

