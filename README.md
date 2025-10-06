# can-beeware-for-android-receive-intents

This repo is the ongoing attempt to connect beeware (https://beeware.org/) to Java using Chaquopy  https://chaquo.com/chaquopy/

The aim is to receive an Android intent and prove that this was received.

The code only needs to work on Android hardware.

Goal:
####
The hardware has a scanner module that can scan barcodes.

The hardware sends scans as intents to `com.example.EVENT`
The intent has a `EXTRA_EVENT_DECODE_VALUE` field that contains the barcode content.
This is a byte array. It also has a `EXTRA_EVENT_STRING_VALUE` which is a string and might
be easier to work with. Later the bytes will be needed.

The code running in the Toga form should receive the intent and display the barcode content. 
It's acceptable if the content is just printed to the console debugger or showing it as an alert.
It's OK if I need to push a button in the form to get the event.

When I connect to ADB with this command

`adb logcat -v time ActivityManager:I Broadcast*:I *:S | findstr /i "com.example.EVENT"`

(on linux replace `findstr` with `grep`)

I can see the intent being received when I push the scan button on the hardware and scan a barcode.

10-05 00:27:32.499 E/ActivityManager( 1405): Sending non-protected broadcast com.example.EVENT from system 1405:system/1000 pkg android

It's OK to have more than one window open in ADB. 

via ADB I can send an intent to the app. (and can see that in the ADB log)

`adb shell am broadcast -a com.example.EVENT --es EXTRA_EVENT_STRING_VALUE "your_test_value"`



Example Java code to receive an intent:

```
# src/yourapp/scanner/receiver.py
from java import static_proxy, Override, jvoid, jarray, jbyte
from android.content import BroadcastReceiver, Context, Intent

# This generates a Java class: com.example.scanner.PyScannerReceiver
# Change the package=... to your desired Java package.
class PyScannerReceiver(static_proxy(BroadcastReceiver, package="com.example.scanner")):
    # a simple stash you can read from your Toga code
    last_scan: bytes | None = None

    @Override(jvoid, [Context, Intent])
    def onReceive(self, ctx, intent):
        if intent is None:
            return
        extras = intent.getExtras()
        if extras is None:
            return

        if extras.containsKey("EXTRA_EVENT_DECODE_VALUE"):
            b = intent.getByteArrayExtra("EXTRA_EVENT_DECODE_VALUE")  # byte[]
            # Convert Java byte[] -> Python bytes
            PyScannerReceiver.last_scan = bytes(jarray(jbyte)(b))
```

How to proceed:

Follow the beeware tutorial to create a new Android app until step 5. (https://docs.beeware.org/en/latest/tutorial/tutorial-0.html)

Now try to add intent receiving code.

It's probably easier to try with a string value like EXTRA_EVENT_STRING_VALUE to get things working first.
---------

Hints: 

Look at https://toga.readthedocs.io/en/stable/reference/platforms/android.html
for an example to make a call via Intent
and https://developer.android.com/reference/android/content/Intent for Java descriptions

