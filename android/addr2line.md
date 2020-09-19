# addr2line
* 32bit so
```
./prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin/arm-eabi-addr2line -f -e ./out/target/product/tulip-pax/symbols/system/lib64/libril.so 0xed60
```
* 64bit so
```
./prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-addr2line -f -e out/target/product/tulip-pax/symbols/system/lib64/libril.so 0xed60
_ZL11firePendingv
/proc/self/cwd/hardware/ril/libril/ril_event.cpp:209
 
./prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-addr2line -f -e out/target/product/tulip-pax/symbols/system/lib64/libreference-ril.so 0xaca0
get_modem_type
/proc/self/cwd/hardware/ril/reference-ril/reference-ril.c:10165
```
 
* backtrace  
```
backtrace:
    #00 pc 000000000001aa58  /system/lib64/libc.so (memcpy+336)
    #01 pc 000000000005dffc  /system/lib64/libbinder.so (_ZN7android6Parcel5writeEPKvm+68)
    #02 pc 000000000001b4f4  /system/bin/paxservice
    #03 pc 000000000000a57c  /system/bin/paxservice //这个用如下
 
$:~/project/a64/ap/android$ ./prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android-addr2line -Cfpi -e ./out/target/product/tulip-pax/symbols/system/bin/paxservice 0x1b4f4
PaxApiService::PiccCmdExchange(android::Parcel const&, android::Parcel*) at /proc/self/cwd/paxdroid/external/pax/lib/libpaxapisvr/paxservice.cpp:4670
```