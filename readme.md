Add Strimzi OAuth to Kafka confluentinc
=======================================

Strimzi Kafka OAuth provides token-based authorization using Keycloak as authorization server, and taking advantage of [Keycloak Authorization Services](https://www.keycloak.org/docs/latest/authorization_services/).

Building
--------
    git clone https://github.com/strimzi/strimzi-kafka-oauth.git
    mvn clean install

Create a volume with strimizi ouath jars
----------------------------------------

Copy the following jars into your Kafka strimizi ouath volume that will be mounted later on:

    oauth-common/target/kafka-oauth-common-*.jar
    oauth-server/target/kafka-oauth-server-*.jar
    oauth-server-plain/target/kafka-oauth-server-plain-*.jar
    oauth-keycloak-authorizer/target/kafka-oauth-keycloak-authorizer-*.jar
    oauth-client/target/kafka-oauth-client-*.jar
    oauth-common/target/lib/nimbus-jose-jwt-*.jar
    oauth-server/target/lib/json-path-*.jar
    oauth-server/target/lib/json-smart-*.jar
    oauth-server/target/lib/accessors-smart-*.jar

Configuration of kafka confluentinc per services
================================================
cp-kafka
--------

modify the following block of the file cp-kafka/templates/statefulset.yaml as :

        env:
        - name: SSL_TRUSTSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_TRUSTSTORE_PASSWORD
        - name: CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: kafka-client-jaas-secret
              key: CLIENT_ID
        - name: CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: kafka-client-jaas-secret
              key: CLIENT_SECRET
        - name: KAFKA_INTER_BROKER_LISTENER_NAME
          value: REPLICATION
        - name: KAFKA_SASL_ENABLED_MECHANISMS
          value: OAUTHBEARER
        - name: KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL
          value: OAUTHBEARER
        - name: KAFKA_LISTENER_NAME_CLIENT_OAUTHBEARER_SASL_JAAS_CONFIG
          value: >
                  org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                  oauth.valid.issuer.uri="https://keycloak:8443/realms/demo"
                  oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                  oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)"
                  oauth.jwks.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/certs"
                  oauth.username.claim="preferred_username"
                  unsecuredLoginStringClaim_sub="unused";
        - name: KAFKA_LISTENER_NAME_REPLICATION_OAUTHBEARER_SASL_JAAS_CONFIG
          value: >
                  org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                  oauth.client.id="$(CLIENT_ID)"
                  oauth.client.secret="$(CLIENT_SECRET)"
                  oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                  oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                  oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)"
                  oauth.valid.issuer.uri="https://keycloak:8443/realms/demo"
                  oauth.jwks.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/certs"
                  oauth.username.claim="preferred_username";
        - name: KAFKA_LISTENER_NAME_CLIENT_OAUTHBEARER_SASL_SERVER_CALLBACK_HANDLER_CLASS
          value: io.strimzi.kafka.oauth.server.JaasServerOauthValidatorCallbackHandler
        - name: KAFKA_LISTENER_NAME_REPLICATION_OAUTHBEARER_SASL_SERVER_CALLBACK_HANDLER_CLASS
          value: io.strimzi.kafka.oauth.server.JaasServerOauthValidatorCallbackHandler
        - name: KAFKA_LISTENER_NAME_REPLICATION_OAUTHBEARER_SASL_LOGIN_CALLBACK_HANDLER_CLASS
          value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
        - name: KAFKA_PRINCIPAL_BUILDER_CLASS
          value: io.strimzi.kafka.oauth.server.OAuthKafkaPrincipalBuilder
        - name: KAFKA_AUTHORIZER_CLASS_NAME
          value: io.strimzi.kafka.oauth.server.authorizer.KeycloakAuthorizer
        - name: KAFKA_STRIMZI_AUTHORIZATION_TOKEN_ENDPOINT_URI
          value: https://keycloak:8443/realms/demo/protocol/openid-connect/token
        - name: KAFKA_STRIMZI_AUTHORIZATION_CLIENT_ID
          value: "kafka"
        - name: KAFKA_STRIMZI_AUTHORIZATION_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
          value:
        - name: KAFKA_STRIMZI_AUTHORIZATION_SSL_TRUSTSTORE_LOCATION
          value: /etc/ssl/certs/docker/truststore.jks
        - name: KAFKA_STRIMZI_AUTHORIZATION_SSL_TRUSTSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_TRUSTSTORE_PASSWORD
        - name: KAFKA_STRIMZI_AUTHORIZATION_GRANTS_REFRESH_PERIOD_SECONDS
          value: "120"
        - name: KAFKA_STRIMZI_AUTHORIZATION_GRANTS_REFRESH_POOL_SIZE
          value: "10"
        - name: KAFKA_STRIMZI_AUTHORIZATION_HTTP_RETRIES
          value: "3"
        - name: KAFKA_STRIMZI_AUTHORIZATION_CONNECT_TIMEOUT_SECONDS
          value: "10"
        - name: KAFKA_STRIMZI_AUTHORIZATION_GRANTS_REFRESH_PERIOD_SECONDS
          value: "5"
        - name: KAFKA_STRIMZI_AUTHORIZATION_REUSE_GRANTS
          value: "true"
        - name: KAFKA_STRIMZI_AUTHORIZATION_GRANTS_MAX_IDLE_TIME_SECONDS
          value: "600"
        - name: KAFKA_STRIMZI_AUTHORIZATION_DELEGATE_TO_KAFKA_ACL
          value: "false"
        - name: KAFKA_STRIMZI_AUTHORIZATION_CONNECTIONS_MAX_REAUTH_MS
          value: "3600000"
        - name: KAFKA_SUPER_USERS
          value: "User:service-account-kafka"
        - name: KAFKA_OAUTH_USERNAME_CLAIM
          value: "preferred_username"
        - name: KAFKA_SECURITY_PROTOCOL
          value: SASL_SSL
        - name: KAFKA_SSL_KEYSTORE_LOCATION
          value: "/etc/ssl/certs/docker/broker1.jks"
        - name: KAFKA_SSL_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_KEYSTORE_PASSWORD
        - name: KAFKA_SSL_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_KEY_PASSWORD
        - name: KAFKA_SSL_TRUSTSTORE_LOCATION
          value: "/etc/ssl/certs/docker/truststore.jks"
        - name: KAFKA_SSL_TRUSTSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_TRUSTSTORE_PASSWORD
        - name: KAFKA_SSL_KEYSTORE_TYPE
          value: "JKS"
        - name: KAFKA_SSL_TRUSTSTORE_TYPE
          value: "JKS"
        - name: KAFKA_SSL_CLIENT_AUTH
          value: required
        - name: CONFLUENT_METRICS_REPORTER_SSL_KEYSTORE_LOCATION
          value: "/etc/ssl/certs/docker/broker1.jks"
        - name: CONFLUENT_METRICS_REPORTER_SSL_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_KEYSTORE_PASSWORD
        - name: CONFLUENT_METRICS_REPORTER_SSL_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_KEY_PASSWORD
        - name: CONFLUENT_METRICS_REPORTER_SSL_TRUSTSTORE_LOCATION
          value: "/etc/ssl/certs/docker/truststore.jks"
        - name: CONFLUENT_METRICS_REPORTER_SSL_TRUSTSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ssl-kafka-jaas-secret
              key: SSL_TRUSTSTORE_PASSWORD
        - name: CONFLUENT_METRICS_REPORTER_SASL_MECHANISM
          value: OAUTHBEARER
        - name: CONFLUENT_METRICS_REPORTER_SECURITY_PROTOCOL
          value: SASL_SSL
        - name:  CONFLUENT_METRICS_REPORTER_SASL_LOGIN_CALLBACK_HANDLER_CLASS
          value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
        - name: CONFLUENT_METRICS_REPORTER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
          value:
        - name: KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
          value:
        - name: CONFLUENT_METRICS_REPORTER_SASL_JAAS_CONFIG
          value: >
                  org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                  oauth.client.id="$(CLIENT_ID)"
                  oauth.client.secret="$(CLIENT_SECRET)"
                  oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                  oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                  oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)"
                  oauth.valid.issuer.uri="https://keycloak:8443/realms/demo"
                  oauth.jwks.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/certs"
                  oauth.username.claim="preferred_username";
        - name: CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS
          value: {{ printf "SASL_SSL://%s:9092" (include "cp-kafka.cp-kafka-headless.fullname" .) | quote }}
          {{- range $key, $value := .Values.configurationOverrides }}
        command:
        - sh
        - -exc
        - |
          cp /app/* /usr/share/java/kafka/
          cp /app/* /usr/share/java/cp-base-new/
          export KAFKA_BROKER_ID=${HOSTNAME##*-} && \
          export KAFKA_ADVERTISED_LISTENERS=CLIENT://$POD_NAME.{{ template "cp-kafka.fullname" . }}-headless.$POD_NAMESPACE:9092{{ include "cp-kafka.configuration.advertised.listeners" . }} && \
          exec /etc/confluent/docker/run
        volumeMounts:
          - name: jar-for-kafka
            mountPath: "/app/"
          - name: volume-ssl
            mountPath: "/etc/ssl/certs/docker/"
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"


modify the following block of the file cp-kafka/templates/_helpers.tpl as :

Form the Advertised Listeners. We will use the value of nodeport.firstListenerPort to create the
external advertised listeners if configurationOverrides.advertised.listeners is not set.
*/}}
{{- define "cp-kafka.configuration.advertised.listeners" }}
{{- if (index .Values "configurationOverrides" "advertised.listeners") -}}
{{- printf ",%s" (first (pluck "advertised.listeners" .Values.configurationOverrides)) }}
{{- else -}}
{{- printf ",REPLICATION://%s:$((%d + ${KAFKA_BROKER_ID}))" (include "cp-kafka.cp-kafka-headless.fullname" .) (.Values.nodeport.firstListenerPort | toString) }}
{{- end -}}
{{- end -}}

cp-kafka-rest
-------------
modify the following block of the file cp-kafka-rest/templates/deployment.yaml as :

          command:
          - /bin/bash
          - -c
          - |
            cp /app/* /usr/share/java/cp-base-new/
            /etc/confluent/docker/run
          volumeMounts:
          - name: volume-ssl
            mountPath: "/etc/ssl/certs/docker/"
          - name: jar-for-kafka
            mountPath: "/app/"
          env:
          - name: SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
          - name: CLIENT_ID
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_ID
          - name: CLIENT_SECRET
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_SECRET
          - name: KAFKA_REST_CLIENT_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
            value:
          - name: KAFKA_REST_CLIENT_SSL_KEYSTORE_LOCATION
            value: "/etc/ssl/certs/docker/broker1.jks"
          - name: KAFKA_REST_CLIENT_SSL_KEYSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_KEYSTORE_PASSWORD
          - name: KAFKA_REST_CLIENT_SSL_KEY_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_KEY_PASSWORD
          - name: KAFKA_REST_CLIENT_SSL_TRUSTSTORE_LOCATION
            value: "/etc/ssl/certs/docker/truststore.jks"
          - name: KAFKA_REST_CLIENT_SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
          - name: KAFKA_REST_CLIENT_SECURITY_PROTOCOL
            value: SASL_SSL
          - name: KAFKA_REST_CLIENT_SASL_MECHANISM
            value: OAUTHBEARER
          - name: KAFKA_REST_CLIENT_SASL_LOGIN_CALLBACK_HANDLER_CLASS
            value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
          - name: KAFKA_REST_CLIENT_SASL_JAAS_CONFIG
            value: >
                    org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                    oauth.client.id="$(CLIENT_ID)"
                    oauth.client.secret="$(CLIENT_SECRET)"
                    oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                    oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                    oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)";
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"


cp-kafka-connect
----------------
modify the following block of the file cp-kafka-connect/templates/deployment.yaml as :

            env:
            - name: SSL_TRUSTSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_TRUSTSTORE_PASSWORD
            - name: CLIENT_ID
              valueFrom:
                secretKeyRef:
                  name: kafka-client-jaas-secret
                  key: CLIENT_ID
            - name: CLIENT_SECRET
              valueFrom:
                secretKeyRef:
                  name: kafka-client-jaas-secret
                  key: CLIENT_SECRET
            - name: CONNECT_SASL_JAAS_CONFIG
              value: >
                      org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                      oauth.client.id="$(CLIENT_ID)"
                      oauth.client.secret="$(CLIENT_SECRET)"
                      oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                      oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                      oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)";
            - name: CONNECT_SECURITY_PROTOCOL
              value: SASL_SSL
            - name: CONNECT_SASL_MECHANISM
              value: OAUTHBEARER
            - name: CONNECT_SASL_LOGIN_CALLBACK_HANDLER_CLASS
              value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
            - name: CONNECT_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
              value:
            - name: CONNECT_SSL_KEYSTORE_LOCATION
              value: "/etc/ssl/certs/docker/broker1.jks"
            - name: CONNECT_SSL_KEYSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_KEYSTORE_PASSWORD
            - name: CONNECT_SSL_KEY_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_KEY_PASSWORD
            - name: CONNECT_SSL_TRUSTSTORE_LOCATION
              value: "/etc/ssl/certs/docker/truststore.jks"
            - name: CONNECT_SSL_TRUSTSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_TRUSTSTORE_PASSWORD
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"
   

cp-ksql-server
--------------
modify the following block of the file cp-ksql-server/templates/deployment.yaml as :

          volumeMounts:
          - name: volume-ssl
            mountPath: "/etc/ssl/certs/docker/"
          - name: jar-for-kafka
            mountPath: "/app/"
          {{- if .Values.ksql.headless }}
          - name: ksql-queries
            mountPath: /etc/ksql/queries
          {{- end }}
          command:
            - /bin/bash
            - -c
            - |
              cp /app/* /usr/share/java/ksqldb-server/
              cp /app/* /usr/share/java/cp-base-new/
              /etc/confluent/docker/run
          env:
          - name: SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
          - name: CLIENT_ID
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_ID
          - name: CLIENT_SECRET
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_SECRET
          - name: KSQL_SASL_JAAS_CONFIG
            value: >
                   org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                   oauth.client.id="$(CLIENT_ID)"
                   oauth.client.secret="$(CLIENT_SECRET)"
                   oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                   oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                   oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)";
          - name: KSQL_SECURITY_PROTOCOL
            value: SASL_SSL
          - name: KSQL_SASL_MECHANISM
            value: OAUTHBEARER
          - name: KSQL_SASL_LOGIN_CALLBACK_HANDLER_CLASS
            value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
          - name: KSQL_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
            value:
          - name: KSQL_SSL_KEYSTORE_LOCATION
            value: "/etc/ssl/certs/docker/broker1.jks"
          - name: KSQL_SSL_KEYSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_KEYSTORE_PASSWORD
          - name: KSQL_SSL_KEY_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_KEY_PASSWORD
          - name: KSQL_SSL_TRUSTSTORE_LOCATION
            value: "/etc/ssl/certs/docker/truststore.jks"
          - name: KSQL_SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"


modify the following block of the file cp-ksql-server/templates/_helpers.tpl as:


Form the Kafka URL. If Kafka is installed as part of this chart, use k8s service discovery,
else use user-provided URL
*/}}
{{- define "cp-ksql-server.kafka.bootstrapServers" -}}
{{- if .Values.kafka.bootstrapServers -}}
{{- .Values.kafka.bootstrapServers -}}
{{- else -}}
{{- printf "SASL_SSL://%s:9092" (include "cp-ksql-server.cp-kafka-headless.fullname" .) -}}
{{- end -}}
{{- end -}}


cp-schema-registry
------------------
modify the following block of the file cp-schema-registry/templates/deployment.yaml as:

          command:
          - /bin/bash
          - -c
          - |
            cp /app/* /usr/share/java/cp-base-new/
            cp /app/* /usr/share/java/schema-registry/
            /etc/confluent/docker/run
          volumeMounts:
          - name: volume-ssl
            mountPath: "/etc/ssl/certs/docker/"
          - name: jar-for-kafka
            mountPath: "/app/"
          env:
          - name: SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
          - name: CLIENT_ID
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_ID
          - name: CLIENT_SECRET
            valueFrom:
              secretKeyRef:
                name: kafka-client-jaas-secret
                key: CLIENT_SECRET
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
            value:
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_KEYSTORE_LOCATION
            value: "/etc/ssl/certs/docker/broker1.jks"
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_KEYSTORE_PASSWORD
            valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_KEYSTORE_PASSWORD
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_KEY_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_KEY_PASSWORD
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_TRUSTSTORE_LOCATION
            value: "/etc/ssl/certs/docker/truststore.jks"
          - name: SCHEMA_REGISTRY_KAFKASTORE_SSL_TRUSTSTORE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: ssl-kafka-jaas-secret
                key: SSL_TRUSTSTORE_PASSWORD
          - name: SCHEMA_REGISTRY_KAFKASTORE_SECURITY_PROTOCOL
            value: SASL_SSL
          - name: SCHEMA_REGISTRY_KAFKASTORE_SASL_MECHANISM
            value: OAUTHBEARER
          - name: SCHEMA_REGISTRY_KAFKASTORE_SASL_LOGIN_CALLBACK_HANDLER_CLASS
            value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
          - name: SCHEMA_REGISTRY_KAFKASTORE_SASL_JAAS_CONFIG
            value: >
                    org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                    oauth.client.id="$(CLIENT_ID)"
                    oauth.client.secret="$(CLIENT_SECRET)"
                    oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                    oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                    oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)";
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"


modify the following block of the file cp-schema-registry/templates/_helpers.tpl as:

Form the Kafka URL. If Kafka is installed as part of this chart, use k8s service discovery,
else use user-provided URL
*/}}
{{- define "cp-schema-registry.kafka.bootstrapServers" -}}
{{- if .Values.kafka.bootstrapServers -}}
{{- .Values.kafka.bootstrapServers -}}
{{- else -}}
{{- printf "SASL_SSL://%s:9092" (include "cp-kafka-rest.cp-kafka-headless.fullname" .) -}}
{{- end -}}
{{- end -}}


cp-control-center
-----------------
modify the following block of the file cp-control-center/templates/deployment.yaml as:

          command:
          - /bin/bash
          - -c
          - |
            cp /app/* /usr/share/java/cp-base-new/
            cp /app/* /usr/share/java/confluent-control-center/
            /etc/confluent/docker/run
          volumeMounts:
          - name: volume-ssl
            mountPath: "/etc/ssl/certs/docker/"
          - name: jar-for-kafka
            mountPath: "/app/"
          env:
            - name: SSL_TRUSTSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_TRUSTSTORE_PASSWORD
            - name: CLIENT_ID
              valueFrom:
                secretKeyRef:
                  name: kafka-client-jaas-secret
                  key: CLIENT_ID
            - name: CLIENT_SECRET
              valueFrom:
                secretKeyRef:
                  name: kafka-client-jaas-secret
                  key: CLIENT_SECRET
            - name: CONTROL_CENTER_STREAMS_SSL_KEYSTORE_LOCATION
              value: "/etc/ssl/certs/docker/broker1.jks"
            - name: CONTROL_CENTER_STREAMS_SSL_KEYSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_KEYSTORE_PASSWORD
            - name: CONTROL_CENTER_STREAMS_SSL_KEY_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_KEY_PASSWORD
            - name: CONTROL_CENTER_STREAMS_SSL_TRUSTSTORE_LOCATION
              value: "/etc/ssl/certs/docker/truststore.jks"
            - name: CONTROL_CENTER_STREAMS_SSL_TRUSTSTORE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ssl-kafka-jaas-secret
                  key: SSL_TRUSTSTORE_PASSWORD
            - name: CONTROL_CENTER_STREAMS_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
              value:
            - name: CONTROL_CENTER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
              value:
            - name: CONTROL_CENTER_STREAMS_SASL_MECHANISM
              value: OAUTHBEARER
            - name: CONTROL_CENTER_STREAMS_SECURITY_PROTOCOL
              value: SASL_SSL
            - name: CONTROL_CENTER_STREAMS_SASL_LOGIN_CALLBACK_HANDLER_CLASS
              value: io.strimzi.kafka.oauth.client.JaasClientOauthLoginCallbackHandler
            - name: CONTROL_CENTER_STREAMS_SASL_JAAS_CONFIG
              value: >
                      org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required
                      oauth.client.id="$(CLIENT_ID)"
                      oauth.client.secret="$(CLIENT_SECRET)"
                      oauth.token.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/token"
                      oauth.valid.issuer.uri="https://keycloak:8443/realms/demo"
                      oauth.jwks.endpoint.uri="https://keycloak:8443/realms/demo/protocol/openid-connect/certs"
                      oauth.ssl.truststore.location="/etc/ssl/certs/docker/truststore.jks"
                      oauth.ssl.truststore.password="$(SSL_TRUSTSTORE_PASSWORD)"
                      oauth.username.claim="preferred_username";
      volumes:
      - name: volume-ssl
        persistentVolumeClaim:
          claimName: pvc-ssl
      - name: jar-for-kafka
        hostPath:
          path: "/home/meta/jar-for-kafka/"
