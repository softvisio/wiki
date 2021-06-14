# NOTE

!!! Do not forget to add `<script src="cordova.js" type="text/javascript" integrity="" crossorigin="anonymous"></script>` to your `index.html`.

# BUILD APPLICATION

## WINDOWS + ANDROID

Do not use latest platform, some plugins may not work.

To reinstall everything from the scratch remove `node_modules`, `platforms`, `plugins` directories.

```shell
# setup cordova build environment
env-android

# add android platform
cordova platform add --nosave android

# build
cordova build android
```

## MACOS + IOS

!!! Under macos do not work with the project in vmware shared folder, copy it to internal filesystem.

```shell
# add ios platform
cordova platform add --nosave ios

# build
cordova build ios
```

If you will see errors, related to the pods, try this:

```shell
cd platform/ios
pod install
```

# RUN APPLICATION

## WINDOWS

```shell
# run on android device, connected to the USB port
cordova run android --device
```

Access app console from the browser:

```text
chrome://inspect/#devices
```

## MACOS

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

# UPLOAD APP TO ITUNES

You can install and use tool:

```text
https://apps.apple.com/us/app/transporter/id1450874784?mt=12
```

or upload, using command line:

```shell
# upload via command line
xcrun altool --upload-app -f -u dzagashev@gmail.com <filename >.ipa
```

If your account has 2FA enabled, first visit https://appleid.apple.com/ and generate an app password.
