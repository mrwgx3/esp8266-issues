``` 
jantje commented 7 hours ago

This is actually a issue for https://github.com/jantje/hardware repository and not for sloeber.
Do I understand correctly that the esp tool used is a wrong version?
```

Yes, that is correct; I've verified this for (2) separate installations. You did comment in the video that both the 1.65 and 2.30 SDK's were needed for debugging; I thought perhaps this was the reason.

To explore what 'esptool' version would upload Project A's debug sketch, I did some experimentation using the command prompt for:
```
a) My original Feather board, with peripherals attached (I2C bus, SD card, and interrupt lines; pretty full plate)
b) Another Feather board, no peripherals attached
c) My Breakout board, also sans peripherals
```
Test case B was added as a cross-check, as my original circuit needs all the lines controlling the ESP-12 boot mode for I/O. During reset, external logic fixes these line states to insure proper booting, then connects them to where they're needed.

Included below are excerpts from my notes recording the command prompt results for various esptool uploads:
```
================================

Test upload which failed using admin. command prompt:

Huzzah Feather Board, on protoboard with peripherals
C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd ck -cb 921600 -cp COM4 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
warning: espcomm_sync failed
error: espcomm_open failed
error: espcomm_upload_mem failed

Huzzah Feather Board, stand-alone
C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd ck -cb 921600 -cp COM8 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
warning: espcomm_sync failed
error: espcomm_open failed
error: espcomm_upload_mem failed

NOTE: The 'esptool' makes (3) attempts at connection before generating failure messages; the RED led followed BLUE led flash for each attempt.

Huzzah Breakout Board, stand-alone, expect this to work
C:\WINDOWS\system32>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd ck -cb 115200 -cp COM6 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Access is denied.

Big Surprise!
When I used the above command, got a pop-up window stating:
  "This app can't run on your PC
   To find a version for your PC, check with the software publisher."

Futhermore, the following 'esptool' will not run now either on command line or through the Sloeber IDE
C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe

Now try 'esptool' from another install...

Huzzah Breakout Board, stand-alone
NOTE THAT THIS WORKS, IDENTICAL WITH PRIOR COMMANDS EXCEPT FOR COMPORT AND BAUDRATE SELECTION

C:\WINDOWS\system32>C:\SloeberBeR2\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd ck -cb 115200 -cp COM6 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
................................................................................ [ 37% ]
................................................................................ [ 75% ]
....................................................

Success! what just got changed!?
Go inspect file, found, but reported to be '0 bytes' in size.
Move to 'DamagedEsptool' folder
Copy 'esptool.exe' from 'SloeberBeR2' install to repair, test again...
THINGS SEEM BACK TO NORMAL
Suspect I damaged the file myself with a bad paste to the command line.

================================
```
Given the upload failures for both Feather boards, I conclude my peripheral hardware is not affecting the outcome. A successful upload to the Breakout board indicates the Feather board may be a key variable.

```
================================

1st test of alternate upload(s):
  Older 'esptool' version, with args identical those just used with the 'newer' tool that failed.

Huzzah Feather Board, on protoboard with peripherals
C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.5\esptool.exe -cd ck -cb 921600 -cp COM4 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
....................................................................................................................................................................................................................

Huzzah Feather Board, stand-alone
C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.5\esptool.exe -cd ck -cb 921600 -cp COM8 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
....................................................................................................................................................................................................................

Huzzah Breakout Board, stand-alone, expect this to work
C:\WINDOWS\system32>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.5\esptool.exe -cd ck -cb 115200 -cp COM6 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
....................................................................................................................................................................................................................
```
Given all uploads worked using the 'older' esptool with identical args, it would seem the 'newer' esptool needs a different set of arguments. Hence its likely the invocation of the 'newer' tool here is most likely an error.

```
================================

2nd test of alternate upload(s):
  Newer 'esptool' version works with change of reset method, all cases.

C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd nodemcu -cb 921600 -cp COM4 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
................................................................................ [ 37% ]
................................................................................ [ 75% ]
....................................................                             [ 100% ]

C:\Users\DELL>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd nodemcu -cb 921600 -cp COM8 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
................................................................................ [ 37% ]
................................................................................ [ 75% ]
....................................................                             [ 100% ]

Huzzah Breakout Board, stand-alone, expect this to work
C:\Users\DELL>C:\WINDOWS\system32>C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.9\esptool.exe -cd nodemcu -cb 115200 -cp COM6 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65\debug\blink_wtdelay_gdbA_r1p65.bin to flash at 0x00000000
................................................................................ [ 37% ]
................................................................................ [ 75% ]
....................................................                             [ 100% ]

================================
```
This last test would indicate that the 'newer' esptool doesn't have an intrisic fault; it just needs different args.

Given this additional testing, its likely that
```
a) Preventing the 'newer' esptool from being invoked when the 'older' esptool is needed, and
b) Using a modified set of arguments when the 'newer' esptool is needed
```

would correct the problems seen the Huzzah Feather board.

