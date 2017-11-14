# Instructions to start Spring Boot with jolokia agent

## Locally
```bash
mvn clean package
java -javaagent:jar/jolokia-jvm-1.3.7-agent.jar=config=etc/jolokia.properties -jar target/spring-boot-jolokia-1.0.0-SNAPSHOT.jar
```

# Query the agent

```bash
curl -v -u jolokia:MYPASSWORD http://localhost:8778/jolokia/

or

http --auth jolokia:MYPASSWORD http://localhost:8778/jolokia/version
HTTP/1.1 200 OK
Cache-control: no-cache
Content-type: text/plain; charset=utf-8
Date: Tue, 14 Nov 2017 11:40:57 GMT
Expires: Tue, 14 Nov 2017 10:40:57 GMT
Pragma: no-cache
Transfer-encoding: chunked

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

http --auth jolokia:MYPASSWORD http://localhost:8778/jolokia/read/java.lang:type\=Memory/HeapMemoryUsage
HTTP/1.1 200 OK
Cache-control: no-cache
Content-type: text/plain; charset=utf-8
Date: Tue, 14 Nov 2017 11:56:54 GMT
Expires: Tue, 14 Nov 2017 10:56:54 GMT
Pragma: no-cache
Transfer-encoding: chunked

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

## Install manually the Jolokia service/route

- Create Service/Route on Openshift

```bash
oc delete route/spring-boot-http-jolokia
oc delete svc/spring-boot-http-jolokia
oc create -f src/fabric8/jolokia-svc.yaml          
oc create -f src/fabric8/jolokia-route.yaml 
```

- Access to jolokia

```bash
export PASSWORD=r1pWReSZzGJka4Bs4VcsXs1ukmJyKz
curl -k -u jolokia:$PASSWORD https://spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com/jolokia/version

or

http --verify=no --auth jolokia:$PASSWORD https://spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com/jolokia/version
```

