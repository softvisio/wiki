# Firebase

### Message

```javascript
const message = {
    "token": "",
    "topic": "",

    "notification": {
        "title": "",
        "body": "",
        "imageUrl": "https://example.com/favicon.ico",
    },

    "data": {

        // cordova-plugin-firebasex, disply notification also if app in foreground
        "notification_foreground": "true",
    },

    // https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.androidconfig.md#androidconfig_interface
    "android": {

        // https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.androidnotification.md#androidnotification_interface
        "notification": {},
    },

    // https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.apnsconfig.md#apnsconfig_interface
    "apns": {},

    // https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.webpushconfig.md#webpushconfig_interface
    "webpush": {

        // https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.webpushnotification.md#webpushnotification_interface
        "notification": {
            "icon": "/favicon.ico",

            "actions": [
                {
                    "action": "action1",
                    "icon": "/favicon.ico",
                    "title": "Action 1",
                },
                {
                    "action": "action2",
                    "icon": "/favicon.ico",
                    "title": "Action 2",
                },
            ],

            // replace notification with the same tag, tag is required
            "renotify": true,

            "requireInteraction": false,
            "silent": false,
            "tag": "",

            // required if silent is true, https://developer.mozilla.org/en-US/docs/Web/API/Vibration_API#vibration_patterns
            "vibration": [ 200, 100, 200 ],
        },

        "fcmOptions": {

            // must be relative url, not used, if actions specified
            "link": "/",
        },
    },
};
```

### Android

[Notifications icons builder](http://romannurik.github.io/AndroidAssetStudio/index.html)

### iOS

### Generate APNS key

1. In apple developers console goto `Keys` and generate new `Apple Push Notifications service (APNs)`.
   Store `KeyID` and download `.p8` file;

2. Get your `TeamID` from <https://developer.apple.com/account/#/membership>;

### Setup firebase

1. Goto firebase console.

2. Create new firebase project, linked to the gcloud project.

3. Open project settings -> `Service account`. Click `Generate new private key`. Use downloaded file in backend config.

4. Create new android app:
    - download `google-services.json`, this file will be used in cordova push notifications plugin;

5. Create new iOS app:
    - set `TeamID`;
    - download `GoogleService-Info.plist` for use in cordova;

6. Setup APNS connection:
    - goto `project settings / cloud messaging / ios app configuration / your app`;
    - add APNs Authentication Key, use `.p8` file, `KeyID`, `TeamID`;
