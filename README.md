
# CamelK Hello World example

Camel K is an integration framework designed specifically for Kubernetes, enabling rapid development, deployment, and management of integration flows directly on Kubernetes clusters. It leverages Apache Camel's extensive library of integration patterns to simplify the creation of complex workflows, making it ideal for cloud-native applications and microservices architectures.




## Prerequisites

 - Red Hat Integration - AMQ Broker for RHEL 8 Operator Installed.
 - Red Hat Integration - Camel K, latest mirror image.
 - [Camel K Client](https://github.com/apache/camel-k/releases) installed.

## AMQ 7 Broker Setup

Install your AMQ 7 Broker instance via operator hub with one acceptor using the following parameters:

- name=acceptor1
- exposed=true
- port=61626
## Running the POC

 -  Create a camel route:
```java
$ vi HelloK.java

import org.apache.camel.builder.RouteBuilder;

public class HelloK extends RouteBuilder {
@Override
    public void configure() throws Exception {
        from ("timer:java?period=1000")
        .setBody ()
        .simple ("Hello, World")
        .log ("${body}")
        .to("jms:foo");
    }
}
```
 -  Create a properties file:
 ```bash
$ vi application.properties

#Artemis Client Runtime
camel.beans.artemisCF = #class:org.apache.activemq.artemis.jms.client.ActiveMQConnectionFactory
camel.beans.artemisCF.brokerURL = tcp://ex-aao-acceptor1-0-svc:61626
# Pool Connection Factory
camel.beans.poolCF = #class:org.messaginghub.pooled.jms.JmsPoolConnectionFactory
camel.beans.poolCF.connectionFactory = #bean:artemisCF
camel.beans.poolCF.maxSessionsPerConnection = 5
camel.beans.poolCF.connectionIdleTimeout = 20000
camel.component.jms.connection-factory = #bean:poolCF
```
 - Create a configmap with the previously created application.properties file:
```bash
$ oc create configmap hello-config --from-file application.properties
```
 - Execute the project:
```bash
 $ kamel run HelloK.java --config configmap:hello-config -d mvn:org.apache.activemq:artemis-jms-client-all:2.28.0 -d mvn:org.messaginghub:pooled-jms:1.2.3.redhat-00001
You're using Camel K 2.3.4-nightly client with a 1.10.6 cluster operator, it's recommended to use the same version to improve compatibility.

Integration "hello-k" created
 $ oc get pods
 $ oc get logs hello-k-XXX
 ...
2024-06-11 14:08:00,653 INFO  [route1] (Camel (camel-1) thread #1 - timer://java) Hello, World
2024-06-11 14:08:01,653 INFO  [route1] (Camel (camel-1) thread #1 - timer://java) Hello, World
2024-06-11 14:08:02,654 INFO  [route1] (Camel (camel-1) thread #1 - timer://java) Hello, World
2024-06-11 14:08:03,653 INFO  [route1] (Camel (camel-1) thread #1 - timer://java) Hello, World
...
```