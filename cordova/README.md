# Cordova

:warn: Do not forget to add `<script src="cordova.js" type="text/javascript" integrity="" crossorigin="anonymous"></script>` to your `index.html`.

### Build application

#### Windows + android

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

### Run application

#### Windows

```shell
# run on android device, connected to the USB port
cordova run android --device
```

Access app console from the browser:

```text
chrome://inspect/#devices
```
