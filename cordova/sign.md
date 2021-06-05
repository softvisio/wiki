# ANDROID

```
# setup build environment
env-android

# generate keystore
keytool -genkey -v -keystore zdm@softvisio.net.jks -alias zdm@softvisio.net -keyalg RSA -keysize 2048 -validity 10000

# convert keystore to pkcs12 format
keytool -importkeystore -srckeystore zdm@softvisio.net.jks -destkeystore zdm@softvisio.net.pkcs12 -deststoretype pkcs12

# .jks keystore can be removed
rm zdm@softvisio.net.jks
```

# IOS

For ios, macos create certificates and povisioning profiles in xcode.

## IDENTIFIER

-   Register new `App ID` identifier;

-   Do not forget to add `Push Notifications` capability. Do not create any push notifications certificates because we will use keys;

## PROFILE

Provisioning profile links certificate, identified and devices together.

-   Create new provisioning profile, select `Distribution / App Store`;
