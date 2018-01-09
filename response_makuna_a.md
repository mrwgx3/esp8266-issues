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

Before answering your questions, let's first proceed with an allegory (thinly veiled, of course):

Say you just got the preliminary spec. on Maxim's newest RTC, the DS3231HE, the RTC designed
for HELL, and asked me to code the part. As Hell's denizens wished to precisely predict
when Hell freezes over while it's happening (a New Year's Eve Ball-Drop kind of thing, only more so),
Maxim's designers gave them the best 2's complement fixed point word width and alignment
they could come up with on short notice:
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

To write a new AsWholeDegrees() to accomodate the new off-boundary alignment, one must:
```
     a) Concatentate the temperature registers into a single 16-bit signed integer,
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
//    SF_GETFRAC = 15625, MASK_GETFRAC = 0x003F, and RSF_GETFRAC = 0
//    Function returns  { 0, 15625 through 984375 }
//
uint32_t Proto_GetFractional( int8_t r11h, uint8_t r12h )

{
   int16_t wordTemp = (int16_t)( (r11h << 8) + r12h ); // (a)
   int8_t  sgnT = ( wordTemp < 0 ) : -1 : 1;           // (b)
   return  SF_GETFRAC *
           ( MASK_FRAC & ((sgnT * wordTemp) >> RSF_GETFRAC) )  // (c-e)

} //Proto_GetFractional

As I'm finishing up the above code, however, Maxim sends a final project update quoting Hell's CEO: 

"Given general human nature, Hell isn't isn't likely to freeze over anytime soon. Furthermore, with the
recent upturn in bizzare human behavior, Hell's not likely to be experiencing any lack of "cold days"
either. The standard part will work fine, and the DS3231HE project is shelved until further notice."

No worries, just a final tweek the constants:
```
The original fixed point word and alignment is:

      |         r11h          | DP |         r12h         |
Bit:   15 14 13 12 11 10  9  8   .  7  6  5  4  3  2  1  0  -1 -2 -3
        s  i  i  i  i  i  i  i   .  f  f  0  0  0  0  0  0

and

RSI_GETFRAC  = 8       to align the LSbit of the integer    part to bit 0
                       the overall word scale-factor is 256

RSF_GETFRAC  = 6       to align the LSbit of the fractional part to bit 0
MASK_GETFRAC = 0x0003  to extract the fractional part
SF_GETFRAC   = 25      to scale the result

and GetFractional() now returns {0, 25, 50, and 75 }

```
But wait you say, "With the decimal point now falling on a byte boundary, along with the greatly reduced resolution, can't one simplify the code such that AsWholeDegrees() and GetFractional() are only functions
of 'r11h' and r12h', respectively?

### This is what I assumed you meant by your question, @Makuna. ###

Alas, no, due to the "1's complement + 1" nature of 2's complement (signed) arithmetic.
```
a) Knowing when to negate both integer and fractional portions always needs sign test of the integer part.

b) Adding (1) to a negated integer portion is only required when the original fractional part equals zero,
   hence requires a test of the fractional part.

```
It's inescapable, to write a either a AsWholeDegrees() or GetFractional(), you need both temperature registers.
