
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
    SONARQUBE_JDBC_URL: jdbc:mysql://((hostname)):((port))/((name))?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
EOF
```

```
cf push --random-route --vars-file credentials.yml
```

## Installing plugins

Installing plugins via Marketplace doesn't work well since download files exist in the ephemeral disk and will disapper when the container is restarted.
You can use [Manual Installation](https://docs.sonarqube.org/display/SONAR/Installing+a+Plugin#InstallingaPlugin-ManualInstallation).
You need to have plugins before sonar starts and place them into `/opt/sonarqube/extensions/plugins`.

Here is an example to install [SonarJava](https://docs.sonarqube.org/display/PLUG/SonarJava).

```yaml
cat <<EOF > manifest.yml
applications:
- name: sonar
  memory: 2g
  health-check-type: http
  health-check-http-endpoint: /
  command: |
    wget https://sonarsource.bintray.com/Distribution/sonar-java-plugin/sonar-java-plugin-5.7.0.15470.jar -O /opt/sonarqube/extensions/plugins/sonar-java-plugin-5.7.0.15470.jar && \
    ./bin/run.sh 
  docker:
    image: sonarqube
  env:
    SONARQUBE_JDBC_USERNAME: ((username))
    SONARQUBE_JDBC_PASSWORD: ((password))
    SONARQUBE_JDBC_URL: jdbc:mysql://((hostname)):((port))/((name))?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
EOF
```