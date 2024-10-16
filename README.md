# JC4827W543_board
 ESP32-S3 with 4.3' TFT 480x270 driver NV3041A and capacitive touch
 Read Docs/Getting started.pdf
 
![JC4827W543_board](/Pictures/1-1.jpg)
![JC4827W543_board](/Pictures/1-2.png)

+ 4.3-inch LCD-TFT (display driver NV3041A  4-bit parallel SPI)
+ Supports WiFi and Bluetooth
+ Support lithium battery power supply
+ Capacitive touch (driver GT911 I2C)
+ or Resistance touch (driver XPT2046)
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
+ for capacitive touch: TouchLib by mmMicky add library from zip file  https://github.com/mmMicky/TouchLib
+ for resistance touch: XPT2046_Touchscreen by Paul Stoffregen (ver. 1.4) https://github.com/PaulStoffregen/XPT2046_Touchscreen.git
+ for resistance touch modify touch.h file as shown below:

```
  /* uncomment for XPT2046 */
 #define TOUCH_XPT2046
 #define TOUCH_XPT2046_SCK 12
 #define TOUCH_XPT2046_MISO 13
 #define TOUCH_XPT2046_MOSI 11
 #define TOUCH_XPT2046_CS 38
 #define TOUCH_XPT2046_INT 3
 #define TOUCH_XPT2046_ROTATION 0
 #define TOUCH_XPT2046_SAMPLES 50

//uncomment for most capacitive touchscreen
//#define TOUCH_MODULES_GT911 // GT911 / CST_SELF / CST_MUTUAL / ZTW622 / L58 / FT3267 / FT5x06
//#define TOUCH_MODULE_ADDR GT911_SLAVE_ADDRESS1 // CTS328_SLAVE_ADDRESS / L58_SLAVE_ADDRESS / CTS826_SLAVE_ADDRESS / CTS820_SLAVE_ADDRESS / CTS816S_SLAVE_ADDRESS / FT3267_SLAVE_ADDRESS / FT5x06_ADDR / GT911_SLAVE_ADDRESS1 / GT911_SLAVE_ADDRESS2 / ZTW622_SLAVE1_ADDRESS / ZTW622_SLAVE2_ADDRESS
//#define TOUCH_SCL 4
// #define TOUCH_SDA 8
// #define TOUCH_RES 38
// #define TOUCH_INT 3
```
+ Please config the touch panel in touch.h
+ use functions from touch.h as shown below:
```
#include "touch.h"
//----------------------------
if (touch_has_signal())
  {
    if (touch_touched())
    {
      // your code here
     }
    else if (touch_released())
    {
      // your code here
    }
  }
//---------------------------
```

+ Move lv_conf.h file (from Examples/Demo_Arduino
/Libraries folder) to ..../Arduino/libraries directory

WARNING: install lvgl ver 8.4.0 !!!

![JC4827W543_board](/Pictures/Arduino.png)

## Fast DMA drawing LVGL example
In lv_conf.h file set:
```
/*Swap the 2 bytes of RGB565 color. Useful if the display has an 8-bit interface (e.g. SPI)*/
#define LV_COLOR_16_SWAP 0
```
Example:
```
/*******************************************************************************
Created by profi-max (Oleg Linnik) 2024
https://profimaxblog.ru
https://github.com/profi-max

*******************************************************************************/
#include "lv_demo_widgets.h"
#include <Arduino_GFX_Library.h>
#include "touch.h"

// FOR ARDUINO Uncomment the line below if you wish debug messages
//#define CORE_DEBUG_LEVEL 4

// FOR VSCODE see platformio.ini file -D CORE_DEBUG_LEVEL=1

/***************************************************************************************************
 * Arduino_GFX Setup for JC4827W543C
 ***************************************************************************************************/
#define SCREEN_WIDTH 480
#define SCREEN_HEIGHT 272
#define SCR_BUF_LEN 32

// LCD backlight PWM 
#define LCD_BL 1           // lcd BL pin
#define LEDC_CHANNEL_0     0 // use first channel of 16 channels (started from zero)
#define LEDC_TIMER_12_BIT  12 // use 12 bit precission for LEDC timer
#define LEDC_BASE_FREQ     5000 // use 5000 Hz as a LEDC base frequency


Arduino_DataBus *bus = new Arduino_ESP32QSPI(
    45 /* cs */, 47 /* sck */, 21 /* d0 */, 48 /* d1 */, 40 /* d2 */, 39 /* d3 */);
Arduino_NV3041A *panel = new Arduino_NV3041A(bus, GFX_NOT_DEFINED /* RST */, 0 /* rotation */, true /* IPS */);
Arduino_GFX *gfx = new Arduino_Canvas(SCREEN_WIDTH /* width */, SCREEN_HEIGHT /* height */, panel);
//------------------------------- End TFT+TOUCH setup ------------------------------

static const char* logTAG = "main.cpp";

/***************************************************************************************************
 * LVGL functions
 ***************************************************************************************************/

//=====================================================================================================================
void IRAM_ATTR my_disp_flush_(lv_disp_drv_t *disp, const lv_area_t *area, lv_color_t *color_p)
{
  uint32_t w = (area->x2 - area->x1 + 1);
  uint32_t h = (area->y2 - area->y1 + 1);

  panel->startWrite();
  panel->setAddrWindow(area->x1, area->y1, w, h); 
  panel->writePixels((uint16_t *)&color_p->full, w * h);
  panel->endWrite();
  lv_disp_flush_ready(disp); /* tell lvgl that flushing is done */
}
//=====================================================================================================================
void IRAM_ATTR my_touchpad_read_(lv_indev_drv_t *indev_driver, lv_indev_data_t *data)
{
  if (touch_has_signal())
  {
    if (touch_touched())
    {
      data->state = LV_INDEV_STATE_PR;
      data->point.x = touch_last_x;
      data->point.y = touch_last_y;
    }
    else if (touch_released())
    {
      data->state = LV_INDEV_STATE_REL;
    }
  }
  else
  {
    data->state = LV_INDEV_STATE_REL;
  }
}
//=====================================================================================================================
void IRAM_ATTR my_touchpad_feedback_(lv_indev_drv_t *indev_driver, uint8_t data)
{
  // Beep sound here
}
//=====================================================================================================================
// value has to be between 0 and 255
void setBrightness(uint8_t value)
{
  uint32_t duty = 4095 * value / 255;
  ledcWrite(LEDC_CHANNEL_0, duty);
}
//=====================================================================================================================

/***************************************************************************************************
 * LVGL Setup
 ***************************************************************************************************/

//=====================================================================================================================
void lvgl_init()
{
  static lv_disp_draw_buf_t draw_buf;
  static lv_disp_drv_t disp_drv;
  static lv_color_t disp_draw_buf[SCREEN_WIDTH * SCR_BUF_LEN];
  //static lv_color_t disp_draw_buf2[SCREEN_WIDTH * SCR_BUF_LEN];

  // Setting up the LEDC and configuring the Back light pin
  ledcSetup(LEDC_CHANNEL_0, LEDC_BASE_FREQ, LEDC_TIMER_12_BIT);
  ledcAttachPin(LCD_BL, LEDC_CHANNEL_0);
  setBrightness(250);

   // Init Display
  if (!gfx->begin())
  {
    ESP_LOGI(logTAG, "gfx->begin() failed!\n");  
  }
  gfx->fillScreen(BLACK);

 
  touch_init(gfx->width(), gfx->height(), gfx->getRotation());
  lv_init(); 

  if (!disp_draw_buf)  ESP_LOGE(logTAG, "LVGL disp_draw_buf allocate failed!"); 
  else 
  {
    lv_disp_draw_buf_init(&draw_buf, disp_draw_buf, NULL, SCREEN_WIDTH * SCR_BUF_LEN);

    /* Initialize the display */
    lv_disp_drv_init(&disp_drv);
    disp_drv.hor_res = SCREEN_WIDTH;
    disp_drv.ver_res = SCREEN_HEIGHT;
    disp_drv.flush_cb = my_disp_flush_;
    disp_drv.draw_buf = &draw_buf;
    lv_disp_drv_register(&disp_drv);

    /* Initialize the input device driver */
    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touchpad_read_;
    indev_drv.feedback_cb = my_touchpad_feedback_;
    lv_indev_drv_register(&indev_drv);
  }

   ESP_LOGI(logTAG, "Lvgl v%d.%d.%d initialized\n", lv_version_major(), lv_version_minor(), lv_version_patch());  
}
//=====================================================================================================================

void setup()
{
  Serial.begin(115200);
  lvgl_init();
  lv_demo_widgets();
  ESP_LOGI(logTAG, "Setup done");
}

//=====================================================================================================================
void loop()
{
  lv_timer_handler(); /* let the GUI do its work */
  delay(5);
}

//=====================================================================================================================

```

## Video example
https://github.com/user-attachments/assets/85a67478-a86b-4d5d-81d4-401bac6c66d0

