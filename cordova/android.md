# Android

### Prepare environment

- Java SE JDK
    - [download latest JDK](https://www.oracle.com/java/technologies/downloads/#jdk18-windows)
    - unpack to devel/android/java

- Gradle:
    - [download binary-only](https://gradle.org/install/#manually)
    - unpack to devel/android/gradle

- Android SDK tools
    - [download sdk-tools-windows.zip](https://developer.android.com/studio/#downloads)
    - unpack to devel/android/cmdline-tools/latest

### Prepare project

- Install cordova platform:

    ```sh
    env-android
    npm up
    cordova platform add --nosave android
    ```

    Do not use latest platform, some plugins may not work.

    To reinstall everything from the scratch remove `node_modules`, `platforms`, `plugins` directories.

### Build

```sh
cordova build android
```

### Release

```sh
cordova build android --release
```

### Run

Run on android device, connected to the USB port:

```sh
cordova run android --device
```

Access app console from the browser:

```
chrome://inspect/#devices
```

### Prepare device

- Sony device drivers
    - [download and install drivers for your device](https://developer.sony.com/develop/drivers/)

- To enable developers mode tap 7 times on `Settings -> System -> About phone -> Build number`.

- Enable / disable USB debugging: `Settings -> Advanced -> Developer options`.

### Prepare android emulator

- Intel HEXM

    Download and install [Intel HEXM](https://software.intel.com/en-us/articles/intel-hardware-accelerated-execution-manager-intel-haxm) or using sdkmanager:

    ```sh
    sdkmanager "extras;intel;Hardware_Accelerated_Execution_Manager"
    ```

    and install from `cordova/android/extras`.

- Install emulator

    ```sh
    sdkmanager emulator
    ```

- List abi system images

    ```sh
    sdkmanager --list | grep "system-images;android-"
    ```

- Install abi system image

    ```sh
    sdkmanager "system-images;android-28;google_apis;x86_64"
    ```

- Create avd

    ```sh
    avdmanager create avd --force --abi "google_apis/x86_64" --package "system-images;android-28;google_apis;x86_64" --device "pixel" --name "pixel"
    ```

- Run emulator
    ```sh
    emulator -avd pixel
    ```

### Certificates

```sh
# setup build environment
env-android

# generate keystore
keytool -genkey -v -keystore zdm@softvisio.net.jks -alias zdm@softvisio.net -keyalg RSA -keysize 2048 -validity 10000

# convert keystore to pkcs12 format
keytool -importkeystore -srckeystore zdm@softvisio.net.jks -destkeystore zdm@softvisio.net.pkcs12 -deststoretype pkcs12

# .jks keystore can be removed
rm zdm@softvisio.net.jks
```
