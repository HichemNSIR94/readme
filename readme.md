Building
--------

    mvn clean install

Installing
----------

Copy the following jars into your Kafka `libs` directory (`cp-base-new` or `kafka`) under /usr/share/java/:

    oauth-common/target/kafka-oauth-common-*.jar
    oauth-server/target/kafka-oauth-server-*.jar
    oauth-server-plain/target/kafka-oauth-server-plain-*.jar
    oauth-keycloak-authorizer/target/kafka-oauth-keycloak-authorizer-*.jar
    oauth-client/target/kafka-oauth-client-*.jar
    oauth-common/target/lib/nimbus-jose-jwt-*.jar
    oauth-server/target/lib/json-path-*.jar
    oauth-server/target/lib/json-smart-*.jar
    oauth-server/target/lib/accessors-smart-*.jar
