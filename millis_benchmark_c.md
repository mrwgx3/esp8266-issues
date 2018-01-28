Hi All

It turns out that I had (2) problems with the proposed 'millis()' solution:
```
a) For high values of 'm' and 'c', the calculation exceeded the 32-bit word
   length used. This is why things seem to blow up around  (7) years; one
   which I consider unacceptably short.

b) The approximation occasionally did not match the gold-standard code, and was
   typically off by +/- 1, pretty much throughout the ranges for both 'm' and 'c'.

```
The only thing left to explore was to find a fast implementation for the 64-bit arithmetic used by the slow 'gold-standard' routine. After doing a little research, I found that one can do fixed-point division by a constant simply multiplying by a scaled multiplicative inverse of the divisor. These 'magic numbers' are often used by compilers when dividing by small constants. See this link for a good discussion:

http://ridiculousfish.com/blog/posts/labor-of-division-episode-i.html

Given a magic number of 64-bits precision and a divisor of 10-bits precision (1000), one can accurately divide a dividend with up to 54-bits precision simply by multiplying by the magic number. This corresponds a dividend range of (54-32 = 22 bits), or 0x400000, which translates to about 570 years of usec counter overflows.
```
//---------------------------------------------------------------------------
// millis() 'magic multiplier' approximation
// Input:
//    'm' - 32-bit usec counter,           0 <= m <= 0xFFFFFFFF
//    'c' - 16-bit usec overflow counter   0 <= c <  0x00400000
// Output:
//    Returns milliseconds in modulo 0x10000000  ( 0 to 0xFFFFFFFF)
//
// A 'magic multiplier', equal to '(2^n) / const', is used to
// approximate fixed-point division by a constant.  For this
// application, k = Ceiling[ 2^64 / 1000 ] (max. bits we can use)
//
// A distributed multiply with offset-summing is used find k( 2^32 c + m):
//  prd = (2^32 kh + kl) * ( 2^32 c + m )  
//      = 2^64 kh c + 2^32 kl c + 2^32 kh m + kl m
//           (d)         (c)         (b)       (a)
//
// Important: ** Don't mess with (uint64_t) type castings **

#define  MAGIC_1E3_wLO  0x4bc6a7f0    // LS part
#define  MAGIC_1E3_wHI  0x00418937    // MS part

unsigned long ICACHE_RAM_ATTR millis_test ( void )
{
  uint32_t  a[3];  // 96-bit accumulator, little endian
  a[2] = 0;        // Zero high-acc

  // Get usec system time, usec overflow counter
  uint32_t  m = system_get_time();
  uint32_t  c = us_ovflow + ((m < us_at_last_ovf) ? 1 : 0);

  ((uint64_t *)(&a[0]))[0]  =              // (a) Init. low acc
     ( m * (uint64_t)MAGIC_1E3_wLO );

  ((uint64_t *)(&a[1]))[0] +=              // (b) Offset sum, mid-acc
     ( m * (uint64_t)MAGIC_1E3_wHI );

  ((uint64_t *)(&a[1]))[0] +=              // (c) Offset sum, mid-acc
     ( c * (uint64_t)MAGIC_1E3_wLO );
  
  ((uint32_t *)(&a[2]))[0] +=              // (d) Truncated sum, high-acc
     (uint32_t)( c * (uint64_t)MAGIC_1E3_wHI );

  return ( a[2] );  // Extract result, high-acc

} //millis_test
```
I have exhaustively tested the above code using the Borland C++ Builder v4.0 compiler for 0 < m < (2^32-1) at c = 0x400000, and did sampled testing for 0 < 'c' <= 0x400000. Benchmark times for the ESP8266 Huzzah Feather were:

```
Millis RunTime Benchmark

         usec   x Orig   Comment
 Orig:   3.18   1.00     Original code

 Corr:  13.94   4.38     64-bit reference code, DEBUG
 Test:   5.61   1.76     64-bit magic multiply, 4x32 DEBUG

 Corr:  13.21   4.15     64-bit reference code
 Test:   4.66   1.46     64-bit magic multiply, 4x32

*** End, Bench Test ***
```
The magic multiplier ran ~3x faster than the gold-standard. Execution times, however, vary considerably with the numbers being multiplied, so one should probably lower this factor to the expected worst case of around x2.

Given the trade-offs between complexity and speed, this is best algorithm I've been able to come up with. Got any ideas for improvements? How do wish to proceed?

