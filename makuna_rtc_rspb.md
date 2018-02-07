@makuna
```
Ok, thinking about the model to expose the temperature values, I keep getting back the core requirement
is to not require floats; but the inherent problem of dealing with a negative value in the range of one
degree is not solvable easily without them. By introducing the separate concept of sign that you have
done is an attempt but it makes it harder for comparisons and math (even though they were flawed before).
```

The problem has been already solved by the RS3231 RTC itself, you don't need floats. The concatentation of DS3231 temperature registers R11/R2 is the __primary__ math object, not the float derived from it. It has a sign, magnitude, and you can do math (including comparisons) natively with the compiler. You need nothing else!

From the commentary on your proposed (2) step solution, I'm guessing you still want to use the following variable triad as your primary math object, considering your willingness to write math operators using it:

```
int8_t sign() // returns -1 or 1 for sign.  
uint8_t integer() // returns the unsigned whole  integer of the value, you must apply sign yourself
uint8_t fractional() // returns the fractional value (0, 25, 50, 75)
```
There is a simplier way to avoid the need for cumbersome scaling in order to do math, one hinted at by the 'fractional()' routine:

Just use a __decimal__ scaling (x100) for integer temperatures, as it is quite easy to multiply by 100 in one's head.

It's almost as easy to do the scaling in software, just right-shift (6) places to convert to the native temperature scaling from 256 to 4, then multiply by 25 to achieve a (x 100) scaling. I would let the constructor handle the 256-to-4 rescaling, then change all the other routines to use the (x4) scale.

The integer temperature method looks like:
```
    // Integer temperature, scaled to decimal degrees
    int16_t AsDecimalDegC()
    {
       return( 25 * scaledDegC );  // Scale change from (x4) to (x100)
    }
```
It's easy to do human-friendly math and comparisons:
```
  degreesF = (int16_t)(  ( 320000 + td.AsDecimalDegC() * (uint32_t)180 ) / 100  );  // Fahrenheit conversion
  if ( degreesF > 21200 )
      Serial.print( "My tea is boiling..." );
```

__I would only use the sign/int/frac triad methods only for printing.__

```
I would also change the sample code to not use printf and instead use direct value
prints (printf is also heavy in code).
```
See attached sketch (text format), DS3231_TemperatureB.ino.txt
```
I will investigate adding a print to it that will minimize the complexity of
printing the temperature.
```
All changes discussed here have been put into my fork of your library, in branch ['getTemp_rework_take5'][rtc_rework_final]. The attached sketch demonstrates the use of the new methods.

Review the code, test it, rename things however you wish.


[rtc_rework_final]: https://github.com/mrwgx3/Rtc/tree/getTemp_rework_take5  "RtcTemperatue Rework"
