I2C bus reset with status info to user, re issue 1025 #2058  
https://github.com/esp8266/Arduino/pull/2058  

reset I2C hangup #1025 #2027  
https://github.com/esp8266/Arduino/pull/2027  

method for recovering I2C bus #1025  
https://github.com/esp8266/Arduino/issues/1025

```
// Wrapper, I2C bus initialization
uint8_t  I2CIO::busInit ( void )
{
// Set I2C bus pins to INPUT_PULLUP
   Wire.begin( SDA, SCL );            // Default I2C bus pins used here

// Change bus clockrate and stretch timeout here
   Wire.setClock( 100000 );           // 'Wire.begin()' defaults
   Wire.setClockStretchLimit( 230 );  //  used here

// Reset the I2C bus
   uint8_t  status = Wire.status();

// If I2C bus reset went OK, re-initialize bus
   if ( stat == I2C_OK )
      Wire.begin();

   return( status );
   
} //busInit
```
