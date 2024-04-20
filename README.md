# JC4827W543_board
 ESP32-S3 with 4.3' TFT 480x270 driver NV3041A and capacitive touch

 Read Docs/Getting started.pdf

 Board info via USB:   u-blox NORA-W10 series (ESP32-S3)

![JC4827W543_board](/Pictures/1-1.jpg)
![JC4827W543_board](/Pictures/1-2.png)

+ 4.3-inch LCD-TFT (display driver NV3041A  4-bit parallel)
+ Supports WiFi and Bluetooth
+ Support lithium battery power supply
+ Capacitive touch or Resistance touch (Capacitive touch driver GT911 I2C)
+ 480 x 272 screen resolution
+ RGB 65K 16 bit
+ Onboard 240MHz ESP32-S3-WROOM-1-N4R8 dual-core
+ integrated WI-FI and Bluetooth
+ 520K Byte RAM
+ 8 MB PSRAM (OSPI)
+ 4 MB Flash memory (QSPI)
+ Operating Voltage 5V
+ Power consumption 260 mA
+ Module size 120x70.2(mm)
+ Up to 10 IOs can be used
+ JST1.25 4 Pins 

Product spec download links:
http://www.jczn1688.com/zlxz

https://aliexpress.ru/item/1005006729657546.html

1.25mm 4pin 30cm,  1.25mm 2pin 30cm

https://aliexpress.ru/item/1005004837211340.html

![JC4827W543_board](/Pictures/1-3.png)



## Arduino example 3_3-3_TFT-LVGL-Widgets
Select board:
+ u-blox NORA-W10 series (ESP32-S3)
OR
+ ESP32S3 Dev module
Need install library:
+ GFX Library for Arduino by Moon On Our Nation (ver 1.4.6)  https://github.com/moononournation/Arduino_GFX
+ lvgl by kisvegabor (ver 8.4.0)    https://github.com/lvgl/lvgl/releases/tag/v8.4.0
+ TouchLib by mmMicky add library from zip file  https://github.com/mmMicky/TouchLib

Move lv_conf.h file to ..../Arduino/libraries directory

WARNING: install lvgl ver 8.4.0 !!!

![JC4827W543_board](/Pictures/Arduino.png)

