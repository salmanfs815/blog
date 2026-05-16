---
title: "Quick Reference: Setting up HTTPS in Mule 4"
date: 2021-06-01T17:19:00.001Z
tags:
  - mulesoft
  - tutorial
  - encryption
---

Inputs required:
* keystore file name
* alias
* keypass/password
* encryption key

Need to generate keystore:
```bash
keytool -genkeypair -dname "CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown" -keyalg RSA -sigalg SHA256withRSA -keysize 2048 -validity 9999 -keystore <keystore_file_name (*.jks)> -alias <alias> -keypass <keypass/password> -storepass <keypass/password>
```

keypass/password is sensitive information
needs to be stored in encrypted format in config/properties file

```bash
java -cp secure-properties-tool.jar com.mulesoft.tools.SecurePropertiesTool string encrypt AES CBC <encryption_key (16 chars)> <keypass/password>
```

output string should be placed in config file inside: `"![...]"`
