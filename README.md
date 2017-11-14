# Instructions to start Spring Boot with jolokia agent

When a Spring Boot application is deployed on Openshift using the openjdk java 1.8 S2I docker image, the jolokia agent is deployed within the image and, when the docker container
is starter, then it is instrumented by the java -javaagent parameter.
The HTTPS service exposed by Jolokia allows to access using the HTTP protocol the information about the MBeans created within the JVM. 

The Jolokia service can be customized using the following parameters defined within the DeploymentConfig resource as env variables :

- AB_JOLOKIA_USER = "jolokia"
- AB_JOLOKIA_PASSWORD = "admin123"
- AB_JOLOKIA_PASSWORD_RANDOM = "false"

Remark : The list of the paramters available is defined [here](https://github.com/jboss-container-images/redhat-openjdk-18-openshift-image/blob/openjdk18-dev/image.yaml)

## Run the project locally
```bash
mvn clean package
java -javaagent:jar/jolokia-jvm-1.3.7-agent.jar=config=etc/jolokia.properties -jar target/spring-boot-jolokia-1.0.0-SNAPSHOT.jar
```

# Query the Jolokia Service

- Get the Jolokia Info

```bash
curl -v -u jolokia:admin123 http://localhost:8778/jolokia/

or

http --auth jolokia:admin123 http://localhost:8778/jolokia/version
...

{
    "request": {
        "type": "version"
    },
    "status": 200,
    "timestamp": 1510659657,
    "value": {
        "agent": "1.3.7",
        "config": {
            "agentContext": "/jolokia",
            "agentId": "192.168.1.80-52647-2503dbd3-jvm",
            "agentType": "jvm",
            "debug": "false",
            "debugMaxEntries": "100",
            "discoveryEnabled": "true",
            "historyMaxEntries": "10",
            "maxCollectionSize": "0",
            "maxDepth": "15",
            "maxObjects": "0"
        },
        "info": {
            "product": "tomcat",
            "vendor": "Apache",
            "version": "8.5.23"
        },
        "protocol": "7.2"
    }
}
```
- Get the Memory usage
```bash
http --auth jolokia:admin123 http://localhost:8778/jolokia/read/java.lang:type\=Memory/HeapMemoryUsage
HTTP/1.1 200 OK
...

{
    "request": {
        "attribute": "HeapMemoryUsage",
        "mbean": "java.lang:type=Memory",
        "type": "read"
    },
    "status": 200,
    "timestamp": 1510660614,
    "value": {
        "committed": 378535936,
        "init": 268435456,
        "max": 3817865216,
        "used": 132615736
    }
}
```

## Deploy on OpenShift

```bash
oc new-project demo-jolokia
mvn clean package fabric8:deploy -Popenshift
```

## Access Jolokia Https Service

- Get the jolokia route endpoint 
```bash
JOLOKIA_URL=oc get route spring-boot-monitoring-jolokia -o jsonpath='https://{.spec.host}'
```

- Next issue a curl or httpie request
```bash
curl -k -u jolokia:admin123 <ROUTE_ADDRESS>/jolokia/version

or

http --verify=no --auth jolokia:admin123 <ROUTE_ADDRESS>/jolokia/version
```

## Install manually the Jolokia service/route

- Create Service/Route on Openshift

```bash
oc delete route/spring-boot-http-jolokia
oc delete svc/spring-boot-http-jolokia
oc create -f src/fabric8/jolokia-svc.yaml          
oc create -f src/fabric8/jolokia-route.yaml 
```

