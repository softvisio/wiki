# iOS

### Errors

If you have cordova platform install or build errors:

-   Cheche, that you have latest `xcode` version installed. If new `xcode` version is exists but not available in updates - you need to upgrade `macos` to the latest release.

-   Build project in `xcode` (product -> build) and apply all recommended fixes to config (using panel at the left) until it will built siccessfully.

-   Update `cocoapods`:

    ```shell
    brew upgrade cocoapods

    pod repo update
    ```

-   Update platform `pods`:

    ```shell
    cd platform/ios

    pod deintegrate
    pod setup
    pod install --verbose
    ```

### Prepare environment

Init `macos` environment:

```shell
# install xcode devel tools
xcode-select --install

# install brew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# set the patb to the active devel dir
# sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

# install brew packages
brew install wget mc node cocoapods ios-sim

# setup cocoapods environment
pod setup

# update cocoapods repositories
pod repo update

# npm install --global ios-sim npm
npm install --global cordova
```

Update `macos` environment:

```shell
# update brew to the lates version
brew update

# update brew packages to the latest varions
brew upgrade

# update cocoapods repositories
pod repo update

# update global node packages
npm up -g
```

### Prepare project

-   Copy project from the VMWare host directory to the local filesystem.

-   Install cordova platform:

    ```shell
    npm up

    cordova platform add --nosave ios
    ```

-   Open project workspace in `xcode`:

    ```shell
    open -a xcode platforms/ios/*.xcworkspace
    ```

    In `xcode` set apple team and select automanage signing for development anf release. Try to build project in `xcode` and apply all recommended patches for config.

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

### Manage certificates

#### Create certificates

xcode -> preferences -> account tab (at the top) -> select team -> press manage certificates -> create certificate (at the bottom)

#### Remove old certificates

-   Revoke certificate on [https://developer.apple.com/account/resources/certificates/list](https://developer.apple.com/account/resources/certificates/list);
-   Delete certificate in `Keychain Access Tool`: Finder -> Applications -> Utils -> Keychain Access -> My Certificates;

### Sign

For ios, macos creates certificates and povisioning profiles in xcode.

#### Identifier

-   Register new `App ID` identifier;

-   Do not forget to add `Push Notifications` capability. Do not create any push notifications certificates because we will use keys;

#### Profile

Provisioning profile links certificate, identified and devices together.

-   Create new provisioning profile, select `Distribution / App Store`;
