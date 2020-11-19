

Dockerfile

```
FROM maven:3.6.3-jdk-11
ARG JAVA_AGENT_BRANCH=master
ARG JAVA_AGENT_REPO=elastic/apm-agent-java

WORKDIR /usr/src/java-app

#build the application
ADD . /usr/src/java-code
WORKDIR /usr/src/java-code/opbeans

#Bring the latest frontend code
COPY --from=opbeans/opbeans-frontend:latest /app/build src/main/resources/public

RUN mvn -q --batch-mode package \
  -DskipTests \
  -Dmaven.repo.local=.m2 \
  --no-transfer-progress \
  -Dmaven.wagon.http.retryHandler.count=3 \
  -Dhttps.protocols=TLSv1.2 \
  -Dhttp.keepAlive=false \
  -Dmaven.javadoc.skip=true \
  -DskipTests=true \
  -Dmaven.gitcommitid.skip=true
RUN cp -v /usr/src/java-code/opbeans/target/*.jar /usr/src/java-app/app.jar

#build the agent
WORKDIR /usr/src/java-agent-code
RUN curl -L https://github.com/$JAVA_AGENT_REPO/archive/$JAVA_AGENT_BRANCH.tar.gz | tar --strip-components=1 -xz
RUN mvn -q --batch-mode clean package \
  -Dmaven.repo.local=.m2 \
  --no-transfer-progress \
  -Dmaven.wagon.http.retryHandler.count=3 \
  -Dhttps.protocols=TLSv1.2 \
  -Dhttp.keepAlive=false \
  -Dmaven.javadoc.skip=true \
  -DskipTests=true \
  -Dmaven.gitcommitid.skip=true

RUN export JAVA_AGENT_BUILT_VERSION=$(mvn -q -Dexec.executable="echo" -Dexec.args='${project.version}' --non-recursive org.codehaus.mojo:exec-maven-plugin:1.3.1:exec) \
    && cp -v /usr/src/java-agent-code/elastic-apm-agent/target/elastic-apm-agent-${JAVA_AGENT_BUILT_VERSION}.jar /usr/src/java-app/elastic-apm-agent.jar


#Run application Stage
#We only need java

FROM adoptopenjdk:11-jre-hotspot

RUN export
RUN apt-get -qq update \
 && apt-get install --no-install-recommends -y -qq curl \
 && rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true
WORKDIR /app
COPY --from=0 /usr/src/java-app/*.jar ./

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.vendor="Elastic" \
    org.label-schema.name="opbeans-java" \
    org.label-schema.version="1.18.1" \
    org.label-schema.url="https://hub.docker.com/r/opbeans/opbeans-java" \
    org.label-schema.vcs-url="https://github.com/elastic/opbeans-java" \
    org.label-schema.license="MIT"

CMD java -javaagent:/app/elastic-apm-agent.jar -Dspring.profiles.active=${OPBEANS_JAVA_PROFILE:-}\
                                        -Dserver.port=${OPBEANS_SERVER_PORT:-}\
                                        -Dserver.address=${OPBEANS_SERVER_ADDRESS:-0.0.0.0}\
                                        -Dspring.datasource.url=${DATABASE_URL:-}\
                                        -Dspring.datasource.driverClassName=${DATABASE_DRIVER:-}\
                                        -Dspring.jpa.database=${DATABASE_DIALECT:-}\
                                        -jar /app/app.jar





```































docker-compose.yml


```

version: "3"
services:
  opbeans-java:
    build: .
    image: opbeans/opbeans-java:latest
    container_name: opbeans-java
    ports:
      - "127.0.0.1:${OPBEANS_SERVER_PORT:-8000}:8000"
    logging:
      driver: 'json-file'
      options:
          max-size: '2m'
          max-file: '5'
    environment:
      - ELASTIC_APM_SERVICE_NAME=${ELASTIC_APM_SERVICE_NAME:-opbeans-java}
      - ELASTIC_APM_SERVER_URL=${ELASTIC_APM_SERVER_URL:-http://apm-server:8200}
      - ELASTIC_APM_APPLICATION_PACKAGES=co.elastic.apm.opbeans
      - ELASTIC_APM_JS_SERVER_URL=${ELASTIC_APM_JS_SERVER_URL:-http://localhost:8200}
      - OPBEANS_SERVER_PORT=${OPBEANS_SERVER_PORT:-8000}
      - ELASTIC_APM_ENABLE_LOG_CORRELATION=true
      - ELASTIC_APM_ENVIRONMENT=production
    depends_on:
      apm-server:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "--write-out", "'HTTP %{http_code}'", "--silent", "--output", "/dev/null", "http://opbeans-java:8000/"]
      interval: 10s
      retries: 10

  apm-server:
    image: docker.elastic.co/apm/apm-server:${STACK_VERSION:-7.3.0}
    ports:
      - "127.0.0.1:${APM_SERVER_PORT:-8200}:8200"
      - "127.0.0.1:${APM_SERVER_MONITOR_PORT:-6060}:6060"
    command: >
      apm-server -e
        -E apm-server.frontend.enabled=true
        -E apm-server.frontend.rate_limit=100000
        -E apm-server.host=0.0.0.0:8200
        -E apm-server.read_timeout=1m
        -E apm-server.shutdown_timeout=2m
        -E apm-server.write_timeout=1m
        -E apm-server.rum.enabled=true
        -E setup.kibana.host=kibana:5601
        -E setup.template.settings.index.number_of_replicas=0
        -E xpack.monitoring.elasticsearch=true
        -E output.elasticsearch.enabled=${APM_SERVER_ELASTICSEARCH_OUTPUT_ENABLED:-true}
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_OVERRIDE
      - SETGID
      - SETUID
    logging:
      driver: 'json-file'
      options:
          max-size: '2m'
          max-file: '5'
    depends_on:
      elasticsearch:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "--write-out", "'HTTP %{http_code}'", "--silent", "--output", "/dev/null", "http://apm-server:8200/"]
      retries: 10
      interval: 10s

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION:-7.3.0}
    environment:
      - cluster.name=docker-cluster
      - xpack.security.enabled=false
      - bootstrap.memory_lock=true
      - network.host=0.0.0.0
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - "path.data=/usr/share/elasticsearch/data/${STACK_VERSION:-7.3.0}"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 2g
    logging:
      driver: 'json-file'
      options:
          max-size: '2m'
          max-file: '5'
    ports:
      - "127.0.0.1:${ELASTICSEARCH_PORT:-9200}:9200"
    healthcheck:
      test: ["CMD-SHELL", "curl -s http://localhost:9200/_cluster/health | grep -vq '\"status\":\"red\"'"]
      retries: 10
      interval: 20s
    volumes:
      - esdata:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION:-7.3.0}
    environment:
      SERVER_NAME: kibana.example.org
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - "127.0.0.1:${KIBANA_PORT:-5601}:5601"
    logging:
      driver: 'json-file'
      options:
          max-size: '2m'
          max-file: '5'
    healthcheck:
      test: ["CMD", "curl", "--write-out", "'HTTP %{http_code}'", "--silent", "--output", "/dev/null", "http://kibana:5601/"]
      retries: 10
      interval: 10s
    depends_on:
      elasticsearch:
        condition: service_healthy

  wait:
    image: busybox
    depends_on:
      opbeans-java:
        condition: service_healthy
        
volumes:
  esdata:
    driver: local






```
