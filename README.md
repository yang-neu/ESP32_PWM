# ESP32_PWM Library

[![arduino-library-badge](https://www.ardu-badge.com/badge/ESP32_PWM.svg?)](https://www.ardu-badge.com/ESP32_PWM)
[![GitHub release](https://img.shields.io/github/release/khoih-prog/ESP32_PWM.svg)](https://github.com/khoih-prog/ESP32_PWM/releases)
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/khoih-prog/ESP32_PWM/blob/master/LICENSE)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](#Contributing)
[![GitHub issues](https://img.shields.io/github/issues/khoih-prog/ESP32_PWM.svg)](http://github.com/khoih-prog/ESP32_PWM/issues)

---
---

## Table of Contents

* [Important Change from v1.2.0](#Important-Change-from-v120)
* [Why do we need this ESP32_PWM library](#why-do-we-need-this-ESP32_PWM-library)
  * [Why using ISR-based PWM-channels is better](#Why-using-ISR-based-PWM-channels-is-better)
  * [Currently supported Boards](#currently-supported-boards)
  * [Important Notes about ISR](#important-notes-about-isr)
* [Changelog](changelog.md)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
  * [Use Arduino Library Manager](#use-arduino-library-manager)
  * [Manual Install](#manual-install)
  * [VS Code & PlatformIO](#vs-code--platformio)
* [HOWTO Fix `Multiple Definitions` Linker Error](#howto-fix-multiple-definitions-linker-error)
* [HOWTO Use analogRead() with ESP32 running WiFi and/or BlueTooth (BT/BLE)](#howto-use-analogread-with-esp32-running-wifi-andor-bluetooth-btble)
  * [1. ESP32 has 2 ADCs, named ADC1 and ADC2](#1--esp32-has-2-adcs-named-adc1-and-adc2)
  * [2. ESP32 ADCs functions](#2-esp32-adcs-functions)
  * [3. ESP32 WiFi uses ADC2 for WiFi functions](#3-esp32-wifi-uses-adc2-for-wifi-functions)
* [More useful Information](#more-useful-information)
* [How to use](#how-to-use)
* [Examples](#examples)
  * [ 1. ISR_16_PWMs_Array](examples/ISR_16_PWMs_Array)
  * [ 2. ISR_16_PWMs_Array_Complex](examples/ISR_16_PWMs_Array_Complex)
  * [ 3. ISR_16_PWMs_Array_Simple](examples/ISR_16_PWMs_Array_Simple)
  * [ 4. ISR_Changing_PWM](examples/ISR_Changing_PWM)
  * [ 5. ISR_Modify_PWM](examples/ISR_Modify_PWM)
  * [ 6. multiFileProject](examples/multiFileProject) **New**
* [Example ISR_16_PWMs_Array_Complex](#Example-ISR_16_PWMs_Array_Complex)
* [Debug Terminal Output Samples](#debug-terminal-output-samples)
  * [1. ISR_16_PWMs_Array_Complex on ESP32_DEV](#1-ISR_16_PWMs_Array_Complex-on-ESP32_DEV)
  * [2. ISR_16_PWMs_Array on ESP32_DEV](#2-ISR_16_PWMs_Array-on-ESP32_DEV)
  * [3. ISR_16_PWMs_Array_Simple on ESP32_DEV](#3-ISR_16_PWMs_Array_Simple-on-ESP32_DEV)
  * [4. ISR_Modify_PWM on ESP32_DEV](#4-ISR_Modify_PWM-on-ESP32_DEV)
  * [5. ISR_Changing_PWM on ESP32_DEV](#5-ISR_Changing_PWM-on-ESP32_DEV)
  * [6. ISR_Modify_PWM on ESP32S2_DEV](#6-ISR_Modify_PWM-on-ESP32S2_DEV)
  * [7. ISR_Changing_PWM on ESP32S2_DEV](#7-ISR_Changing_PWM-on-ESP32S2_DEV)
  * [8. ISR_Modify_PWM on ESP32C3_DEV](#8-ISR_Modify_PWM-on-ESP32C3_DEV)
  * [9. ISR_Changing_PWM on ESP32C3_DEV](#9-ISR_Changing_PWM-on-ESP32C3_DEV)
* [Debug](#debug)
* [Troubleshooting](#troubleshooting)
* [Issues](#issues)
* [TO DO](#to-do)
* [DONE](#done)
* [Contributions and Thanks](#contributions-and-thanks)
* [Contributing](#contributing)
* [License](#license)
* [Copyright](#copyright)

---
---

### Important Change from v1.2.0

Please have a look at [HOWTO Fix `Multiple Definitions` Linker Error](#howto-fix-multiple-definitions-linker-error)

---
---

### Why do we need this [ESP32_PWM library](https://github.com/khoih-prog/ESP32_PWM)

### Features

This library enables you to use Interrupt from Hardware Timers on an ESP32, ESP32_S2-based board to create and output PWM to pins. Becayse this library doesn't use the powerful hardware-controlled PWM with limitations, the maximum PWM frequency is currently limited at **500Hz**, which is suitable for many real-life applications. Now you can also modify PWM settings on-the-fly.

---

This library enables you to use Interrupt from Hardware Timers on an ESP32, ESP32_S2 or ESP32_C3-based board to create and output PWM to pins. It now supports 16 ISR-based synchronized PWM channels, while consuming only 1 Hardware Timer. PWM interval can be very long (uint32_t millisecs). The most important feature is they're ISR-based PWM channels. Therefore, their executions are not blocked by bad-behaving functions or tasks. This important feature is absolutely necessary for mission-critical tasks. These hardware PWM channels, using interrupt, still work even if other functions are blocking. Moreover, they are much more precise (certainly depending on clock frequency accuracy) than other software timers using millis() or micros(). That's necessary if you need to measure some data requiring better accuracy.

As **Hardware Timers are rare, and very precious assets** of any board, this library now enables you to use up to **16 ISR-based synchronized PWM channels, while consuming only 1 Hardware Timer**. Timers' interval is very long (**ulong millisecs**).

Now with these new **16 ISR-based timers**, the maximum interval is **practically unlimited** (limited only by unsigned long miliseconds) while **the accuracy is nearly perfect** compared to software timers. 

The most important feature is they're ISR-based PWM channels. Therefore, their executions are **not blocked by bad-behaving functions / tasks**. This important feature is absolutely necessary for mission-critical tasks. 

The [**ISR_16_PWMs_Array_Complex**](examples/ISR_16_PWMs_Array_Complex) example will demonstrate the nearly perfect accuracy compared to software timers by printing the actual elapsed millisecs of each type of PWM channels.

Being ISR-based timers, their executions are not blocked by bad-behaving functions / tasks, such as connecting to WiFi, Internet and Blynk services. You can also have many `(up to 16)` timers to use.

This non-being-blocked important feature is absolutely necessary for mission-critical tasks.


### Why using ISR-based PWM-channels is better

Imagine you have a system with a **mission-critical** function, measuring water level and control the sump pump or doing something much more important. You normally use a software timer to poll, or even place the function in loop(). But what if another function is **blocking** the loop() or setup().

So your function **might not be executed, and the result would be disastrous.**

You'd prefer to have your function called, no matter what happening with other functions (busy loop, bug, etc.).

The correct choice is to use a Hardware PWM-channels with **Interrupt** to call your function.

These hardware timers, using interrupt, still work even if other functions are blocking. Moreover, they are much more **precise** (certainly depending on clock frequency accuracy) than other software timers using millis() or micros(). That's necessary if you need to measure some data requiring better accuracy.

Functions using normal software timers, relying on loop() and calling millis(), won't work if the loop() or setup() is blocked by certain operation. For example, certain function is blocking while it's connecting to WiFi or some services.

The catch is **your function is now part of an ISR (Interrupt Service Routine), and must be lean / mean, and follow certain rules.** More to read on:

[**HOWTO Attach Interrupt**](https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/)

---

### Currently supported Boards

1. ESP32 boards, such as `ESP32_DEV`, etc.
2. ESP32S2-based boards, such as `ESP32S2_DEV`, `ESP32_S2 Saola`, etc.
3. ESP32C3-based boards, such as `ESP32C3_DEV`, etc. **New**

---

### Important Notes about ISR

1. Inside the attached function, **delay() won’t work and the value returned by millis() will not increment.** Serial data received while in the function may be lost. You should declare as **volatile any variables that you modify within the attached function.**

2. Typically global variables are used to pass data between an ISR and the main program. To make sure variables shared between an ISR and the main program are updated correctly, declare them as volatile.


---
---

## Prerequisites

1. [`Arduino IDE 1.8.19+` for Arduino](https://github.com/arduino/Arduino). [![GitHub release](https://img.shields.io/github/release/arduino/Arduino.svg)](https://github.com/arduino/Arduino/releases/latest)
2. [`ESP32 Core 2.0.2+`](https://github.com/espressif/arduino-esp32) for ESP32-based boards. [![Latest release](https://img.shields.io/github/release/espressif/arduino-esp32.svg)](https://github.com/espressif/arduino-esp32/releases/latest/).
3. [`SimpleTimer library`](https://github.com/jfturcot/SimpleTimer) to use with some examples.


---
---

## Installation

### Use Arduino Library Manager

The best and easiest way is to use `Arduino Library Manager`. Search for [**ESP32_PWM**](https://github.com/khoih-prog/ESP32_PWM), then select / install the latest version.
You can also use this link [![arduino-library-badge](https://www.ardu-badge.com/badge/ESP32_PWM.svg?)](https://www.ardu-badge.com/ESP32_PWM) for more detailed instructions.

### Manual Install

Another way to install is to:

1. Navigate to [**ESP32_PWM**](https://github.com/khoih-prog/ESP32_PWM) page.
2. Download the latest release `ESP32_PWM-master.zip`.
3. Extract the zip file to `ESP32_PWM-master` directory 
4. Copy whole `ESP32_PWM-master` folder to Arduino libraries' directory such as `~/Arduino/libraries/`.

### VS Code & PlatformIO

1. Install [VS Code](https://code.visualstudio.com/)
2. Install [PlatformIO](https://platformio.org/platformio-ide)
3. Install [**ESP32_PWM** library](https://platformio.org/lib/show/12846/ESP32_PWM) by using [Library Manager](https://platformio.org/lib/show/12846/ESP32_PWM/installation). Search for **ESP32_PWM** in [Platform.io Author's Libraries](https://platformio.org/lib/search?query=author:%22Khoi%20Hoang%22)
4. Use included [platformio.ini](platformio/platformio.ini) file from examples to ensure that all dependent libraries will installed automatically. Please visit documentation for the other options and examples at [Project Configuration File](https://docs.platformio.org/page/projectconf.html)


---
---


### HOWTO Fix `Multiple Definitions` Linker Error

The current library implementation, using `xyz-Impl.h` instead of standard `xyz.cpp`, possibly creates certain `Multiple Definitions` Linker error in certain use cases.

You can include this `.hpp` file

```
// Can be included as many times as necessary, without `Multiple Definitions` Linker Error
#include "ESP32_PWM.hpp"     //https://github.com/khoih-prog/ESP32_PWM
```

in many files. But be sure to use the following `.h` file **in just 1 `.h`, `.cpp` or `.ino` file**, which must **not be included in any other file**, to avoid `Multiple Definitions` Linker Error

```
// To be included only in main(), .ino with setup() to avoid `Multiple Definitions` Linker Error
#include "ESP32_PWM.h"           //https://github.com/khoih-prog/ESP32_PWM
```

Check the new [**multiFileProject** example](examples/multiFileProject) for a `HOWTO` demo.

Have a look at the discussion in [Different behaviour using the src_cpp or src_h lib #80](https://github.com/khoih-prog/ESPAsync_WiFiManager/discussions/80)

---
---

### HOWTO Use analogRead() with ESP32 running WiFi and/or BlueTooth (BT/BLE)

Please have a look at [**ESP_WiFiManager Issue 39: Not able to read analog port when using the autoconnect example**](https://github.com/khoih-prog/ESP_WiFiManager/issues/39) to have more detailed description and solution of the issue.

#### 1.  ESP32 has 2 ADCs, named ADC1 and ADC2

#### 2. ESP32 ADCs functions

- ADC1 controls ADC function for pins **GPIO32-GPIO39**
- ADC2 controls ADC function for pins **GPIO0, 2, 4, 12-15, 25-27**

#### 3.. ESP32 WiFi uses ADC2 for WiFi functions

Look in file [**adc_common.c**](https://github.com/espressif/esp-idf/blob/master/components/driver/adc_common.c#L61)

> In ADC2, there're two locks used for different cases:
> 1. lock shared with app and Wi-Fi:
>    ESP32:
>         When Wi-Fi using the ADC2, we assume it will never stop, so app checks the lock and returns immediately if failed.
>    ESP32S2:
>         The controller's control over the ADC is determined by the arbiter. There is no need to control by lock.
> 
> 2. lock shared between tasks:
>    when several tasks sharing the ADC2, we want to guarantee
>    all the requests will be handled.
>    Since conversions are short (about 31us), app returns the lock very soon,
>    we use a spinlock to stand there waiting to do conversions one by one.
> 
> adc2_spinlock should be acquired first, then adc2_wifi_lock or rtc_spinlock.


- In order to use ADC2 for other functions, we have to **acquire complicated firmware locks and very difficult to do**
- So, it's not advisable to use ADC2 with WiFi/BlueTooth (BT/BLE).
- Use ADC1, and pins GPIO32-GPIO39
- If somehow it's a must to use those pins serviced by ADC2 (**GPIO0, 2, 4, 12, 13, 14, 15, 25, 26 and 27**), use the **fix mentioned at the end** of [**ESP_WiFiManager Issue 39: Not able to read analog port when using the autoconnect example**](https://github.com/khoih-prog/ESP_WiFiManager/issues/39) to work with ESP32 WiFi/BlueTooth (BT/BLE).

---
---

## More useful Information

### ESP32 Hardware Timers

  - **The ESP32 and ESP32_S2 has two timer groups, each one with two general purpose hardware timers.**
  - **The ESP32_C3 has two timer groups, each one with only one general purpose hardware timer.**
  - All the timers are based on **64-bit counters and 16-bit prescalers.**
  - The timer counters can be configured to count up or down and support automatic reload and software reload.
  - They can also generate alarms when they reach a specific value, defined by the software. 
  - The value of the counter can be read by the software program.

---

Now with these new `16 ISR-based PWM-channels` (while consuming only **1 hardware timer**), the maximum interval is practically unlimited (limited only by unsigned long miliseconds). The accuracy is nearly perfect compared to software PWM-channels. The most important feature is they're ISR-based PWM-channels Therefore, their executions are not blocked by bad-behaving functions / tasks.
This important feature is absolutely necessary for mission-critical tasks. 

The [**ISR_16_PWMs_Array_Complex**](examples/ISR_16_PWMs_Array_Complex) example will demonstrate the nearly perfect accuracy compared to software-based PWM-channels by printing the actual elapsed `microsecs / millisecs` of each type of PWM-channels.
Being ISR-based PWM-channels, their executions are not blocked by bad-behaving functions / tasks, such as connecting to WiFi, Internet and Blynk services. You can also have many `(up to 16)` synchronized PWM-channels to use.
This non-being-blocked important feature is absolutely necessary for mission-critical tasks. 
You'll see `SimpleTimer` is blocked while system is connecting to WiFi / Internet / Blynk, as well as by blocking task 
in loop(), using delay() function as an example. The elapsed time then is very unaccurate

---
---

## How to use

Before using any Timer, you have to make sure the Timer has not been used by any other purpose.

`Timer0, Timer1, Timer2 and Timer3` are supported for ESP32

---
---

### Examples: 

 1. [ISR_16_PWMs_Array](examples/ISR_16_PWMs_Array)
 2. [ISR_16_PWMs_Array_Complex](examples/ISR_16_PWMs_Array_Complex)
 3. [ISR_16_PWMs_Array_Simple](examples/ISR_16_PWMs_Array_Simple)
 4. [ISR_Changing_PWM](examples/ISR_Changing_PWM)
 5. [ISR_Modify_PWM](examples/ISR_Modify_PWM)
 6. [**multiFileProject**](examples/multiFileProject) **New** 

---
---

### Example [ISR_16_PWMs_Array_Complex](examples/ISR_16_PWMs_Array_Complex)

```
#if !defined( ESP32 )
  #error This code is designed to run on ESP32 platform, not Arduino nor ESP8266! Please check your Tools->Board setting. 
#endif

// These define's must be placed at the beginning before #include "ESP32_PWM.h"
// _PWM_LOGLEVEL_ from 0 to 4
// Don't define _PWM_LOGLEVEL_ > 0. Only for special ISR debugging only. Can hang the system.
#define _PWM_LOGLEVEL_      4

#define USING_MICROS_RESOLUTION       true    //false

// Default is true, uncomment to false
//#define CHANGING_PWM_END_OF_CYCLE     false 

// To be included only in main(), .ino with setup() to avoid `Multiple Definitions` Linker Error
#include "ESP32_PWM.h"

#include <SimpleTimer.h>              // https://github.com/jfturcot/SimpleTimer

#ifndef LED_BUILTIN
  #define LED_BUILTIN       2
#endif

#ifndef LED_BLUE
  #define LED_BLUE          25
#endif

#ifndef LED_RED
  #define LED_RED           27
#endif

#define HW_TIMER_INTERVAL_US      20L

volatile uint32_t startMicros = 0;

// Init ESP32 timer 1
ESP32Timer ITimer(1);

// Init ESP32_ISR_PWM
ESP32_PWM ISR_PWM;

bool IRAM_ATTR TimerHandler(void * timerNo)
{ 
  ISR_PWM.run();
 
  return true;
}

/////////////////////////////////////////////////

#if ( ARDUINO_ESP32C3_DEV )
  #define NUMBER_ISR_PWMS         4
#else
  #define NUMBER_ISR_PWMS         16
#endif

#define PIN_D0            0         // Pin D0 mapped to pin GPIO0/BOOT/ADC11/TOUCH1 of ESP32
#define PIN_D1            1         // Pin D1 mapped to pin GPIO1/TX0 of ESP32
#define PIN_D2            2         // Pin D2 mapped to pin GPIO2/ADC12/TOUCH2 of ESP32
#define PIN_D3            3         // Pin D3 mapped to pin GPIO3/RX0 of ESP32
#define PIN_D4            4         // Pin D4 mapped to pin GPIO4/ADC10/TOUCH0 of ESP32
#define PIN_D5            5         // Pin D5 mapped to pin GPIO5/SPISS/VSPI_SS of ESP32
#define PIN_D12           12        // Pin D12 mapped to pin GPIO12/HSPI_MISO/ADC15/TOUCH5/TDI of ESP32
#define PIN_D13           13        // Pin D13 mapped to pin GPIO13/HSPI_MOSI/ADC14/TOUCH4/TCK of ESP32
#define PIN_D14           14        // Pin D14 mapped to pin GPIO14/HSPI_SCK/ADC16/TOUCH6/TMS of ESP32
#define PIN_D15           15        // Pin D15 mapped to pin GPIO15/HSPI_SS/ADC13/TOUCH3/TDO of ESP32
#define PIN_D16           16        // Pin D16 mapped to pin GPIO16/TX2 of ESP32
#define PIN_D17           17        // Pin D17 mapped to pin GPIO17/RX2 of ESP32     
#define PIN_D18           18        // Pin D18 mapped to pin GPIO18/VSPI_SCK of ESP32
#define PIN_D19           19        // Pin D19 mapped to pin GPIO19/VSPI_MISO of ESP32

#define PIN_D21           21        // Pin D21 mapped to pin GPIO21/SDA of ESP32
#define PIN_D22           22        // Pin D22 mapped to pin GPIO22/SCL of ESP32
#define PIN_D23           23        // Pin D23 mapped to pin GPIO23/VSPI_MOSI of ESP32
#define PIN_D24           24        // Pin D24 mapped to pin GPIO24 of ESP32
#define PIN_D25           25        // Pin D25 mapped to pin GPIO25/ADC18/DAC1 of ESP32
#define PIN_D26           26        // Pin D26 mapped to pin GPIO26/ADC19/DAC2 of ESP32
#define PIN_D27           27        // Pin D27 mapped to pin GPIO27/ADC17/TOUCH7 of ESP32   

typedef void (*irqCallback)  ();

//////////////////////////////////////////////////////

#define USING_PWM_FREQUENCY     false //true

//////////////////////////////////////////////////////

volatile unsigned long deltaMicrosStart    [] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };
volatile unsigned long previousMicrosStart [] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

volatile unsigned long deltaMicrosStop     [] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };
volatile unsigned long previousMicrosStop  [] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

// You can assign pins here. Be carefull to select good pin to use or crash, e.g pin 6-11
// Can't use PIN_D1 for core v2.0.1+

#if ( ARDUINO_ESP32C3_DEV )
uint32_t PWM_Pin[] =
// Bad pins to use: PIN_D12-PIN_D24
{
  LED_BUILTIN, PIN_D3,  PIN_D4,  PIN_D5
};
#else
uint32_t PWM_Pin[] =
{
  PIN_D24, LED_BUILTIN,  PIN_D3,  PIN_D4,  PIN_D5,  PIN_D12, PIN_D13, PIN_D14,
  PIN_D15, PIN_D16,      PIN_D17, PIN_D18, PIN_D19, PIN_D21, PIN_D22, PIN_D23
};
#endif

// You can assign any interval for any timer here, in microseconds
double PWM_Period[] =
{
  1000000.0,     500000.0,   333333.333,   250000.0,   200000.0,   166666.666,   142857.143,   125000.0,
   111111.111,   100000.0,    66666.666,    50000.0,    40000.0,    33333.333,    25000.0,      20000.0
};

// You can assign any interval for any timer here, in Hz
double PWM_Freq[] =
{
  1.0f,  2.0f,  3.0f,  4.0f,  5.0f,  6.0f,  7.0f,  8.0f,
  9.0f, 10.0f, 15.0f, 20.0f, 25.0f, 30.0f, 40.0f, 50.0f
};

// You can assign any interval for any timer here, in milliseconds
double PWM_DutyCycle[] =
{
   5.00, 10.00, 20.00, 30.00, 40.00, 45.00, 50.00, 55.00,
  60.00, 65.00, 70.00, 75.00, 80.00, 85.00, 90.00, 95.00
};

void doingSomethingStart(int index)
{
  unsigned long currentMicros  = micros();

  deltaMicrosStart[index]    = currentMicros - previousMicrosStart[index];
  previousMicrosStart[index] = currentMicros;
}

void doingSomethingStop(int index)
{
  unsigned long currentMicros  = micros();

  // Count from start to stop PWM pulse
  deltaMicrosStop[index]    = currentMicros - previousMicrosStart[index];
  previousMicrosStop[index] = currentMicros;
}

////////////////////////////////////
// Shared
////////////////////////////////////

void doingSomethingStart0()
{
  doingSomethingStart(0);
}

void doingSomethingStart1()
{
  doingSomethingStart(1);
}

void doingSomethingStart2()
{
  doingSomethingStart(2);
}

void doingSomethingStart3()
{
  doingSomethingStart(3);
}

void doingSomethingStart4()
{
  doingSomethingStart(4);
}

void doingSomethingStart5()
{
  doingSomethingStart(5);
}

void doingSomethingStart6()
{
  doingSomethingStart(6);
}

void doingSomethingStart7()
{
  doingSomethingStart(7);
}

void doingSomethingStart8()
{
  doingSomethingStart(8);
}

void doingSomethingStart9()
{
  doingSomethingStart(9);
}

void doingSomethingStart10()
{
  doingSomethingStart(10);
}

void doingSomethingStart11()
{
  doingSomethingStart(11);
}

void doingSomethingStart12()
{
  doingSomethingStart(12);
}

void doingSomethingStart13()
{
  doingSomethingStart(13);
}

void doingSomethingStart14()
{
  doingSomethingStart(14);
}

void doingSomethingStart15()
{
  doingSomethingStart(15);
}

//////////////////////////////////////////////////////

void doingSomethingStop0()
{
  doingSomethingStop(0);
}

void doingSomethingStop1()
{
  doingSomethingStop(1);
}

void doingSomethingStop2()
{
  doingSomethingStop(2);
}

void doingSomethingStop3()
{
  doingSomethingStop(3);
}

void doingSomethingStop4()
{
  doingSomethingStop(4);
}

void doingSomethingStop5()
{
  doingSomethingStop(5);
}

void doingSomethingStop6()
{
  doingSomethingStop(6);
}

void doingSomethingStop7()
{
  doingSomethingStop(7);
}

void doingSomethingStop8()
{
  doingSomethingStop(8);
}

void doingSomethingStop9()
{
  doingSomethingStop(9);
}

void doingSomethingStop10()
{
  doingSomethingStop(10);
}

void doingSomethingStop11()
{
  doingSomethingStop(11);
}

void doingSomethingStop12()
{
  doingSomethingStop(12);
}

void doingSomethingStop13()
{
  doingSomethingStop(13);
}

void doingSomethingStop14()
{
  doingSomethingStop(14);
}

void doingSomethingStop15()
{
  doingSomethingStop(15);
}

//////////////////////////////////////////////////////

irqCallback irqCallbackStartFunc[] =
{
  doingSomethingStart0,  doingSomethingStart1,  doingSomethingStart2,  doingSomethingStart3,
  doingSomethingStart4,  doingSomethingStart5,  doingSomethingStart6,  doingSomethingStart7,
  doingSomethingStart8,  doingSomethingStart9,  doingSomethingStart10, doingSomethingStart11,
  doingSomethingStart12, doingSomethingStart13, doingSomethingStart14, doingSomethingStart15
};

irqCallback irqCallbackStopFunc[] =
{
  doingSomethingStop0,  doingSomethingStop1,  doingSomethingStop2,  doingSomethingStop3,
  doingSomethingStop4,  doingSomethingStop5,  doingSomethingStop6,  doingSomethingStop7,
  doingSomethingStop8,  doingSomethingStop9,  doingSomethingStop10, doingSomethingStop11,
  doingSomethingStop12, doingSomethingStop13, doingSomethingStop14, doingSomethingStop15
};

//////////////////////////////////////////////////////

#define SIMPLE_TIMER_MS        2000L

// Init SimpleTimer
SimpleTimer simpleTimer;

// Here is software Timer, you can do somewhat fancy stuffs without many issues.
// But always avoid
// 1. Long delay() it just doing nothing and pain-without-gain wasting CPU power.Plan and design your code / strategy ahead
// 2. Very long "do", "while", "for" loops without predetermined exit time.
void simpleTimerDoingSomething2s()
{
  static unsigned long previousMicrosStart = startMicros;

  unsigned long currMicros = micros();

  Serial.print(F("SimpleTimer (ms): ")); Serial.print(SIMPLE_TIMER_MS);
  Serial.print(F(", us : ")); Serial.print(currMicros);
  Serial.print(F(", Dus : ")); Serial.println(currMicros - previousMicrosStart);

  for (uint16_t i = 0; i < NUMBER_ISR_PWMS; i++)
  {
    Serial.print(F("PWM Channel : ")); Serial.print(i);
    
  #if USING_PWM_FREQUENCY
    Serial.print(1000000 / PWM_Freq[i]);
  #else
    Serial.print(PWM_Period[i]);
  #endif
  
    Serial.print(F(", programmed Period (us): ")); Serial.print(PWM_Period[i]);
    Serial.print(F(", actual : ")); Serial.print(deltaMicrosStart[i]);

    Serial.print(F(", programmed DutyCycle : ")); 
  
    Serial.print(PWM_DutyCycle[i]);
      
    Serial.print(F(", actual : ")); Serial.println( (float) deltaMicrosStop[i] * 100.0f / deltaMicrosStart[i]);
  }

  previousMicrosStart = currMicros;
}

void setup()
{
  Serial.begin(115200);
  while (!Serial);

  delay(2000);

  Serial.print(F("\nStarting ISR_16_PWMs_Array_Complex on ")); Serial.println(ARDUINO_BOARD);
  Serial.println(ESP32_PWM_VERSION);
  Serial.print(F("CPU Frequency = ")); Serial.print(F_CPU / 1000000); Serial.println(F(" MHz"));

  // Interval in microsecs
  if (ITimer.attachInterruptInterval(HW_TIMER_INTERVAL_US, TimerHandler))
  {
    startMicros = micros();
    Serial.print(F("Starting ITimer OK, micros() = ")); Serial.println(startMicros);
  }
  else
    Serial.println(F("Can't set ITimer. Select another freq. or timer"));

  startMicros = micros();

  // Just to demonstrate, don't use too many ISR Timers if not absolutely necessary
  // You can use up to 16 timer for each ISR_PWM
  for (uint16_t i = 0; i < NUMBER_ISR_PWMS; i++)
  {
    previousMicrosStart[i] = micros();
    
  #if USING_PWM_FREQUENCY
    // You can use this with PWM_Freq in Hz
    ISR_PWM.setPWM(PWM_Pin[i], PWM_Freq[i], PWM_DutyCycle[i], irqCallbackStartFunc[i], irqCallbackStopFunc[i]);
  #else
    // Or You can use this with PWM_Period in us
    ISR_PWM.setPWM_Period(PWM_Pin[i], PWM_Period[i], PWM_DutyCycle[i], irqCallbackStartFunc[i], irqCallbackStopFunc[i]);
  #endif 
  }

  // You need this timer for non-critical tasks. Avoid abusing ISR if not absolutely necessary.
  simpleTimer.setInterval(SIMPLE_TIMER_MS, simpleTimerDoingSomething2s);
}

#define BLOCKING_TIME_MS      10000L

void loop()
{
  // This unadvised blocking task is used to demonstrate the blocking effects onto the execution and accuracy to Software timer
  // You see the time elapse of ISR_PWM still accurate, whereas very unaccurate for Software Timer
  // The time elapse for 2000ms software timer now becomes 3000ms (BLOCKING_TIME_MS)
  // While that of ISR_PWM is still prefect.
  delay(BLOCKING_TIME_MS);

  // You need this Software timer for non-critical tasks. Avoid abusing ISR if not absolutely necessary
  // You don't need to and never call ISR_PWM.run() here in the loop(). It's already handled by ISR timer.
  simpleTimer.run();
}
```
---
---

### Debug Terminal Output Samples

### 1. ISR_16_PWMs_Array_Complex on ESP32_DEV

The following is the sample terminal output when running example [ISR_16_PWMs_Array_Complex](examples/ISR_16_PWMs_Array_Complex) to demonstrate the accuracy of ISR Hardware PWM-channels, **especially when system is very busy**.  The ISR PWM-channels is **running exactly according to corresponding programmed periods and duty-cycles**


```Starting ISR_16_PWMs_Array_Complex on ESP32_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2059700
Channel : 0	Period : 1000000.00		OnTime : 50000	Start_Time : 2059892
Channel : 1	Period : 500000.00		OnTime : 50000	Start_Time : 2070635
Channel : 2	Period : 333333.33		OnTime : 66666	Start_Time : 2071142
Channel : 3	Period : 250000.00		OnTime : 75000	Start_Time : 2081882
Channel : 4	Period : 200000.00		OnTime : 80000	Start_Time : 2082378
Channel : 5	Period : 166666.67		OnTime : 74999	Start_Time : 2093142
Channel : 6	Period : 142857.14		OnTime : 71428	Start_Time : 2093630
Channel : 7	Period : 125000.00		OnTime : 68750	Start_Time : 2104408
Channel : 8	Period : 111111.11		OnTime : 66666	Start_Time : 2115168
Channel : 9	Period : 100000.00		OnTime : 65000	Start_Time : 2115698
Channel : 10	Period : 66666.67		OnTime : 46666	Start_Time : 2126457
Channel : 11	Period : 50000.00		OnTime : 37500	Start_Time : 2127002
Channel : 12	Period : 40000.00		OnTime : 32000	Start_Time : 2137818
Channel : 13	Period : 33333.33		OnTime : 28333	Start_Time : 2138406
Channel : 14	Period : 25000.00		OnTime : 22500	Start_Time : 2149243
Channel : 15	Period : 20000.00		OnTime : 19000	Start_Time : 2149824
SimpleTimer (ms): 2000, us : 12159983, Dus : 10100150
PWM Channel : 01000000.00, programmed Period (us): 1000000.00, actual : 1000000, programmed DutyCycle : 5.00, actual : 5.00
PWM Channel : 1500000.00, programmed Period (us): 500000.00, actual : 500000, programmed DutyCycle : 10.00, actual : 10.00
PWM Channel : 2333333.33, programmed Period (us): 333333.33, actual : 333340, programmed DutyCycle : 20.00, actual : 20.00
PWM Channel : 3250000.00, programmed Period (us): 250000.00, actual : 249999, programmed DutyCycle : 30.00, actual : 30.00
PWM Channel : 4200000.00, programmed Period (us): 200000.00, actual : 199999, programmed DutyCycle : 40.00, actual : 39.99
PWM Channel : 5166666.67, programmed Period (us): 166666.67, actual : 166679, programmed DutyCycle : 45.00, actual : 44.99
PWM Channel : 6142857.14, programmed Period (us): 142857.14, actual : 142859, programmed DutyCycle : 50.00, actual : 50.00
PWM Channel : 7125000.00, programmed Period (us): 125000.00, actual : 125005, programmed DutyCycle : 55.00, actual : 54.99
PWM Channel : 8111111.11, programmed Period (us): 111111.11, actual : 111124, programmed DutyCycle : 60.00, actual : 59.99
PWM Channel : 9100000.00, programmed Period (us): 100000.00, actual : 100005, programmed DutyCycle : 65.00, actual : 64.98
PWM Channel : 1066666.67, programmed Period (us): 66666.67, actual : 66679, programmed DutyCycle : 70.00, actual : 69.98
PWM Channel : 1150000.00, programmed Period (us): 50000.00, actual : 49999, programmed DutyCycle : 75.00, actual : 75.00
PWM Channel : 1240000.00, programmed Period (us): 40000.00, actual : 39999, programmed DutyCycle : 80.00, actual : 80.00
PWM Channel : 1333333.33, programmed Period (us): 33333.33, actual : 33340, programmed DutyCycle : 85.00, actual : 84.95
PWM Channel : 1425000.00, programmed Period (us): 25000.00, actual : 25001, programmed DutyCycle : 90.00, actual : 90.00
PWM Channel : 1520000.00, programmed Period (us): 20000.00, actual : 20000, programmed DutyCycle : 95.00, actual : 95.00
SimpleTimer (ms): 2000, us : 22327864, Dus : 10167881
PWM Channel : 01000000.00, programmed Period (us): 1000000.00, actual : 1000000, programmed DutyCycle : 5.00, actual : 5.00
PWM Channel : 1500000.00, programmed Period (us): 500000.00, actual : 500000, programmed DutyCycle : 10.00, actual : 10.00
PWM Channel : 2333333.33, programmed Period (us): 333333.33, actual : 333340, programmed DutyCycle : 20.00, actual : 20.00
PWM Channel : 3250000.00, programmed Period (us): 250000.00, actual : 250001, programmed DutyCycle : 30.00, actual : 30.00
PWM Channel : 4200000.00, programmed Period (us): 200000.00, actual : 200001, programmed DutyCycle : 40.00, actual : 40.00
PWM Channel : 5166666.67, programmed Period (us): 166666.67, actual : 166682, programmed DutyCycle : 45.00, actual : 44.98
PWM Channel : 6142857.14, programmed Period (us): 142857.14, actual : 142860, programmed DutyCycle : 50.00, actual : 49.99
PWM Channel : 7125000.00, programmed Period (us): 125000.00, actual : 124999, programmed DutyCycle : 55.00, actual : 54.99
PWM Channel : 8111111.11, programmed Period (us): 111111.11, actual : 111120, programmed DutyCycle : 60.00, actual : 59.99
PWM Channel : 9100000.00, programmed Period (us): 100000.00, actual : 100000, programmed DutyCycle : 65.00, actual : 65.00
PWM Channel : 1066666.67, programmed Period (us): 66666.67, actual : 66680, programmed DutyCycle : 70.00, actual : 69.98
PWM Channel : 1150000.00, programmed Period (us): 50000.00, actual : 50000, programmed DutyCycle : 75.00, actual : 75.00
PWM Channel : 1240000.00, programmed Period (us): 40000.00, actual : 40002, programmed DutyCycle : 80.00, actual : 80.00
PWM Channel : 1333333.33, programmed Period (us): 33333.33, actual : 33339, programmed DutyCycle : 85.00, actual : 84.94
PWM Channel : 1425000.00, programmed Period (us): 25000.00, actual : 25000, programmed DutyCycle : 90.00, actual : 90.00
PWM Channel : 1520000.00, programmed Period (us): 20000.00, actual : 20020, programmed DutyCycle : 95.00, actual : 94.91
```

---

### 2. ISR_16_PWMs_Array on ESP32_DEV

The following is the sample terminal output when running example [ISR_16_PWMs_Array](examples/ISR_16_PWMs_Array) to demonstrate how to use multiple Hardware PWM channels.

```
Starting ISR_16_PWMs_Array on ESP32_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2059691
Channel : 0	Period : 1000000.00		OnTime : 50000	Start_Time : 2059913
Channel : 1	Period : 500000.00		OnTime : 50000	Start_Time : 2070693
Channel : 2	Period : 333333.33		OnTime : 66666	Start_Time : 2071194
Channel : 3	Period : 250000.00		OnTime : 75000	Start_Time : 2081914
Channel : 4	Period : 200000.00		OnTime : 80000	Start_Time : 2082412
Channel : 5	Period : 166666.67		OnTime : 75000	Start_Time : 2093199
Channel : 6	Period : 142857.14		OnTime : 71428	Start_Time : 2093710
Channel : 7	Period : 125000.00		OnTime : 68750	Start_Time : 2104434
Channel : 8	Period : 111111.11		OnTime : 66666	Start_Time : 2104954
Channel : 9	Period : 100000.00		OnTime : 65000	Start_Time : 2115730
Channel : 10	Period : 66666.67		OnTime : 46666	Start_Time : 2126471
Channel : 11	Period : 50000.00		OnTime : 37500	Start_Time : 2127049
Channel : 12	Period : 40000.00		OnTime : 32000	Start_Time : 2137787
Channel : 13	Period : 33333.33		OnTime : 28333	Start_Time : 2138390
Channel : 14	Period : 25000.00		OnTime : 22500	Start_Time : 2149228
Channel : 15	Period : 20000.00		OnTime : 19000	Start_Time : 2149811
```

---


### 3. ISR_16_PWMs_Array_Simple on ESP32_DEV

The following is the sample terminal output when running example [ISR_16_PWMs_Array_Simple](examples/ISR_16_PWMs_Array_Simple) to demonstrate how to use multiple Hardware PWM channels.

```Starting ISR_16_PWMs_Array_Simple on ESP32_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2059738
Channel : 0	Period : 1000000.00		OnTime : 50000	Start_Time : 2059964
Channel : 1	Period : 500000.00		OnTime : 50000	Start_Time : 2070731
Channel : 2	Period : 333333.33		OnTime : 66666	Start_Time : 2071237
Channel : 3	Period : 250000.00		OnTime : 75000	Start_Time : 2081939
Channel : 4	Period : 200000.00		OnTime : 80000	Start_Time : 2082423
Channel : 5	Period : 166666.67		OnTime : 75000	Start_Time : 2093138
Channel : 6	Period : 142857.14		OnTime : 71428	Start_Time : 2093640
Channel : 7	Period : 125000.00		OnTime : 68750	Start_Time : 2104422
Channel : 8	Period : 111111.11		OnTime : 66666	Start_Time : 2104942
Channel : 9	Period : 100000.00		OnTime : 65000	Start_Time : 2115738
Channel : 10	Period : 66666.67		OnTime : 46666	Start_Time : 2126500
Channel : 11	Period : 50000.00		OnTime : 37500	Start_Time : 2127060
Channel : 12	Period : 40000.00		OnTime : 32000	Start_Time : 2137840
Channel : 13	Period : 33333.33		OnTime : 28333	Start_Time : 2138424
Channel : 14	Period : 25000.00		OnTime : 22500	Start_Time : 2149216
Channel : 15	Period : 20000.00		OnTime : 19000	Start_Time : 2149798
```

---

### 4. ISR_Modify_PWM on ESP32_DEV

The following is the sample terminal output when running example [ISR_Modify_PWM](examples/ISR_Modify_PWM) on **ESP32_DEV** to demonstrate how to modify PWM settings on-the-fly without deleting the PWM channel

```
Starting ISR_Modify_PWM on ESP32_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2059726
Using PWM Freq = 1.00, PWM DutyCycle = 10.00
Channel : 0	    Period : 1000000.00		OnTime : 100000	Start_Time : 2060379
Channel : 0	New Period : 500000.00		OnTime : 450000.00	Start_Time : 12060409
Channel : 0	New Period : 1000000.00		OnTime : 100000.00	Start_Time : 22060409
Channel : 0	New Period : 500000.00		OnTime : 450000.00	Start_Time : 31560409
Channel : 0	New Period : 1000000.00		OnTime : 100000.00	Start_Time : 42060409
```

---

### 5. ISR_Changing_PWM on ESP32_DEV

The following is the sample terminal output when running example [ISR_Changing_PWM](examples/ISR_Changing_PWM) on **ESP32_DEV** to demonstrate how to modify PWM settings on-the-fly by deleting the PWM channel and reinit the PWM channel

```
Starting ISR_Changing_PWM on ESP32_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2059714
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 2060368
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 12071352
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 22071379
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 32071338
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 42071356
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 52071338
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 62071356
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 72071338
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 82071356
```

---

### 6. ISR_Modify_PWM on ESP32S2_DEV

The following is the sample terminal output when running example [ISR_Modify_PWM](examples/ISR_Modify_PWM) on **ESP32S2_DEV** to demonstrate how to modify PWM settings on-the-fly without deleting the PWM channel

```
Starting ISR_Modify_PWM on ESP32S2_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_S2_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2559974
Using PWM Freq = 1.00, PWM DutyCycle = 10.00
Channel : 0	    Period : 1000000.00		OnTime : 100000	Start_Time : 2570620
Channel : 0	New Period : 500000.00		OnTime : 450000.00	Start_Time : 12570649
Channel : 0	New Period : 1000000.00		OnTime : 100000.00	Start_Time : 22570649
Channel : 0	New Period : 500000.00		OnTime : 450000.00	Start_Time : 32070689
Channel : 0	New Period : 1000000.00		OnTime : 100000.00	Start_Time : 42570749
```

---

### 7. ISR_Changing_PWM on ESP32S2_DEV

The following is the sample terminal output when running example [ISR_Changing_PWM](examples/ISR_Changing_PWM) on **ESP32S2_DEV** to demonstrate how to modify PWM settings on-the-fly by deleting the PWM channel and reinit the PWM channel

```
Starting ISR_Changing_PWM on ESP32S2_DEV
ESP32_PWM v1.2.1
CPU Frequency = 240 MHz
[PWM] ESP32_S2_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 1 , _timerGroup = 0
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2561799
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 2566886
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 12576879
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 22581846
```

---

### 8. ISR_Modify_PWM on ESP32C3_DEV

The following is the sample terminal output when running example [ISR_Modify_PWM](examples/ISR_Modify_PWM) on **ESP32S2_DEV** to demonstrate how to modify PWM settings on-the-fly without deleting the PWM channel

```
Starting ISR_Modify_PWM on ESP32C3_DEV
ESP32_PWM v1.2.1
CPU Frequency = 160 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 0 , _timerGroup = 1
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2100385
Using PWM Freq = 1.00, PWM DutyCycle = 10.00
Channel : 0	    Period : 1000000.00		OnTime : 100000	Start_Time : 2113126
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 12118037
Channel : 0	    Period : 1000000.00		OnTime : 100000	Start_Time : 22119020
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 32120019
Channel : 0	    Period : 1000000.00		OnTime : 100000	Start_Time : 42121019
```

---

### 9. ISR_Changing_PWM on ESP32C3_DEV

The following is the sample terminal output when running example [ISR_Changing_PWM](examples/ISR_Changing_PWM) on **ESP32S2_DEV** to demonstrate how to modify PWM settings on-the-fly by deleting the PWM channel and reinit the PWM channel

```
Starting ISR_Changing_PWM on ESP32C3_DEV
ESP32_PWM v1.2.1
CPU Frequency = 160 MHz
[PWM] ESP32_TimerInterrupt: _timerNo = 1 , _fre = 1000000
[PWM] TIMER_BASE_CLK = 80000000 , TIMER_DIVIDER = 80
[PWM] _timerIndex = 0 , _timerGroup = 1
[PWM] _count = 0 - 20
[PWM] timer_set_alarm_value = 20.00
Starting ITimer OK, micros() = 2100339
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 2105212
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 12117109
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 22122103
Using PWM Freq = 2.00, PWM DutyCycle = 90.00
Channel : 0	    Period : 500000.00		OnTime : 450000	Start_Time : 32122107
Using PWM Freq = 1.00, PWM DutyCycle = 50.00
Channel : 0	    Period : 1000000.00		OnTime : 500000	Start_Time : 42127102
```


---
---

### Debug

Debug is enabled by default on Serial.

You can also change the debugging level `_PWM_LOGLEVEL_` from 0 to 4

```cpp
// These define's must be placed at the beginning before #include "ESP32_PWM.h"
// _PWM_LOGLEVEL_ from 0 to 4
// Don't define _PWM_LOGLEVEL_ > 0. Only for special ISR debugging only. Can hang the system.
#define _PWM_LOGLEVEL_      4
```

---

### Troubleshooting

If you get compilation errors, more often than not, you may need to install a newer version of the core for Arduino boards.

Sometimes, the library will only work if you update the board core to the latest version because I am using newly added functions.


---
---

### Issues

Submit issues to: [ESP32_PWM issues](https://github.com/khoih-prog/ESP32_PWM/issues)

---

## TO DO

1. Search for bug and improvement.
2. Similar features for remaining Arduino boards such as SAMD21, SAMD51, SAM-DUE, nRF52, ESP8266, STM32, Portenta_H7, RP2040, etc.



## DONE

 1. Basic hardware PWM-channels for ESP32, ESP32_C2 and ESP32_C3 for [ESP32 core v2.0.0+](https://github.com/espressif/arduino-esp32/releases/tag/2.0.0)
 2. Longer time interval
 3. Add complex examples.
 4. Add functions to modify PWM settings on-the-fly
 5. Fix examples to use with ESP32 core v2.0.1+
 6. Fix `multiple-definitions` linker error. Drop `src_cpp` and `src_h` directories
 7. Add example [multiFileProject](examples/multiFileProject) to demo for multiple-file project
 8. Improve accuracy by using `double`, instead of `uint32_t` for `dutycycle`, `period`. Check [Change Duty Cycle #1](https://github.com/khoih-prog/ESP8266_PWM/issues/1#issuecomment-1024969658)
 9. DutyCycle to be optionally updated at the end current PWM period instead of immediately. Check [DutyCycle to be updated at the end current PWM period #2](https://github.com/khoih-prog/ESP8266_PWM/issues/2)

---
---

### Contributions and Thanks

Many thanks for everyone for bug reporting, new feature suggesting, testing and contributing to the development of this library.


---

## Contributing

If you want to contribute to this project:
- Report bugs and errors
- Ask for enhancements
- Create issues and pull requests
- Tell other people about this library

---

### License

- The library is licensed under [MIT](https://github.com/khoih-prog/ESP32_PWM/blob/master/LICENSE)

---

## Copyright

Copyright 2021- Khoi Hoang


