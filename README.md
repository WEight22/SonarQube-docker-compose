# SonarQube-docker-compose
```yaml

version: "2"

services:
  sonarqube:
    image: sonarqube:6.7.1
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
      - /srv/docker/sonarqube/sonarqube_extensions:/opt/sonarqube/extensions

  db:
    image: postgres:9.6
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
*在`db`中需要明確定義Postgres數據庫的用戶名、密碼和數據庫名,
*在`sonarqube`中的數據庫的用戶名、密碼和數據庫名要和db中的定義保持一致,
*由于`sonarqube`镜像有bug，需要同时用`sonar.jdbc.xxx`和S`ONARQUBE_JDBC_XXX`指定数据库的用户名、密码和数据库名（否则会出现仍然使用默认的H2数据库的问题，或者打开SonarQube后发现Rules和Quality Profile为空的问题）,

另外，`sonarqube`和`postgres`镜像也有版本兼容问题，经测试的兼容版本包括：,

`sonarqube:6.7.1` and `postgres:9.6`,
`sonarqube:6.4` and `postgres:9.4`,
`sonarqube:7.0` and `postgres:9.6`,
详细的`docker-compose.yml`和一键安装脚本参见：,
https://github.com/cookcodeblog/OneDayDevOps/tree/master/components/sonarqube
————————————————
版权声明：本文为CSDN博主「nklinsirui」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/nklinsirui/java/article/details/90405159
