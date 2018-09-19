
```
cf create-service <your-mysql-service> <your-mysql-plan> sonar-db
# Sonar support MySQL 5.6+
# e.g. cf create-service p.mysql db-small sonar-db
cf create-service-key sonar-db sonar
cf service-key sonar-db sonar | awk 'NR>2 {print}' | ruby -ryaml -rjson -e 'puts YAML.dump(JSON.load(ARGF))' > credentials.yml
```

```yaml
cat <<EOF > manifest.yml
applications:
- name: sonar
  memory: 2g
  health-check-type: http
  health-check-http-endpoint: /
  docker:
    image: sonarqube
  env:
    SONARQUBE_JDBC_USERNAME: ((username))
    SONARQUBE_JDBC_PASSWORD: ((password))
    SONARQUBE_JDBC_URL: jdbc:mysql://((hostname)):((port))/((name))?useUnicode=true&characterEncoding=utf8&useSSL=false
EOF
```

```
cf push --random-route --vars-file credentials.yml
```