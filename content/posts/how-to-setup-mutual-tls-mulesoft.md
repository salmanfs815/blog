---
title: How to Setup Mutual TLS in MuleSoft
date: 2023-06-07T22:49:00.001Z
tags:
  - mulesoft
  - tutorial
  - encryption
---

Mutual TLS (mTLS), or 2-way SSL, is a server-to-server communication protocol where both client and server authenticate each other.
In this context, client simply means the party requesting a resource and server is the party serving the client's request.

2-way SSL differs from 1-way SSL in which only the client authenticates the server. In 1-way SSL, the client is only required to maintain the server's public key in its truststore and the server only needs to maintain its own private key in its keystore.

However, mTLS requires both client and server to maintain a keystore and a truststore.
The client's keystore will contain its own private key and its truststore will contain the server's public key (or certificate).
Likewise, the server's keystore will contain its own private key and its truststore will contain the client's public key (or certificate).

To generate client and server private keys and public certs:

```bash
openssl req -newkey rsa:2048 -x509 -keyout client-key.pem -out client-cert.pem -days 365
openssl req -newkey rsa:2048 -x509 -keyout server-key.pem -out server-cert.pem -days 365
```

This will need to be exported to PKCS12 format to use in Mule:

```bash
openssl pkcs12 -export -in client-cert.pem -inkey client-key.pem -out client-certificate.p12 -name "mule-client-certificate"
openssl pkcs12 -export -in server-cert.pem -inkey server-key.pem -out server-certificate.p12 -name "mule-server-certificate"
```

The `*.p12` files can be copied into `src/main/resources` in Anypoint Studio.

The HTTP Listener Config will need a TLS Context using the client cert as the truststore and the server cert as the keystore:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:tls="http://www.mulesoft.org/schema/mule/tls" xmlns:http="http://www.mulesoft.org/schema/mule/http"
	xmlns="http://www.mulesoft.org/schema/mule/core"
	xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
http://www.mulesoft.org/schema/mule/tls http://www.mulesoft.org/schema/mule/tls/current/mule-tls.xsd">
	<http:listener-config name="HTTP_Listener_config" doc:name="HTTP Listener config" doc:id="c84bf7ba-052c-4857-aa57-ab6144a5c2cc" >
		<http:listener-connection host="localhost" port="8082" protocol="HTTPS">
			<tls:context >
				<tls:trust-store path="client-certificate.p12" password="mulesoft" type="pkcs12" />
				<tls:key-store type="pkcs12" path="server-certificate.p12" keyPassword="mulesoft" password="mulesoft"/>
			</tls:context>
		</http:listener-connection>
	</http:listener-config>
	<flow name="mtls-demoFlow" doc:id="6ee5b359-beb3-4ab2-b8bb-244d6938d1db" >
		<http:listener doc:name="Listener" doc:id="ab826fca-46ae-44f9-a40b-804cff069b42" config-ref="HTTP_Listener_config" path="/test"/>
		<set-payload value="#['Tested successfully']" doc:name="Set Payload" doc:id="1d2de681-547d-4ab8-b36c-877d3a8e917a" />
	</flow>
</mule>
```

To test via Postman, use `server-cert.pem` as the CA Certificates PEM file.
Client certificate will also need to be added. Specify the domain and port for the host. Use `client-cert.pem` as the CRT file, `client-key.pem` as the KEY file, and include the key password in the `passphrase` field.

Sending a GET request via Postman to `https://localhost:8082/test` should now return a 200 OK response.

mTLS requires dedicated load balancer (DLB). Will not work with shared load balancer (SLB) in CloudHub.

For SOAP Web Services, use a HTTP transport in the WSC Config to setup keystore and truststore.
