
## spring-ssl

![spring-ssl](spring-ssl.png?id=1")  

## spring-ssl-server
Make Spring application work on SSL/HTTPS.



[Oracle keytool reference](https://docs.oracle.com/en/java/javase/13/docs/specs/man/keytool.html)  

### Generate 1 entry / 1 certificate (= public key + private key)

> cd JAVA_HOME/bin/  
> **keytool -genkeypair -alias viki3d -keyalg RSA -keysize 2048 -validity 3650 -keystore viki3d.p12 -storepass 123456 -storetype PKCS12**  

The file: **viki3d.jks** is generated.  

| Command | Description |  
| ------- | ----------- |
| _-genkeypair_ |  Generates pair of private/public keys  
| _-alias_      |  The alias of the **key** entry encapsulated in the keystore.  
| _-keyalg_     |  **Key** algorithm, from [RSA, DSA, EC]
| _-validity_   |  **Key** validity number of days
| _-keystore_   |  The filename of the **keystore**
| _-storepass_  |  **Keystore** password
| _-storetype_  |  **Keystore** type, from [PKCS12, JKS]


### List entries  
We can see only public key here! Private key is not accessible via keytool.  
One entry = one keypair = public key + private key.  
> cd JAVA_HOME/bin/  
> **$> keytool -list -keystore viki3d.p12 -storepass 123456**  
> Keystore type: PKCS12
> Keystore provider: SUN
> 
> Your keystore contains 1 entry  
> 
> viki3d, Oct 15, 2000, PrivateKeyEntry,
Certificate fingerprint (SHA-256): 5B:E3:40:10:1F:E3:29:03:15:60:30:F0:2E:A9:1B:FF:C2:83:8C:0E:BD:B5:CF:12:69:0B:D2:AC:5D:54:76:48  
>  

### List specific entry (detailed)
Private key can not be shown, since it is not accessible via keytool.  
> cd JAVA_HOME/bin/  
> keytool -list -keystore viki3d.p12 -storepass 123456 -v -alias viki3d  

### Option to delete entry from keystore by its alias  
> keytool -delete -alias viki3d -keystore viki3d.p12 -storepass 123456  

### Export the public key (from our entry/alias) into a .crt file  

> cd JAVA_HOME/bin/  
> **keytool -export -keystore viki3d.p12 -alias viki3d -file viki3d.crt**  

The file: **viki3d.crt** is generated.  

The public key is for the client. Client imports the .crt file into Certification Authority.
This prevents client browser to complain about unknown/untrusted certificate.  

**/src/main/resources/application.properties**:  
> \# Server port  
> server.port=8080  
> \# When SSL options are enabled(like here): the server.port will become SSL (HTTPS instead HTTP)  
> server.ssl.enabled=true  
> server.ssl.key-store-type=PKCS12  
> server.ssl.key-store=classpath:viki3d.p12  
> server.ssl.key-store-password=123456  
> server.ssl.key-alias=viki3d  
> \# This will he the HTTP port, where the SpringBoot app will listen  
> \# SpringBoot can have only 1 listening port using configuration only.  
> \# To make this HTTP work as 2nd port - configure it Java code   
> \# (com.viki3d.spring.server.ssl.logic.config.SSlConfig)  
> http.port=9080  



**com.viki3d.spring.server.ssl.logic.config.ServerPortConfig**:  

> @Configuration  
> public class ServerPortConfig {  
> 
> &nbsp;&nbsp; @Value("${http.port}")  
> &nbsp;&nbsp; private int httpPort;  
> 
> &nbsp;&nbsp; @Bean  
> &nbsp;&nbsp; public ServletWebServerFactory servletContainer() {  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tomcat.addAdditionalTomcatConnectors(createStandardConnector());  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return tomcat;  
> &nbsp;&nbsp; }  
> 
> &nbsp;&nbsp; private Connector createStandardConnector() {  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connector.setPort(httpPort);  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return connector;  
> &nbsp;&nbsp; }  
> }  


## spring-ssl-client
Consume REST service from HTTPS.

