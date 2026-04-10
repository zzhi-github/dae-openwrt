# dae-openwrt

## Installation

1. Download the APK files from [here](https://github.com/douglarek/dae-openwrt/releases).
2. Install the packages using the following command:

> [!IMPORTANT]
> Starting from November 2024, OpenWrt will use the apk package manager by default. Sorry, this project will only support building APK packages and will no longer support IPK.


   ```sh
   apk add dae*.apk --allow-untrusted
   ```

**Note:** For optimal stability, it is recommended to use the stable version. Use the unstable version at your own risk.

## For Old Openwrt 24.10.4
A special package was created for old openwrt 24.10.4 that still uses ipk
The ipk files were built for specific version. 
No guarantee for the ipk packages, take it on you own risk.
