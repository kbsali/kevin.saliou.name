Samsung Galaxy Nexus 4.1.1 upgrade
==================================

This small tutorial shows how to upgrade your galaxy nexus from *any* version to Jelly Bean 4.1.1 (of course, the very same procedure applies to any given version).


**Note that this procedure wipes out all your data, this is not a regular upgrade solution.**

Steps :

* If not done already, download + install Android SDK
* If not done already, download fastboot
* Download the factory image
  * V4.1.1 Factory Images "yakju" for Galaxy Nexus "maguro" (GSM/HSPA+)
  * from https://developers.google.com/android/nexus/images#yakju
* Plug mobile to your comptuer
  * Install it all! :)

Isolate it all
--------------
    mkdir TEMP
    cd TEMP

Android SDK + Fastboot
----------------------
    wget http://dl.google.com/android/android-sdk_r20.0.3-linux.tgz
    tar cfzv android-sdk_r20.0.3-linux.tgz
    android-sdk-linux/tools/android update sdk
    wget http://dl.dropbox.com/u/3930957/fastboot # Get fastboot
    mv fastboot android-sdk-linux/tools
    chmod +x android-sdk-linux/tools/fastboot
    chmod +x android-sdk-linux/platform-tools/adb
    vi ~/.bashrc
    .. --- ~/.bashrc ---
      export PATH=${PATH}:/PATH/android-sdk-linux/tools
      export PATH=${PATH}:/PATH/android-sdk-linux/platform-tools
    ..
    source ~/.bashrc

Download Factory image
----------------------
    cd ..
    wget https://dl.google.com/dl/android/aosp/yakju-jro03c-factory-3174c1e5.tgzq
    tar cfzv yakju-jro03c-factory-3174c1e5.tgz
    cd yakju-jro03c
    unzip image-yakju-jro03c.zip

Prepare USB driver
----------------------
    sudo vi /etc/udev/rules.d/70-android.rules
    .. --- /etc/udev/rules.d/70-android.rules ---
      # fastboot protocol on maguro (Galaxy Nexus) -> username = whatever you'd like
      SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e30", MODE="0666", OWNER="username"
    ..
    sudo chmod a+rx /etc/udev/rules.d/70-android.rules
    sudo service udev restart
    sudo killall adb

Install image to device
-----------------------
    adb devices # list any devices connected to your computer if any!
    adb reboot bootloader # restarts your mobile in bootloader mode == 1) turn off, 2) vol-up+vol-down+power button
    cd yakju-jro03c
    fastboot devices # list devices connected in bootloader mode
    fastboot oem unlock
    fastboot reboot-bootloader
    fastboot flash bootloader bootloader-maguro-primela03.img
    fastboot reboot-bootloader
    fastboot flash radio radio-maguro-i9250xxlf1.img
    fastboot reboot-bootloader
    fastboot flash system system.img
    fastboot flash userdata userdata.img
    fastboot flash boot boot.img
    fastboot flash recovery recovery.img
    fastboot erase cache
    fastboot reboot

References
-----------------------
* [\[GUIDE\] Install Java, Android SDK, ADB, and Fastboot in Linux Ubuntu and Mint12](http://rootzwiki.com/topic/20770-guideinstall-java-android-sdk-adb-and-fastboot-in-linux-ubuntu-and-mint12/)
* [Simple linux fastboot installing command - MobileRead Forums](http://www.mobileread.com/forums/showthread.php?t=174923)
* [\[GUIDE\] Return Your GSM Nexus to Stock / Flash to "takju"](http://forums.androidcentral.com/google-galaxy-nexus/192673-guide-return-your-gsm-nexus-stock-flash-takju.html)

DONE :)