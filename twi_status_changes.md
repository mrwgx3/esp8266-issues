### Add 'clockCount' decrement, while-loop, twi_status()


While researching solutions for a stalled I2C bus, I found the following issue and pull-request:

I2C bus reset with status info to user, re issue 1025 #2058  
method for recovering I2C bus #1025  

The software provide by both @drmpf and @dave-prosee both worked fine, but the while-loop inside Dave's ['twi_status()'][twi_status] seems to be missing a 'clockCount' decrement, as noted in pull-request #2058 by both @Frida854 and @vlast3k. This pull-request adds the missing decrement.

[twi_status]: https://github.com/esp8266/Arduino/blob/master/cores/esp8266/core_esp8266_si2c.c#L201-L208 "I2C Bus Reset Code"

The following code was used for testing:
```
uint8_t  I2CIO::busInit ( void )
{
// Set I2C bus pins to INPUT_PULLUP
   Wire.begin( SDA, SCL );            // Default I2C bus pins used here

// Change bus clockrate and stretch timeout here
   Wire.setClock( 100000 );           // 'Wire.begin()' defaults
   Wire.setClockStretchLimit( 230 );  //  shown here

// Reset the I2C bus
   uint8_t  status = Wire.status();   // Wrapper for 'twi_status()'

// If I2C bus reset went OK, re-initialize bus
   if ( status == I2C_OK )
      Wire.begin();

   return( status );
   
} //busInit```
