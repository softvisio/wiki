# PREPARE PROJECT UNDER IOS

```
# firebasex plugin requirements
# https://github.com/dpa99c/cordova-plugin-firebasex/issues/452
cp resources/ios/GoogleService-Info.plist ./

# WARNING!!! Do not use "cordova prepare ios", because it is requires pre-installed dependencies.
cordova platform add ios

# in xcode set apple team and select automanage signing.
open -a xcode platforms/ios
```

# RELEASE SIGNED .IPA

To build signed `.ipa` bundle:

```
cordova build ios --release --device
```

# CERTIFICATES

## CREATE CERTIFICATES

-   xcode -> Preferences -> Accounts
    -   Add apple account;
    -   Manage certificates -> add certificate: development, distributuion;

## REMOVE OLD CERTIFICATES

-   Revoke certificate on [https://developer.apple.com/account/resources/certificates/list](https://developer.apple.com/account/resources/certificates/list);
-   Delete certificate in `Keychain Access Tool`: Finder -> Applications -> Utils -> Keychain Access -> My Certificates;
