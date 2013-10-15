
# Bluetooth

Various notes on using Bluetooth within the OpenSharingToolkit.

## Object Push

The most portable/widely supported option is likely to be Object Push Profile, which is based on OBEX (Object Exchange). E.g. [Object Push Profile 1.1](https://www.bluetooth.org/docman/handlers/DownloadDoc.ashx?doc_id=8706). There are versions over RFCOMM and L2CAP (Bluetooth transports).

## Android Support

### General Android Support

Many Android devices support Bluetooth at the hardware level. Typically the standard settings allow Bluetooth to be enabled/disabled, allow discoverability to be enabled (usually explicitly and time-limited), allow pairing and removal of pairing. Enabling and discovery can also be controlled from an application with suitable permission (BLUETOOTH_ADMIN) but this will typically require explicit user confirmation.

### Android API support

The [Android Bluetooth API](http://developer.android.com/guide/topics/connectivity/bluetooth.html) supports discovery, SDP and RFCOMM. Later extensions (since API 11) add support for Bluetooth Low-Energy devices and profiles, although these are not particularly relevant in this context. L2CAP (alternative transport to RFCOMM) is officially supported (it been usable natively and by reflection on some but not all devices with older versions of Android but may not be now (e.g. [stackoverflow post](http://stackoverflow.com/questions/14761570/connecting-to-a-bluetooth-hid-device-mouse-using-l2cap)).

### Android standard Application support

The standard [Bluetooth share](https://android.googlesource.com/platform/packages/apps/Bluetooth/+/master/src/com/android/bluetooth/opp/) application provides support for sending (via SHARE intent) and receiving a range of file types over bluetooth, apparently including any image, video or audio format, plain text, html, pdf, zip file and MS/open office documents, but excluding APKs (see [Constants](https://android.googlesource.com/platform/packages/apps/Bluetooth/+/master/src/com/android/bluetooth/opp/Constants.java)). Files that are accepted are not necessarily supported on the device (e.g. locally unsupported audio formats). 

Bluetooth share makes heavy use of notifications to interact with the user, e.g. announcing incoming files, reporting incoming and outgoing transfer progress and outcome. Incoming files are stored in SDCard bluetooth/. An explicit user accept is required for each incoming file. This might be sufficient functionality to use to push files from a Kiosk, provided the kiosk can track status/progress programmatically to provide its own feedback (and notifications are concealed). I think I have seen [examples which specify a bluetooth MAC address in the original intent](http://www.kpbird.com/2011/04/android-send-image-via-bluetooth.html), which suggests that the app's own discovery/selection stages can be replaced within the kiosk without replacing the complete functionality.

Bluetooth share is licensed under BSD 3-clause license, so is also open for derivation.

### Android 3rd party application support

3rd Party Android Apps are available (but not apparently open source) which support a broader range of transfers, including [Bluetooth File Transfer](https://play.google.com/store/apps/details?id=it.medieval.blueftp) and [ES File Explorer File Manager](https://play.google.com/store/apps/details?id=com.estrongs.android.pop). These may also allow (e.g.) APKs to be transfered.


