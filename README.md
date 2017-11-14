# Instructions to start Spring Boot with jolokia agent

mvn clean package
java -javaagent:jar/jolokia-jvm-1.3.7-agent.jar=config=jolokia.properties -jar target/actuator-0.0.1-SNAPSHOT.jar

# Query the agent

```bash
curl -v http://localhost:8778/jolokia/

or

http http://localhost:8778/jolokia/
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

http http://localhost:8778/jolokia/read/java.lang:type\=Memory/HeapMemoryUsage
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

## Create Service/Route on Openshift

```bash
oc delete route/spring-boot-http-jolokia
oc delete svc/spring-boot-http-jolokia
oc create -f src/fabric8/svc.yaml          
oc create -f src/fabric8/route.yaml 
```

## Access to jolokia

```bash
curl -kv -u jolokia:uG5YPFG8ZkL1ekUIEuPE1OOQaBJ6hn https://spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com/jolokia/
*   Trying 10.8.243.248...
* TCP_NODELAY set
* Connected to spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com (10.8.243.248) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /usr/local/etc/openssl/cert.pem
  CApath: /usr/local/etc/openssl/certs
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Request CERT (13):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Certificate (11):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=DE; ST=Franconia; L=Pegnitz; O=jolokia.org; OU=JVM; CN=Jolokia Agent 1.3.6
*  start date: Nov 14 12:16:44 2017 GMT
*  expire date: Nov 12 12:16:44 2027 GMT
*  issuer: C=DE; ST=Franconia; L=Pegnitz; O=jolokia.org; OU=JVM; CN=Jolokia Agent 1.3.6
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Server auth using Basic with user 'jolokia'
> GET /jolokia/ HTTP/1.1
> Host: spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com
> Authorization: Basic am9sb2tpYTp1RzVZUEZHOFprTDFla1VJRXVQRTFPT1FhQko2aG4=
> User-Agent: curl/7.56.1
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
* Authentication problem. Ignoring this.
< Www-authenticate: Basic realm="jolokia"
< Date: Tue, 14 Nov 2017 13:01:01 GMT
< Content-length: 0
< 
* Connection #0 to host spring-boot-http-jolokia-cmoulliard.ose.spring-boot.osepool.centralci.eng.rdu2.redhat.com left intact

```

