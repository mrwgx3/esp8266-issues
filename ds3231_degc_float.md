
Hi Makuna

Enjoyed studying your RtcDS3231 library; I found your use of classes and templates most instructive. While studying the temperature processing code, however, I became most puzzled on how raw temperature was converted to a float. A side-by-side comparison of the D3231 datasheet and your code lead me to realize that the rather ambiguous description of the temperature register function admits (2) plausible interpretations:

```
Temperature Registers (11h–12h)
Temperature is represented as a 10-bit code with a resolution of 0.25°C and is accessible at
location 11h and 12h. The temperature is encoded in two’s complement format. The upper 8 bits,
the integer portion, are at location 11h and the lower 2 bits, the fractional portion, are
in the upper nibble at location 12h. For example, 00011001 01b = +25.25°C.
```
The 1st interpretation is that the upper and lower registers reflect a 16-bit wide 2's complement number, which reflects the temperature scaled by a factor of 256.

The 2nd case would be that the upper byte is an 8-bit 2's complement number representing a signed integer temperature of range -128 to 127, and the lower byte is an 8-bit unsigned integer holding a scaled fractional portion, 256 * { 0.00, 0.25, 0.50, 0.75 } = {0x00, 0x40, 0x80, 0xC0 }.

The bit diagrams accompanying the register description support the former, and the example within the description, the latter (too bad the example wasn't signed).

In order the resolve the issue, both interpretations were coded, and a dusty can of freeze spray was dug out of the closet. After cooling the RTC to some negative temperature, the correct interptation should show a monotonically increasing sequence of numbers as the part warms up. The 1st interpretation, the scaled 16-bit 2's complement temperature, won the day. The test code which generated the table below is included at the end of this post.

```
Column     Interpretion / Description                      Function
------------------------------------------------------------------------------
  A        original library code                           AsFloat()
  B        test code,  8-bit int/uint interpretation       AsFloatTestA()
  C        test code, 16-bit 2's complement word           AsFloatTestB()
  D        tweaked library code, added after initial test  AsFloat(), modified 

-12 degC  through -10 degC
--------------------------
   A        B           C        D
-12.00   -12.00      -12.00   -12.00    degC
-12.25   -12.25      -11.75   -11.75    degC
-12.50   -12.50      -11.50   -11.50    degC
-12.75   -12.75      -11.25   -11.25    degC
-11.00   -11.00      -11.00   -11.00    degC
-11.25   -11.25      -10.75   -10.75    degC
-11.50   -11.50      -10.50   -10.50    degC
-11.75   -11.75      -10.25   -10.25    degC
-10.00   -10.00      -10.00   -10.00    degC

Transition through 0 degC
-------------------------
  A       B          C        D
-1.00   -1.00      -1.00   -1.00    degC
-1.25   -1.25      -0.75   -0.75    degC
-1.50   -1.50      -0.50   -0.50    degC
-1.75   -1.75      -0.25   -0.25    degC

0.00   0.00      0.00   0.00    degC      Above 0 degC, all cases the same
0.25   0.25      0.25   0.25    degC
0.50   0.50      0.50   0.50    degC
0.75   0.75      0.75   0.75    degC
1.00   1.00      1.00   1.00    degC
```

Interestingly enough, the library code only requires a minor tweak to function correctly:
```
// Corrected 'AsFloat()' routine
   float AsFloatTestC ( RtcTemperature& rtmp )
   {
      float degrees = (float)rtmp.AsWholeDegrees();;
      degrees += (float)rtmp.GetFractional() / 100.0f ;
      return( degrees );
   
   } //AsFloatTestC

versus

   float AsFloat()
   {
      float degrees = (float)integerDegrees;
      degrees += (float)decimalFraction / ((degrees < 0) ? -100.0f : 100.0f) ;
      return degrees;
   }
```

It would probably make more sense, however, to change [GetTemperature()][get_temp] to return raw register values and rewrite [class RtcTemperature][class_rtctemp] to support those changes.

In closing, Lakuna, thanks again for your fine work!

[get_temp]: https://github.com/Makuna/Rtc/blob/master/src/RtcDS3231.h#L319#L332  "GetTemperature() in class RtcDS3231"
[class_rtctemp]: https://github.com/Makuna/Rtc/blob/master/src/RtcTemperature.h#L7#L36  "class RtcTemperature"

### Test Sketch
```
// CONNECTIONS:
// DS3231 SDA --> SDA
// DS3231 SCL --> SCL
// DS3231 VCC --> 3.3v or 5v
// DS3231 GND --> GND

#if defined(ESP8266)
#include <pgmspace.h>
#else
#include <avr/pgmspace.h>
#endif

/* for software wire use below
#include <SoftwareWire.h>  // must be included here so that Arduino library object file references work
#include <RtcDS3231.h>

SoftwareWire myWire(SDA, SCL);
RtcDS3231<SoftwareWire> Rtc(myWire);
 for software wire use above */

/* for normal hardware wire use below */
#include <Wire.h> // must be included here so that Arduino library object file references work
#include <RtcDS3231.h>
RtcDS3231<TwoWire> Rtc(Wire);
/* for normal hardware wire use above */

void setup () 
{
  Serial.begin(115200);
  while (!Serial) delay(250);      // Wait until Arduino Serial Monitor opens
  Serial.println( " " );
  Serial.println(F("DS3231_Simple Temperature Test"));

    Serial.print("compiled: ");
    Serial.print(__DATE__);
    Serial.println(__TIME__);

    //--------RTC SETUP ------------
    Rtc.Begin();

    // if you are using ESP-01 then uncomment the line below to reset the pins to
    // the available pins for SDA, SCL
    // Wire.begin(0, 2); // due to limited pins, use pin 0 and 2 for SDA, SCL

    RtcDateTime compiled = RtcDateTime(__DATE__, __TIME__);
    printDateTime(compiled);
    Serial.println();

    if (!Rtc.IsDateTimeValid()) 
    {
        // Common Cuases:
        //    1) first time you ran and the device wasn't running yet
        //    2) the battery on the device is low or even missing

        Serial.println("RTC lost confidence in the DateTime!");

        // following line sets the RTC to the date & time this sketch was compiled
        // it will also reset the valid flag internally unless the Rtc device is
        // having an issue

        Rtc.SetDateTime(compiled);
    }

    if (!Rtc.GetIsRunning())
    {
        Serial.println("RTC was not actively running, starting now");
        Rtc.SetIsRunning(true);
    }

    RtcDateTime now = Rtc.GetDateTime();
    if (now < compiled) 
    {
        Serial.println("RTC is older than compile time!  (Updating DateTime)");
        Rtc.SetDateTime(compiled);
    }
    else if (now > compiled) 
    {
        Serial.println("RTC is newer than compile time. (this is expected)");
    }
    else if (now == compiled) 
    {
        Serial.println("RTC is the same as compile time! (not expected but all is fine)");
    }

    // never assume the Rtc was last configured by you, so
    // just clear them to your needed state
    Rtc.Enable32kHzPin(false);
    Rtc.SetSquareWavePin(DS3231SquareWavePin_ModeNone); 
} //setup

// Temperature test loop
void loop () 
{
    if (!Rtc.IsDateTimeValid()) 
    {
        // Common Cuases:
        //    1) the battery on the device is low or even missing and the power line was disconnected
        Serial.println("RTC lost confidence in the DateTime!");
    }

//  Force a temperature A/D read
    Rtc.ForceTemperatureCompensationUpdate( true );

//  Get the temperature data    
    RtcTemperature temp = Rtc.GetTemperature();

    Serial.print (temp.AsFloat() );         // Column A: Original library code
    Serial.print( "   " );

    Serial.print( AsFloatTestA( temp ) );   // Column B: Int/frac Temp. calc.       
    Serial.print( "      " );

    Serial.print( AsFloatTestB( temp ) );   // Column C: 2's comp. temp. calc.
    Serial.print( "   " );
    
    Serial.print( AsFloatTestC( temp ) );   // Column D: Corrected original code
    Serial.print( "    " );
    
    Serial.print( "degC" );
    Serial.println();

    delay(50);
        
} //loop

#define countof(a) (sizeof(a) / sizeof(a[0]))

void printDateTime(const RtcDateTime& dt)
{
    char datestring[20];

    snprintf_P(datestring, 
            countof(datestring),
            PSTR("%02u/%02u/%04u %02u:%02u:%02u"),
            dt.Month(),
            dt.Day(),
            dt.Year(),
            dt.Hour(),
            dt.Minute(),
            dt.Second() );
    Serial.print(datestring);
} //printDateTime

// Calculate temperature as if:
//
//  a) The upper temperature byte represents a signed single-byte integer
//     of range -128 to +127
//  b) The lower temperature byte represents an un-signed fractional part
//     representing  0, 1/4, 1/2 and 3/4 degrees.
//  
//     The number of right-shifts required to register the lower byte with
//     the upper one is 8, corresponding to a scale factor of 256.
//
//    bit:  7  6  5  4  3  2  1  0 -1 -2      7  6  5  4  3  2  1  0 -1 -2
//          b  b  0  0  0  0  0  0  0  0  ->  0  0  0  0  0  0  0  0  b  b
//
//     It easier to right-shift the upper-byte 8 times and leave the lower-byte as is.
//
float AsFloatTestA ( RtcTemperature& rtmp )
{
// Retrieve 'signed' integer temp. value
   int8_t   deg = rtmp.AsWholeDegrees();

// Reverse libary scaling to recover 'unsigned' fractional temp. value   
   uint8_t frac = ( (rtmp.GetFractional() / 25) << 6 );

// Scale integer portion to match fractional term  
   int16_t  sgnT = ( (uint8_t)deg << 8 );

// Take absolute value of integer degC & add 'unsigned' fractional term
   bool  negT = false;
   if ( sgnT < 0 )
      { sgnT = -sgnT; negT = true; }
   sgnT += frac;

// Restore sign of combined result, unscale and return as float
   if ( negT ) sgnT = -sgnT;
   return( (float)sgnT / 256.0 );

} //AsFloatTestA


// Calculate temperature as if...
//
// The upper and lower bytes represent the MS and LS bytes of a
// 16-bit, 2's complement WORD.
//
// The number of right-shifts needed to scale this signed 16-bit integer
// to float degrees is 8, or 256
//
//  15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00 -1 -2
//   S  d  d  d  d  d  d  d  d  d  0  0  0  0  0  0  0  0   before shift
//   S  S  S  S  S  S  S  S  S  d  d  d  d  d  d  d  d  d   after  shift
//
//  It easier to right-shift the upper-byte 8 times and leave the lower-byte as is.
//
float AsFloatTestB ( RtcTemperature& rtmp )
{
// Retrieve MS portion of 2's comp. temperature
   int8_t   msbyte = rtmp.AsWholeDegrees();

// Reverse libary scaling to recover LS portion
   uint8_t lsbyte = ( (rtmp.GetFractional() / 25) << 6 );

   int16_t  itemp  = ( (uint8_t)msbyte << 8 ); // Scale upper byte, now 16-bit integer
            itemp += lsbyte;                   // Concant the lower byte
   return( (float)itemp / 256.0 );             // Un-scale and return
   
} //AsFloatTestB

// Corrected 'AsFloat()' routine
float AsFloatTestC ( RtcTemperature& rtmp )
{
   float degrees = (float)rtmp.AsWholeDegrees();;
   degrees += (float)rtmp.GetFractional() / 100.0f ;
   return( degrees );
   
} //AsFloatTestC

```
