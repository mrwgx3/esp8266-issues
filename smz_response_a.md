Hi @smz, and thanks for you commentary.

First off, we must remember the nature of the bug is a mis-interpretation of the meaning of the RTC temperature register contents. Lets review GetTemperature() and portions of the original RtcTemperature class:

```
RtcTemperature RtcDS3231::GetTemperature( void )
{
   .....

   _wire.requestFrom(DS3231_ADDRESS, DS3231_REG_TEMP_SIZE);

   // Old interpretation:   Signed 8-bit integer / Unsigned 8-bit scaled fraction

-  int8_t degrees = _wire.read();                // 'degrees' is SIGNED, with no scaling
-  // fraction is just the upper bits
-  // representing 1/4 of a degree
-  uint8_t fract = (_wire.read() >> 6) * 25;     // 'fract' is UNSIGNED, with SF = (256 / 64) * 25 = 100
-  return RtcTemperature(degrees, fract);        // Original 'RtcTemperature' uses degrees/fract

} //GetTemperature

class RtcTemperature
{
public:

-    RtcTemperature(int8_t degrees, uint8_t fraction) :
-        integerDegrees(degrees),
-        decimalFraction(fraction)
     {
     }

     int8_t AsWholeDegrees()
     {
-        return integerDegrees;    // This is 'degrees' supplied by GetTemperature()
     }                             // Signed and un-scaled (x1)

     uint8_t GetFractional()
     {
-        return decimalFraction;   // This is 'fract' supplied by GetTemperature()
     }                             // Unsigned and scaled (x100)
 
protected:
-    int8_t  integerDegrees;
-    uint8_t decimalFraction;

}; //end, RtcTemperature

```
When GetTemperature() is called, it returns a RtcTemperature object containing a signed/un-scaled integer temperature and a unsigned/unscaled fractional temperature. When called, AsWholeDegrees() and GetFractional() each deliver their respective temperature components unchanged:
```
a) AsWholeDegrees() returns a signed single-byte integer temperature of range -128 to +127 degC

b) GetFractional()  returns an un-signed single-byte scaled fractional temperature representing
                    100 * {0, 1/4, 1/2 and 3/4 degC} = { 0, 25, 50, and 75}
```
It's important to note that GetTemperature() does the scaling for the fractional portion.

As presented, these values would be idea for printing a float representation of signed temperature, but without needed a float conversion. If the RTC did that, it would be pretty cool, but a little odd as well. This is what prompted me to go look carefully at the temperature register description in theDS3231 datasheet:

```
Temperature Registers (11h–12h)
Temperature is represented as a 10-bit code with a resolution of 0.25°C and is accessible at
location 11h and 12h. The temperature is encoded in two’s complement format. The upper 8 bits,
the integer portion, are at location 11h and the lower 2 bits, the fractional portion, are
in the upper nibble at location 12h. For example, 00011001 01b = +25.25°C.
```

There's nothing in the above wording which rules out an 8-bit signed integer / unsigned fraction register interpretation, as inferred by GetTemperature(). Conversely, the description wording doesn't rule out 16-bit, 2's complement interpretation either, which is also easily realizable in hardware. The inability to decide conclusively prompted me to do the experiment leading up issue (#49).

Now lets look at GetTemperature() and the RtcTemperature class for corrected code:
```
RtcTemperature RtcDS3231::GetTemperature( void )
{
   .....

   _wire.requestFrom(DS3231_ADDRESS, DS3231_REG_TEMP_SIZE);

   // Correct interpretation:  Signed 16-bit, 2's complement integer
+  int8_t  mstmp = _wire.read();                 // MS byte, scaled 2's comp. temperature
+  uint8_t lstmp = _wire.read();                 // LS byte, ............................
+  return RtcTemperature( mstmp, lstmp );        // New 'RtcTemperature' uses mstmp/lstmp 

} //GetTemperature

class RtcTemperature
{
public:

+    RtcTemperature( int8_t msdegC, uint8_t lsdegC ) :
+        mstemp( msdegC ),
+        lstemp( lsdegC & 0xC0 )  // Bind to RTC resolution (only upper (2) bits enter computation)
     {
     }

+    // Scaled integer temperature (x 256)
+    int16_t AsScaledDegrees()
+    {
+        return(  (mstemp << 8) + lstemp  );  // Concatentate raw temp. regs 
+    }

+    // Rounded integer temperature (x 1)
+    int8_t AsRoundedDegrees()
+    {
+        // Equivalent float function:
+        //   SF ( sign(x)    * floor(  abs(       x ) + 0.5    ) ) / SF, with
+        // =    ( sign(SF x) * floor(  abs(    SF x ) + 0.5 SF ) ) / SF  SF = 256
+        // =      sign(x)    * floor( sign(x)( SF x ) + 0.5 SF )   / SF
+        //
+        int8_t  sgnT = ( (mstemp < 0) ? -1 : 1 );
+        return( sgnT * ((sgnT * AsScaledDegrees() + 0x80) >> 8) );
+    }

     // Float temperature (x 1)
     float AsFloat()
     {
+        return ( (float)AsScaledDegrees() / 256.0f );
     }

+    // Integer portion, temperature (x 1), useful for printing
     int8_t AsWholeDegrees()
     {
+        int8_t  sgnT = ( (mstemp < 0) ? -1 : 1 );
+        return( sgnT * ((sgnT * AsScaledDegrees()) >> 8) ); // (x 1)
     }
 
+    // Fractional portion, temperature (x100), useful for printing
+    // Returns (0, 25, 50, or 75)
     uint8_t GetFractional()
     {
+        int8_t  sgnT = ( (mstemp < 0) ? -1 : 1 );
+        return(   25 * ((uint8_t)(sgnT * lstemp) >> 6)  );  // (x 100)
     }

protected:
-    int8_t  mstemp;
-    uint8_t lstemp;

}; //end, RtcTemperature
```

When GetTemperature() is called, it now returns a RtcTemperature object containing a 2's complement (i.e, signed) 16-bit integer; the same information content as the original, but in a different format.

Note also that the methods in RtcTemperature now do all the heavy lifting; GetTemperature() just passes the raw temperature registers along.

Now that we've reviewed code, it would be a good time for a numerical example. First off, lets define our primary variables as the ones returned by GetTemperature():
```
a) degrees / fract   for the original  version
b) mstemp  / lstemp  for the corrected version
```
```
Say my RTC is at a temperature of 23.25 degC, which scales to 256 x 23.25 = 5952, or 0x1740.

Here 'mstemp' and 'degrees' both equal 0x17 = 23, and
     'lstemp' = 0x40 = 64, and 'fract' = (lstemp / 64) * 25 = 25.  All well and good, both original and new

Now say my RTC is at a negative temperature of -23.25 degC, which scales to 256 * -23.25 = -5952, or 0xE8C0.

Here 'mstemp' and 'degrees' now equal 0xE8 = -24,                       which is not -23, and
     'lstemp' = 0xC0 = 192, and 'fract' becomes (lstmp / 64) * 25 = 75, which is not  25
```
As you can see, splitting apart a 16-bit, 2's complement integer breaks the original interpretation of 'signed unscaled / unsigned scale fractional' for negative temperatures. This is the bug quantified.

Now lets plug our numbers into the new methods of RtcTemperature:
```
AsScaledDegrees() recombines 'mstemp' and 'lstemp' back into a single integer:

returned integer equals:  (mstemp << 8) + lstemp
                          (  0xE8 << 8) + 0xC0
                               0xE800   + 0xC0
                          0xE8C0 = -5952, our orignal signed integer

Both AsWholeDegrees() and GetFractional()  now use interm
'sgnT' to extract the sign of the 2's complement integer:

sgnT = ( (mstemp < 0) ? -1 : 1 );
     = ( (-24 < 0) : -1 : 1 );
     = -1

AsWholeDegrees() returns:

   sgnT * ((sgnT * AsScaledDegrees()) >> 8) 
   (-1) * ((-1 * -5952) >> 8 )
   (-1) * ( 0x1740 >> 8 )
   (-1) * ( 0x17, or 23 )
   -23    No problems now

GetFractional() returns:

    25 * ((uint8_t)(sgnT * lstemp) >> 6)
    25 * ((uint8_t)( -1 * 0xC0 ) >> 6 )
    25 * ((uint8_t)( -1 * -64  ) >> 6 )
    25 * ((uint8_t)( 64 >> 6 )
    25 * ((uint8_t)( 1 )
    25   No problem here either; do you recognize the scaling used in the original GetTemperature()?
```
As you can see, both AsWholeDegrees() are GetFractional() now process negative temperatures correctly; they're fixed! They are of great use for printing, but not much use computationally-wise.

Concerning method AsFloat(), I hope you can see now that its exactly the same endian-independent function I tested for your library. As for method AsScaledDegrees(), it is integral to most the the methods in the new RtcTemperature. I decided to keep it public, as it:
```
a) Is generally useful, computationally-wise.
b) Can be used to reconstruct the original RTC temperature registers.
```

What is most useful, integer-wise, is a rounded version of temperature, which is provided by method AsRoundedDegrees() (you can verify it as an exercise).

Hope this addresses you concerns!
