```
     int8_t AsWholeDegrees()
     {
-        return integerDegrees;
+        int8_t  sgnT = ( (mstemp < 0) ? -1 : 1 );
+        return( sgnT * ((sgnT * AsScaledDegrees()) >> 8) ); // (x 1)

@Makuna
Why do the extra work within AsScaledDegrees to just undo it here.
Just return what you are currently calling mstemp.
```
I see by your question I still haven't communicated the fundamental nature of the bug.

Before answering your other questions, let's start with an allegory:

Say you just got the preliminary spec. on Maxim's newest RTC, the DS3231HE, the RTC designed for HELL,
and asked me to code the part. As Hell's denizens wished to precisely predict when Hell freezes over while
it's happening (a New Year's Eve BallDrop kind of thing, only more so), Maxim's designers gave them the
best 2's complement fixed point word width and alignment they could come up with:
``` 
      |         r11h           |      DP     r12h         |
Bit:   15 14 13 12 11 10  9  8   7  6  .  5  4  3  2  1  0  -1 -2 -3
        s  i  i  i  i  i  i  i   i  i  .  f  f  f  f  f  f   0  0  0
```
Only (2) extra bits were needed for the integer portion, because even when Hell
freezes over in its biggest way, it still only approaches absolute zero, or -273.15 degC.
Due to the extra integer bits, however, the decimal point is no longer aligned at a byte boundary.

As it takes (6) right-shifts to register the decimal point (DP) to the right of the 0th bit,
the overall word scaling equals 64. Given (6) fractional bits, our fractional resolution
is 1/64th of a degree.

To write a new AsWholeDegrees() to accommodate the off-boundary alignment, one must:
```
     a) Concatentate the temperature registers into a single 16-bit signed integer
     b) Record the sign of the 16-bit result
     c) Find the absolute value of the concantenation
     d) Align the LSbit of the integer portion with the 0th bit of the word
     e) Restore the sign of the result, then return it
```
Writing a new GetFractional() is just as easy:
```
   a-c) Do steps (a-c) of the prototype AsWholeDegrees() function.
     d) Align the LSbit of the fractional portion with the 0th bit of the word 
     e) Extract the fractional part with a mask, multiply by an appropriate
        scale-factor, then return the result.
```
By noting that  Abs(x) = sign(x) * x, I can write these routines very compactly.
```
// For the DS3231HE, RSI_GETFRAC = 6
// Function returns -512 through 511
//
int16_t Proto_AsWholeDegrees( int8_t r11h, uint8_t r12h )
{
   int16_t wordTemp = (int16_t)( (r11h << 8) + r12h ); // (a)
   int8_t  sgnT = ( wordTemp < 0 ) : -1 : 1;           // (b)
   return  sgnT * ((sgnT * wordTemp) >> RSI_GETFRAC);  // (c-e)

} //Proto_AsWholeDegrees

// For the DS3231HE,
//    SF_GETFRAC = 15625, MASK_GETFRAC = 0x003F, and
//    RSF_GETFRAC = 0, as fractional part is already aligned
//    Function returns  { 0, 15625 through 984375 }
//
uint32_t Proto_GetFractional( int8_t r11h, uint8_t r12h )
{
   int16_t wordTemp = (int16_t)( (r11h << 8) + r12h );     // (a)
   int8_t  sgnT = ( wordTemp < 0 ) : -1 : 1;               // (b)
   return SF_GETFRAC *
      ( MASK_FRAC & ((sgnT * wordTemp) >> RSF_GETFRAC) );  // (c-e)

} //Proto_GetFractional
```
As I'm finishing up the above code, however, Maxim sends a final project update quoting Hell's CEO: 

"Given general human nature, Hell isn't isn't likely to freeze over anytime soon. Furthermore, with the
recent upturn in bizarre human behavior, there is no lack of "cold days", either. The standard part will
do fine, and the DS3231HE project is shelved until further notice."

No worries, just a final tweek of the constants:
```
The original fixed point word/alignment is:

      |         r11h          | DP |         r12h         |
Bit:   15 14 13 12 11 10  9  8   .  7  6  5  4  3  2  1  0  -1 -2 -3
        s  i  i  i  i  i  i  i   .  f  f  0  0  0  0  0  0
and

RSI_GETFRAC  = 8       to align the LSbit of the integer    part to bit 0
                       (overall word scale-factor is 256)
RSF_GETFRAC  = 6       to align the LSbit of the fractional part to bit 0
MASK_GETFRAC = 0x0003  mask to extract the fractional part
SF_GETFRAC   = 25      scale to decimal

GetFractional() now returns {0, 25, 50, and 75 }
```
But wait you say, "With the decimal point now falling on a byte boundary, along with the greatly reduced resolution, can't one simplify the code such that AsWholeDegrees() and GetFractional() are only functions
of 'r11h' and 'r12h', respectively?
```
Alas, no, due to the "1's complement + 1" nature of 2's complement (signed) arithmetic:

   a) Knowing when to negate both integer and fractional portions always needs sign test of the integer part.

   b) Adding (1) to the negated integer portion is only required when the original fractional part equals zero,
      hence requires a test of the fractional part.

It's inescapable, both temperature registers are needed for AsWholeDegrees() or GetFractional().
```
#### THis is what I assumed you meant by your question, Makuna.####  
No work is being undone, and 'mstemp' aka 'r11h' can't do the job alone.

Back to your questions, @smz. Noting your concerns about derived classes and coding for other RTC's,
I searched the web for:
```
   a) Any Arduino libraries written for the DS3231 RTC
   b) Any comparable parts, i.e., with an onboard, temperature compensated crystal oscillator
```
Regarding libraries, of the dozen or so that I examined, I only found (1) other library that looks similar to Makuna's. But as in your library, they tossed the RtcTemperature class and re-coded GetTemperature() to return a float.

Of the remaining libraries, around 40% used the 16-bit integer interpretation, 30% picked the deg/frac approach, and one even merged the two concepts together.

Regarding other RTC's, I only found (1) other device which returned a temperature, Intersil's ISL12022 RTC.
https://www.intersil.com/content/dam/Intersil/documents/isl1/isl12022.pdf

Their temperature register description (pg 21) is even more confusing than Maxim's. Their design is slightly more forward thinking as they've anticipated Hell freezing over; temperatures are reported in degrees Kelvin.

#### It would seem that the DS3231 is a unique part, and Makuna's library a unique approach. ####
I wouldn't worry too much about breaking derived classes.

Back to your remaining comments, @Makuna.

After finishing my research, I've come to the conclusion that the person who wrote the original temperature
register description did not understand the 2's complement nature of the registers. Consider instead:
```
Temperature Registers (11h–12h)
Temperature is represented as a 10-bit code with a resolution of 1/4th °C and is accessable as
a 16-bit, signed integer at locations 11h and 12h. For example, at +/- 25.25°C, concatenated
registers <R11h:R12h> = 256 * (+/- 25+(1/4)) = +/- 6464, or 1940h / E6C0h.
```
#### The above description is correct, as determined by code verified through experiment. ####

For our allegorical 'HE' part, the description is just as succinct:
```
Temperature is represented as a 16-bit code with a resolution of 1/64th °C and is accessable as
a 16-bit, signed integer at locations 11h and 12h. For example, at +/- 25.27°C, concatenated
registers <R11h:R12h> ~= 64 * (+/- 25+(17.28/64)) = +/- 1617, or 0651h / F9AFh.
```
No need to mention integer/fraction alignments to confuse things.

If you'd read this description first, what primary variables would your have used in GetTemperature()?
Passed to and saved in RtcTemperature? Where would you have put your maths?
```
 protected:
-    int8_t integerDegrees;
-    uint8_t decimalFraction;
+    int8_t   mstemp;
+    uint8_t  lstemp;

In reality what you are defining is a more accurate fixed point math object with your implementation.

"integer" and "fractional" are still better variable names for these, not most significant and least significant.

BTW: avoid the name "temp" for variables as it really isn't descriptive to the object or algorithm in most cases, and specifically in this usage.
```
Regarding 'fixed point math object'...

No, I'm just using the fixed point math object provided by the DS3231.

Regarding 'integer" and 'fractional' vs 'mstemp' and 'lstemp'...

As I hope was shown by the allegory, "integer" and "fractional" in a 2's complement context are really describing
bit-field alignments. As demonstrated by the re-written register description, bit-fields are not required
to understand the DS3231 temperature register contents.

'mstemp' and 'lstemp' just transport these bit-fields, but I agree, a more descriptive primary variable could be defined.

Regarding the BTW...
'temp' is descriptive in the context of temperature, but could construed as 'temporary'.

```
+    // Scaled integer temperature (x 256)
+    int16_t AsScaledDegrees()

I don't like the name of this just due to the fact that its basically the raw "8.8 fixed point" math item.
"AsFixedPointDegrees" would be my suggestion.
```
My first inclination was to name it simply 'AsInteger()' in analogy to 'AsFloat()', but it has a non-unity
scale-factor. "AsFixedPointDegrees()" works for me.


As a new primary variable(s), I would propose using:
```
private:
int16_t  fxdpntTemperature;
int8_t   sgnT;               // Sign of 'fxdpntTemperature'
```
GetTemperature() still transfers register values:
```
RtcTemperature GetTemperature ( void )
{
   ....
   _wire.requestFrom(DS3231_ADDRESS, DS3231_REG_TEMP_SIZE);
   int8_t  r11h = _wire.read();                  // MS byte, signed temperature
   return RtcTemperature( r11h, _wire.read() );  // LS byte is r12h
}
```
The constructor creates the primary variable:
```
RtcTemperature( int8_t ubRegT, uint8_t lbRegT )
{
   fxdpntTemperature =  (int16_t)( (ubRegT << 8) | (lbRegT & 0xC0) );
   sgnT = ( (fxdpntTemperature < 0) ? -1 : 1 );  // Sign of 'fxdpntTemperature' 
}
```
Method 'AsScaledDegrees()' becomes 'AsFixedPointDegrees()'
```
//  Used either to...
//  a) As an interger alternative for temperature calculations
//     equals 256 * (temperature in degC)
//  b) Reconstruct the original temperature registers
//
int16_t  AsFixedPointDegrees ( void )
{
   return fixedpntTemperature;
}
```
AsScaledDegrees() would be then be replaced by 'fixedpntTemperature' in all other methods.

No direct access to primary var's, and maths are all in one place. What do you think?
