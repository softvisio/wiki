# iOS

### Prepare environment

```shell
curl -fsSL https://raw.githubusercontent.com/softvisio/scripts/main/setup-host-macos.sh | /bin/bash

brew update
brew upgrade node

npm up -g

pod repo update
```

### Prepare project

-   Copy project from the VMWare host directory to the local filesystem.

-   Install cordova platform:

    ```shell
    npm up
    cordova platform add --nosave ios
    ```

-   Update pods (if you will see errors, related to the pods):

    ```shell
    cd platform/ios
    pod install
    ```

-   Open project workspace in `xcode`:
    ```shell
    open -a xcode platforms/ios/*.xcworkspace
    ```
    In xcode set apple team and select automanage signing.

### Build

```shell
cordova build ios
```

### Release

```shell
cordova build ios --release --device
```

### Run in simulator

```shell
# view available devices under emulator
cordova run --list

# run in default emulator device
cordova run ios --emulator --nobuild

# run emulator for 4.7" device, not mandatory for app store screnshots
cordova run ios --target="iPhone-8, 13.3" --nobuild

# run emulator for 5.5" device
cordova run ios --target="iPhone-8-Plus, 13.3" --nobuild

# run emulator for 6.5" device
cordova run ios --target="iPhone-11-Pro-Max, 13.3" --nobuild

# run emulator for 12.9" device
cordova run ios --target="iPad-Pro--12-9-inch---4th-generation-, 13.5" --nobuild
```

### Deploy

You can install and use `Transporter` tool (preferred):

```text
https://apps.apple.com/us/app/transporter/id1450874784?mt=12
```

or upload, using command line:

```shell
# upload via command line
xcrun altool --upload-app -f -u dzagashev@gmail.com < filename > .ipa
```

If your account has 2FA enabled, first visit https://appleid.apple.com/ and generate an app password.

---

### Sign

For ios, macos creates certificates and povisioning profiles in xcode.

#### Identifier

-   Register new `App ID` identifier;

-   Do not forget to add `Push Notifications` capability. Do not create any push notifications certificates because we will use keys;

#### Profile

Provisioning profile links certificate, identified and devices together.

-   Create new provisioning profile, select `Distribution / App Store`;

### Certificates

#### Create certificates

-   xcode -> Preferences -> Accounts
    -   Add apple account;
    -   Manage certificates -> add certificate: development, distributuion;

#### Remove old certificates

-   Revoke certificate on [https://developer.apple.com/account/resources/certificates/list](https://developer.apple.com/account/resources/certificates/list);
-   Delete certificate in `Keychain Access Tool`: Finder -> Applications -> Utils -> Keychain Access -> My Certificates;