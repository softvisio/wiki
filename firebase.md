# Firebase APNS

### Generate APNS key

1. In apple developers console goto `Keys` and generate new `Apple Push Notifications service (APNs)`.
   Store `KeyID` and download `.p8` file;

2. Get your `TeamID` from https://developer.apple.com/account/#/membership;

### Setup firebase

1. Goto firebase console;

2. Create new android app:

    - download `google-services.json`, this file will be used in cordova push notifications plugin;

3. Create new iOS app:

    - set `TeamID`;
    - download `GoogleService-Info.plist` for use in cordova;

4. Setup APNS connection:
    - goto `project settings / cloud messaging / ios app configuration / your app`;
    - add APNs Authentication Key, use `.p8` file, `KeyID`, `TeamID`;
