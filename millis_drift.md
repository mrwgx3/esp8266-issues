### Basic Infos

#### Hardware

```
Hardware:     ESP12, Adafruit Huzzah Breakout and Feather Boards  
Core Version: 2.3.0
```

#### OS, IDE, and SDK

``` 
OS:   Windows 10 Professional and Linux Ubuntu 16.04LTS   
IDE:  1.6.12  
SDK:  1.5.3_16_04_18
```

### Description

While studying how millis()[millis_esp] and micros()[micros_esp] work in order to extending them to 64-bits, I noticed an approximation[millis_approx] in the millis() function which causes the returned millisecond value to incrementally drift 296us / ~71 minutes (2^32 us) of operation. This drift is cumlative, i.e., does not reset upon roll-over of millis(). The sketch included at the end of this post demonstrates the drift, and gives some corrections.

[millis_esp]: https://github.com/esp8266/Arduino/blob/master/cores/esp8266/core_esp8266_wiring.c#L64#L68  "Millis() Function"
[micros_esp]: https://github.com/esp8266/Arduino/blob/master/cores/esp8266/core_esp8266_wiring.c#L70#L72  "Micros() function"
[millis_approx]: https://github.com/esp8266/Arduino/blob/master/cores/esp8266/core_esp8266_wiring.c#L67#L67 "Approximation with truncated constant"

### Settings in IDE

```
Module:          Adafruit HUZZAH ESP8266  
Flash Size:      4MB (3M SPIFFS)  
CPU Frequency:   80Mhz  
Debug Settings:  None presented by IDE  
```

```
Module:          Generic ESP8266 Module  
Flash Size:      4MB (3M SPIFFS)  
CPU Frequency:   80Mhz  
Flash Mode:      dio  
Flash Frequency: 40Mhz  
Upload Using:    Disabled or SERIAL  
Reset Method:    nodemcu  
Debug Port:      None
Debug Level:     None
```

### Sketch Output

```
Millis() Drift Correction

1st Millis() overflow at ~49 days,  us overflow count = 1000
us_ovflow err: 296.00  ms
     Original     Corrected        Altn    Corr - Altn    Orig - Corr
      mills()      millis()     millis()    Difference     Difference     Heap
          88           384           384             0        -296       49328
        1095          1391          1391             0        -296       49352
        2095          2391          2391             0        -296       49352

At (1) year,  us overflow count = 7342
us_ovflow err: 2173.23  ms
     Original     Corrected        Altn    Corr - Altn    Orig - Corr
      mills()      millis()     millis()    Difference     Difference     Heap
  1468880042    1468882215    1468882215             0       -2173       49352
  1468881049    1468883222    1468883222             0       -2173       49352
  1468882049    1468884222    1468884222             0       -2173       49352

*** End ***
```

### Sketch

```cpp
//
// Millis() Drift Example
// This code illustrates how the ESP8266 implementation of Millis()
// incrementally drifts 296us / ~71 minutes (2^32 us) of operation.
// This drift is cumlative, i.e., does not reset upon roll-over of
// millis()
//

#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <stdio.h>

// Include API-Headers
extern "C" {                  // SDK functions for Arduino IDE access
#include "osapi.h"
#include "user_interface.h"
#include "espconn.h"
}  //end, 'extern C'

//---------------------------------------------------------------------------
// Globals

const int  LedBlu = 2, LedRed = 0, LedOff = 1, LedOn = 0;

// For 64-bit usec ring counter
static os_timer_t  us_ovf_timer;
static uint32_t    us_cnt = 0;
static uint32_t    us_ovflow = 0;
static uint32_t    us_at_last_ovf = 0;

//---------------------------------------------------------------------------
// Interrupt code lifted directly from "cores/core_esp8266_wiring.c",
// with some variable renaming
//---------------------------------------------------------------------------

// Callback for usec counter overflow timer interrupt
void  us_overflow_tick ( void* arg )
{
  us_cnt = system_get_time();

  // Check for usec counter overflow
  if ( us_cnt < us_at_last_ovf ) {
    ++us_cnt;
  } //end-if
  us_at_last_ovf = us_cnt;

} //us_overflow_tick

//---------------------------------------------------------------------------
#define  REPEAT 1
void us_count_init ( void )
{
  os_timer_setfn( &us_ovf_timer, (os_timer_func_t*)&us_overflow_tick, 0 );
  os_timer_arm( &us_ovf_timer, 60000, REPEAT );

} //us_count_init

//---------------------------------------------------------------------------
// Original millis() function
unsigned long ICACHE_RAM_ATTR millis_orig ( void )
{
  uint32_t  m  = system_get_time();
  uint32_t  c  = us_ovflow + ((m < us_at_last_ovf) ? 1 : 0);
  return ( c * 4294967 + m / 1000 );

// Above equation appears to be an approximation for
//
// (c * 4294967296 + m) / 1000  ~= c * 4294967 + m / 1000
//
// Does this simplify the compilers work, i.e., is the multiply
// 'c * 4294967' less complex than 'c * 4294967296'. Both would
// seem to require a 64-bit result, although the former could 
// discard the most significant 32 bits.

} //millis_orig

//---------------------------------------------------------------------------
// Corrected millis(), truncated 64-bit arithmetic
unsigned long ICACHE_RAM_ATTR millis_corr( void )
{
  uint32_t m = system_get_time();
  uint32_t c = us_ovflow + ((m < us_at_last_ovf) ? 1 : 0);
  return ( (c * 4294967296 + m) / 1000 );

} //millis_corr

//---------------------------------------------------------------------------
//  Alternate form, corrected millis()
//
//  Convert 64-bit form into 32-bit arithmetic, assuming that the compiler
//  is smart enough to return the least-significant 32-bit word for the
//  64-bit multiply, 'c * 4294967'
//
//  ( c * 4294967296 +  m ) / 1000
//  c * ( 429496700 + 296 ) / 1000  + (m / 1000)
//  c * 4294967 + ( m + c*296 ) / 1000
//
//  noting  c =  65536 * chi + clo,  where chi & clo are uint16_t quantities...
//
//  c*4294967 +            ( m + (65536 * chi + clo) * 296 )/1000
//    .....   + (m/1000) + ( clo*296 )/1000  +  chi*(65536*296/1000)
//    .....   + (m/1000) + ( clo*296 )/1000  +  chi*(19398 + 0.656)
//    .....   + (m/1000) + ( clo*296 )/1000  + (chi*1000*0.656)/1000 + chi*19398
//    .....   + (m/1000) + ( clo*296 )/1000  + (chi*656)/1000        + chi*19398
//    .....   + (m/1000) + ( clo*296 + chi*656 )/1000 + chi*19398
//
//   Original millis() form                 Additional terms
//
//  Truncated
//  64-bit?      32-bit              32-bit                 32-bit
//  c*4294967 + (m/1000)  +  ((clo*296 + chi*656 )/1000) + chi*19398
//
unsigned long ICACHE_RAM_ATTR millis_altn ( void )
{
  uint32_t m = system_get_time();
  uint32_t c = us_ovflow + ((m < us_at_last_ovf) ? 1 : 0);

  // High/low words of 'c'
  uint16_t chi = c >> 16, clo = c & 0xFFFF;

  return (
           (uint32_t)(c * 4294967) + (m / 1000) +
           ((clo * 296 + chi * 656) / 1000) + chi * 19398
         ); //end-return

} //millis_altn

//---------------------------------------------------------------------------
// Output routines
//---------------------------------------------------------------------------

void millis_header ( void )
{
  Serial.print( "us_ovflow err: " ); Serial.print( 0.296 * us_ovflow );  Serial.println( "  ms" );
  Serial.println( F("     Original     Corrected        Altn    Corr - Altn    Orig - Corr") );
  Serial.println( F("      mills()      millis()     millis()    Difference     Difference     Heap") );

} //millis_header

//---------------------------------------------------------------------------
void millis_compare ( void )
{
  uint32_t ms_orig  = millis_orig();
  uint32_t ms_corr  = millis_corr();
  uint32_t ms_altn  = millis_altn();
  uint16_t heap     = ESP.getFreeHeap();

  char buf[60] = "";
  sprintf( buf, "%12d  %12d  %12d",
           ms_orig,  ms_corr,  ms_altn
         ); //end-sprint
  Serial.print( buf );

  Serial.print( "    " );

  sprintf( buf, "%10d  %10d     %7d",
           (int32_t)(ms_corr - ms_altn),
           (int32_t)(ms_orig - ms_corr),
           heap
         ); //end-sprint
  Serial.println( buf );

} //millis_compare

//---------------------------------------------------------------------------
//---------------------------------------------------------------------------
void setup ()
{
  pinMode(LedRed, OUTPUT);         // Turn off LEDs
  pinMode(LedBlu, OUTPUT);         // ...
  digitalWrite(LedRed, LedOff);    // ...
  digitalWrite(LedBlu, LedOff);    // ...

  Serial.begin(115200);
  while (!Serial) delay(250);      // Wait until Arduino Serial Monitor opens
  Serial.println( " " );
  Serial.println(F("Millis() Drift Correction"));

  WiFi.mode( WIFI_OFF );

  // Interrupt for overflow detection, millis() function tests
  us_count_init();

  // Test A
  Serial.println();
  Serial.println( F("1st Millis() overflow at ~49 days,  us overflow count = 1000") );
  us_at_last_ovf = 0;
  us_ovflow = 1000;      // Error is 0.296 ms * overflow count = 296 ms

  millis_header();
  for ( int i = 0; i < 3; i++ ) {
    millis_compare();
    delay(1000);
  } //end-for

  // Test B
  Serial.println();
  Serial.println( F("At (1) year,  us overflow count = 7342") );
  us_at_last_ovf = 0;
  us_ovflow = 7342;      // Error is 0.296 ms * overflow count = 2173 ms

  millis_header();
  for ( int i = 0; i < 3; i++ ) {
    millis_compare();
    delay(1000);
  } //end-for

  Serial.println();
  Serial.println( "*** End ***" );

} //setup

//---------------------------------------------------------------------------
void loop(void)
{
  yield();
} //loop

//---------------------------------------------------------------------------
//---------------------------------------------------------------------------
```
