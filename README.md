docker-kafka
============

It's a Dockerfile for [Apache Kafka](http://kafka.apache.org/)

Buids a docker image with specified apache kafka version. It DOES NOT include zookeeper.

## Pre-Requisites

- install docker-compose [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
- if you want to customize any Kafka parameters, simply add them as environment variables in `docker-compose.yml`, e.g. in order to increase the `message.max.bytes` parameter set the environment to `KAFKA_MESSAGE_MAX_BYTES: 2000000`. To turn off automatic topic creation set `KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'`
- Kafka's log4j usage can be customized by adding environment variables prefixed with `LOG4J_`. These will be mapped to `log4j.properties`. For example: `LOG4J_LOGGER_KAFKA_AUTHORIZER_LOGGER=DEBUG, authorizerAppender`

## Usage

Start kafka:

- `docker-compose up -d `

Stop kafka:

- `docker-compose stop`

## Note

The default `docker-compose.yml` should be seen as a starting point / example.

## Broker IDs

You can configure the broker id in different ways

1. explicitly, using `KAFKA_BROKER_ID`
2. via a command, using `BROKER_ID_COMMAND`, e.g. `BROKER_ID_COMMAND: "hostname | awk -F'-' '{print $2}'"`

If you don't specify a broker id in your docker-compose file, it will automatically be generated (see [https://issues.apache.org/jira/browse/KAFKA-1070](https://issues.apache.org/jira/browse/KAFKA-1070).


## Automatically create topics

If you want to have kafka-docker automatically create topics in Kafka during
creation, a `KAFKA_CREATE_TOPICS` environment variable can be
added in `docker-compose.yml`.

Here is an example snippet from `docker-compose.yml`:

        environment:
          KAFKA_CREATE_TOPICS: "Topic1:1:3,Topic2:1:1:compact"

`Topic 1` will have 1 partition and 3 replicas, `Topic 2` will have 1 partition, 1 replica and a `cleanup.policy` set to `compact`.

## Advertised hostname

You can configure the advertised hostname in different ways

1. explicitly, using `KAFKA_ADVERTISED_HOST_NAME`
2. via a command, using `HOSTNAME_COMMAND`, e.g. `HOSTNAME_COMMAND: "route -n | awk '/UG[ \t]/{print $$2}'"`

When using commands, make sure you review the "Variable Substitution" section in [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

If `KAFKA_ADVERTISED_HOST_NAME` is specified, it takes precedence over `HOSTNAME_COMMAND`

## JMX

For monitoring purposes you may wish to configure JMX. Additional to the standard JMX parameters, problems could arise from the underlying RMI protocol used to connect

* java.rmi.server.hostname - interface to bind listening port
* com.sun.management.jmxremote.rmi.port - The port to service RMI requests

For example, to connect to a kafka running locally (assumes exposing port 1099)

      KAFKA_JMX_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.rmi.port=1099"
      JMX_PORT: 1099

Jconsole can now connect at `jconsole $jmxHost:$jmxPort`

## Listener Configuration

Newer versions of Kafka have deprecated `advertised.host.name` and `advertised.port` in favor of a more flexible listener configuration that supports multiple listeners using the same or different protocols. This image supports up to three listeners to be configured automatically as shown below.

Note: if the below listener configuration is not used, legacy conventions for "advertised.host.name" and "advertised.port" still operate without change.

1. Use `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP` to configure an INSIDE, OUTSIDE, and optionally a BROKER protocol. These names are arbitrary but used for consistency and clarity.
   * `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:SSL,BROKER:PLAINTEXT` configures three listener names, but only the listener named OUTSIDE uses SSL. Note this example does not concern extra steps in configuring SSL on a broker.
2. Use `KAFKA_ADVERTISED_PROTOCOL_NAME` to set the name from the protocol map to be used for the "advertised.listeners" property. This is "OUTSIDE" in this example.
3. Use `KAFKA_PROTOCOL_NAME` to set the name from the protocol map to be used for the "listeners" property. This is "INSIDE" in this example.
4. Use `KAFKA_INTER_BROKER_LISTENER_NAME` to set the name from the protocol map to be used for the "inter.broker.listener.name". This defaults to `KAFKA_PROTOCOL_NAME` if not supplied. This is "BROKER" in the example.
5. Use `KAFKA_ADVERTISED_PORT` and `KAFKA_ADVERTISED_HOST_NAME` (or the `HOSTNAME_COMMAND` option) to set the name and port to be used in the `advertised.listeners` list.
6. Use `KAFKA_PORT` and `KAFKA_HOST_NAME` (optional) to set the name (optional) and port to be used in the `listeners` list. If `KAFKA_HOST_NAME` is not defined, Kafka's reasonable default behavior will be used and is sufficient. Note that `KAFKA_PORT` defaults to "9092" if not defined.
7. Use `KAFKA_INTER_BROKER_LISTENER_PORT` to set the port number to be used in both `advertised.listeners` and `listeners` for the Inter-broker listener. The host name for this listener is not configurable. Kafka's reasonable default behavior is used.
