# iOS

### Errors

If you have cordova platform install or build errors:

- Cheche, that you have latest `xcode` version installed. If new `xcode` version is exists but not available in updates - you need to upgrade `macos` to the latest release.

- Build project in `xcode` (product -> build) and apply all recommended fixes to config (using panel at the left) until it will built siccessfully.

- Update `cocoapods`:

    ```sh
    brew upgrade cocoapods

    pod repo update
    ```

- Update platform `pods`:

    ```sh
    cd platform/ios

    pod deintegrate
    pod setup
    pod install --verbose
    ```

### Prepare environment

Init `macos` environment:

```sh
script=$(curl -fsSL "https://raw.githubusercontent.com/softvisio/scripts/main/setup-host-macos.sh")
bash <(echo "$script")

# TODO restart terminal
```

Update `macos` environment:

```sh
update
```

### Prepare project

- Copy project from the VMWare host directory to the local filesystem.

- Install cordova platform:

    ```sh
    npm up

    cordova platform add --nosave ios
    ```

- Open project workspace in `xcode`:

    ```sh
    open -a xcode platforms/ios/*.xcworkspace
    ```

    In `xcode` set apple team and select automanage signing for development anf release. Try to build project in `xcode` and apply all recommended patches for config.

### Build

```sh
cordova build ios
```

### Release

```sh
cordova build ios --release --device
```

### Run in simulator

```sh
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

```
https://apps.apple.com/us/app/transporter/id1450874784?mt=12
```

or upload, using command line:

```sh
# upload via command line
xcrun altool --upload-app -f -u dzagashev@gmail.com < filename > .ipa
```

If your account has 2FA enabled, first visit <https://appleid.apple.com/> and generate an app password.

---

### Manage certificates

#### Create certificates

xcode -> preferences -> account tab (at the top) -> select team -> press manage certificates -> create certificate (at the bottom)

#### Remove old certificates

- Revoke certificate on <https://developer.apple.com/account/resources/certificates/list>;
- Delete certificate in `Keychain Access Tool`: Finder -> Applications -> Utils -> Keychain Access -> My Certificates;

### Sign

For ios, macos creates certificates and povisioning profiles in xcode.

#### Identifier

- Register new `App ID` identifier;

- Do not forget to add `Push Notifications` capability. Do not create any push notifications certificates because we will use keys;

#### Profile

Provisioning profile links certificate, identified and devices together.

- Create new provisioning profile, select `Distribution / App Store`;
