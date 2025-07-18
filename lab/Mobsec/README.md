# Batch 2 - Mobile Application Security
Repository ini berisi kumpulan lab exercise untuk pelatihan **Mobile Application Security** (Batch 2).  
Pada repo ini, berisikan informasi terkait kebutuhan lab dan cheatsheet untuk mengerjakan lab.

## 📚 Artikel pendukung
Follow below steps to setup Burp to intercept network traffic on an Android 13 device and based on:
- https://portswigger.net/burp/documentation/desktop/mobile/config-android-device
- https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/install-burp-certificate#on-a-virtual-machine

## 📚 Lab Requirements
- Android Emulator / Rooted Android device
- Android Debug Bridge (adb) installed
- Download Insecured apk di section **Daftar Lab Exercise**

## 📚 Daftar Lab Exercise
- **DIVA (Damn insecure and vulnerable App)** 
```sh
wget http://www.payatu.com/wp-content/uploads/2016/01/diva-beta.tar.gz
```
- **InjuredAndroid**
```sh
wget https://github.com/B3nac/InjuredAndroid/releases/download/v1.0.12/InjuredAndroid-1.0.12-release.apk
```
- **Insecure Bank v2**
```sh
wget https://github.com/dineshshetty/Android-InsecureBankv2/releases/download/2.3.1/InsecureBankv2.apk
```

## 📚 Cheatsheet
- Connect adb from kali linux
```sh
adb connect 192.168.56.101:5555
adb devices
adb shell
```
- Install Docker and MobSF
```sh
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker –now
sudo usermod -aG docker $USER

docker pull opensecurity/mobile-security-framework-mobsf:latest
```
- adb cheatsheet
```sh
#Basic adb Commands
adb connect <IP>:5555      # Connect to a device/emulator over network
adb devices                # List connected devices/emulators
adb disconnect             # Disconnect all devices
adb kill-server            # Restart the ADB server
adb start-server
adb shell                  # Start a remote shell on device

#Install/Uninstall Apps
adb install app.apk        # Install an APK
adb install -r app.apk     # Reinstall an APK (keep data)
adb uninstall <package>    # Uninstall app by package name

#File Management
adb push <local> <remote>  # Copy file from PC to device
adb pull <remote> <local>  # Copy file from device to PC

#Log & Debug
adb logcat                 # View device logs
adb logcat -d > logs.txt   # Save logs to file
adb bugreport              # Full device bug report
```
- sqlite
```sh
.tables
.schema <table_name>
SELECT * FROM <table_name>;
.dump
SELECT COUNT(*) FROM <table_name>;
```

- Frida Installation
```sh
#Frida client
pip3 install frida-tools
pip3 install objection
frida --version

#Frida server
1. Download https://github.com/frida/frida/releases
2. find frida-server-{version}-android-{architecture_version}.xz
ex:
 frida-server-17.2.11-android-arm.xz
 frida-server-17.2.11-android-x86.xz

#Push frida-server to your Android device & start it
mv frida-server-17.2.11-android-arm frida-server
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```
- Frida Cheatsheet
```sh
frida-ps -Uai

frida -U {application_package}
frida -l script.js

frida-trace -i "open" -U {application_package}
```