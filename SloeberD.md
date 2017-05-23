### Basic Infos

#### Hardware

```
Hardware:     ESP12, Adafruit Huzzah Breakout and Feather Boards  
Core Version: 2.3.0
Computer:     Dell, Intel Core i7 CPU, 870 @ 2.93Ghz,  16.0 GB RAM
```

#### OS, IDE

``` 
OS:   Windows 10 Professional   
IDE:  Sloeber	4.0.1.201705071425
```

### Description

Per your [Youtube video on how to setup Sloeber for hardware debugging of the ESP8266][debug_video], I successfully got debugging to work for the Adafruit [Huzzah Breakout Board][huzzah_breakout]. The [Huzzah Feather Board][huzzah_feather], however, seems to have timing issues with both the uploading of the debug sketch and initiating a debugger dialog.

[debug_video]: https://www.youtube.com/watch?v=S_QTMNhaDwM
[huzzah_breakout]: https://www.adafruit.com/product/2471
[huzzah_feather]: https://www.adafruit.com/product/2821

### Discussion
To facilitate testing, (2) 'blink' example projects were created:

```
   Debug Config.               Release Config.                 Serial     Project Title
A) Debug Board with 1.65 SDK   Huzzah Feather  with 1.65 SDK   COM4       blink_wtdelay_gdbA_r1p65
B) Debug Board with 1.65 SDK   Huzzah Breakout with 1.65 SDK   COM6       blink_wtdelay_gdbB_r1p65
```

Initially, only project A existed; project B was created after project A's debug sketch wouldn't upload. Project B's debug sketch did upload, and invoking the Debug perspective resulted in a successful debug session. Project A's debug sketch was then uploaded using the admin. command prompt to run a different 'esptool', but switching to the Debug perspective only resulted in a partial load and the following timeout message:

```
Error in final launch sequence
Failed to execute MI command:
-target-select remote /./COM4
Error message from debugger back end:
Bogus trace status reply from target: timeout
Bogus trace status reply from target: timeout
```
Hence it was concluded that the Huzzah Feather Board likely has some software/hardware timing issues.

### Messages of Interest

Project B Compiler/Upload Messages, Successful Debug Session
```
'Create eeprom image'
"C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/esptool/0.4.5/esptool.exe" -eo "C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/hardware/esp8266/1.6.5-947-g39819f0/bootloaders/eboot/eboot.elf" -bo "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.bin" -bm qio -bf 40 -bz 4M -bs .text -bp 4096 -ec -eo "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.elf" -bs .irom0.text -bs .text -bs .data -bs .rodata -bc -ec
'Finished building: blink_wtdelay_gdbB_r1p65.hex'
' '
'Building target: blink_wtdelay_gdbB_r1p65'
'Printing size:'
"C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/xtensa-lx106-elf-gcc/1.20.0-26-gb404fb9/bin/xtensa-lx106-elf-size" -A "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.elf"
C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.elf  :
section                           size         addr
.data                             1496   1073643520
.rodata                           1716   1073645024
.bss                             42192   1073646744
.irom0.text                     178504   1075843088
.text                            30240   1074790400
.debug_frame                      4520            0
.debug_info                      43855            0
.debug_abbrev                     9320            0
.debug_aranges                    1600            0
.debug_ranges                      784            0
.debug_line                      30937            0
.debug_str                       12145            0
.comment                          6680            0
.xtensa.info                        56            0
.xt.lit                           4824            0
.xt.prop                        160620            0
.xt.prop._ZTV6Stream                12            0
.xt.prop._ZTV14HardwareSerial       12            0
.xt.prop._ZTV5Print                 12            0
.debug_loc                       18402            0
Total                           547927


'Finished building target: blink_wtdelay_gdbB_r1p65'

Starting upload
using arduino loader
Starting reset using DTR toggle process
Toggling DTR
Continuing to useCOM6
Ending reset

LaunchingC:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/esptool/0.4.9/esptool.exe -cd ck -cb 115200 -cp COM6 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.bin 
Output:
Uploading 216096 bytes from C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbB_r1p65/debug/blink_wtdelay_gdbB_r1p65.bin to flash at 0x00000000
................................................................................ [ 37% ]
................................................................................ [ 75% ]
....................................................                             [ 100% ]
/arduinoPlugin/packages/esp8266/tools/esptool/0.4.9/esptool.exe finished
upload done
```

Project A Compiler/Upload Messages, Failed Upload
```
'Create eeprom image'
"C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/esptool/0.4.5/esptool.exe" -eo "C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/hardware/esp8266/1.6.5-947-g39819f0/bootloaders/eboot/eboot.elf" -bo "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.bin" -bm qio -bf 40 -bz 4M -bs .text -bp 4096 -ec -eo "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.elf" -bs .irom0.text -bs .text -bs .data -bs .rodata -bc -ec
'Finished building: blink_wtdelay_gdbA_r1p65.hex'
' '
'Building target: blink_wtdelay_gdbA_r1p65'
'Printing size:'
"C:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/xtensa-lx106-elf-gcc/1.20.0-26-gb404fb9/bin/xtensa-lx106-elf-size" -A "C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.elf"
C:/Users/DELL/SloeberWork/Wksr1/blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.elf  :
section                           size         addr
.data                             1496   1073643520
.rodata                           1716   1073645024
.bss                             42192   1073646744
.irom0.text                     178504   1075843088
.text                            30240   1074790400
.debug_frame                      4520            0
.debug_info                      43855            0
.debug_abbrev                     9320            0
.debug_aranges                    1600            0
.debug_ranges                      784            0
.debug_line                      30937            0
.debug_str                       12145            0
.comment                          6680            0
.xtensa.info                        56            0
.xt.lit                           4824            0
.xt.prop                        160620            0
.xt.prop._ZTV6Stream                12            0
.xt.prop._ZTV14HardwareSerial       12            0
.xt.prop._ZTV5Print                 12            0
.debug_loc                       18402            0
Total                           547927


'Finished building target: blink_wtdelay_gdbA_r1p65'

Starting upload
using arduino loader
Starting reset using DTR toggle process
Toggling DTR
Continuing to useCOM4
Ending reset

LaunchingC:\SloeberBeR1\/arduinoPlugin/packages/esp8266/tools/esptool/0.4.9/esptool.exe -cd ck -cb 921600 -cp COM4 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.bin 
Output:
warning: espcomm_sync failed
error: espcomm_open failed
error: espcomm_upload_mem failed

/arduinoPlugin/packages/esp8266/tools/esptool/0.4.9/esptool.exe finished
upload done
```

Command Line Used to Succesfully Upload Project A's Debug Sketch (use version 0.4.5 rather than 0.4.9)
```
C:\SloeberBeR1\arduinoPlugin\packages\esp8266\tools\esptool\0.4.5\esptool.exe -cd ck -cb 921600 -cp COM4 -ca 0x00000 -cf C:\Users\DELL\SloeberWork\Wksr1\blink_wtdelay_gdbA_r1p65/debug/blink_wtdelay_gdbA_r1p65.bin
```

### Requested Install Information
```
  C/C++ Autotools support	9.2.1.201703062208	org.eclipse.cdt.autotools.feature.group	Eclipse CDT
  C/C++ Common GDB Support	9.2.1.201703062208	org.eclipse.cdt.gdb.feature.group	Eclipse CDT
  C/C++ Development Platform	9.2.1.201703062208	org.eclipse.cdt.platform.feature.group	Eclipse CDT
  C/C++ Development Tools	9.2.1.201703062208	org.eclipse.cdt.feature.group	Eclipse CDT
  C/C++ DSF GDB Debugger Integration	9.2.1.201703062208	org.eclipse.cdt.gnu.dsf.feature.group	Eclipse CDT
  C/C++ GNU Toolchain Build Support	9.2.1.201703062208	org.eclipse.cdt.gnu.build.feature.group	Eclipse CDT
  C/C++ GNU Toolchain Debug Support	9.2.1.201703062208	org.eclipse.cdt.gnu.debug.feature.group	Eclipse CDT
  ECF Core Feature	1.3.0.v20160823-2221	org.eclipse.ecf.core.feature.feature.group	Eclipse.org - ECF
  ECF Core SSL Feature	1.1.0.v20160823-2221	org.eclipse.ecf.core.ssl.feature.feature.group	Eclipse.org - ECF
  ECF Filetransfer Feature	3.13.2.v20160823-2221	org.eclipse.ecf.filetransfer.feature.feature.group	Eclipse.org
  ECF Filetransfer SSL Feature	1.1.0.v20160823-2221	org.eclipse.ecf.filetransfer.ssl.feature.feature.group	Eclipse.org - ECF
  ECF Httpclient4 Filetransfer Provider	3.13.2.v20160823-2221	org.eclipse.ecf.filetransfer.httpclient4.feature.feature.group	Eclipse.org - ECF
  ECF Httpclient4 Filetransfer SSL Provider	1.1.0.v20160823-2221	org.eclipse.ecf.filetransfer.httpclient4.ssl.feature.feature.group	Eclipse.org - ECF
  Eclipse 4 Rich Client Platform	1.5.3.v20170228-0512	org.eclipse.e4.rcp.feature.group	Eclipse.org
  Eclipse Help System	2.2.2.v20170301-0400	org.eclipse.help.feature.group	Eclipse.org
  Eclipse Platform	4.6.3.v20170301-0400	org.eclipse.platform.feature.group	Eclipse.org
  Eclipse RCP	4.6.3.v20170301-0400	org.eclipse.rcp.feature.group	Eclipse.org
  EMF - Eclipse Modeling Framework Core Runtime	2.12.0.v20160420-0247	org.eclipse.emf.ecore.feature.group	Eclipse Modeling Project
  EMF Common	2.12.0.v20160420-0247	org.eclipse.emf.common.feature.group	Eclipse Modeling Project
  Equinox p2, backward compatibility support	1.2.203.v20170131-1444	org.eclipse.equinox.p2.extras.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, Discovery UI support	1.0.401.v20160901-1335	org.eclipse.equinox.p2.discovery.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, headless functionalities	1.3.203.v20170131-1444	org.eclipse.equinox.p2.core.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, minimal support for RCP applications	1.2.203.v20170131-1444	org.eclipse.equinox.p2.rcp.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, Provisioning for IDEs.	2.2.203.v20170131-1444	org.eclipse.equinox.p2.user.ui.feature.group	Eclipse.org - Equinox
  Git integration for Eclipse	4.6.1.201703071140-r	org.eclipse.egit.feature.group	Eclipse EGit
  Java implementation of Git	4.6.1.201703071140-r	org.eclipse.jgit.feature.group	Eclipse JGit
  Marketplace Client	1.5.4.v20170222-1941	org.eclipse.epp.mpc.feature.group	Eclipse Marketplace Client
  Mylyn Commons	3.21.0.v20160707-1856	org.eclipse.mylyn.commons.feature.group	Eclipse Mylyn
  Mylyn Commons Connector: Discovery	3.21.0.v20160729-1739	org.eclipse.mylyn.discovery.feature.group	Eclipse Mylyn
  Mylyn Commons Connector: Monitor	3.21.0.v20160630-1702	org.eclipse.mylyn.monitor.feature.group	Eclipse Mylyn
  Mylyn Commons Identity	1.13.0.v20160630-1702	org.eclipse.mylyn.commons.identity.feature.group	Eclipse Mylyn
  Mylyn Commons Notifications	1.13.0.v20160721-2347	org.eclipse.mylyn.commons.notifications.feature.group	Eclipse Mylyn
  Mylyn Commons Repositories	1.13.0.v20160630-1702	org.eclipse.mylyn.commons.repositories.feature.group	Eclipse Mylyn
  Mylyn Context Connector: Eclipse IDE	3.21.0.v20160912-1820	org.eclipse.mylyn.ide_feature.feature.group	Eclipse Mylyn
  Mylyn Context Connector: Team Support	3.21.0.v20160701-1337	org.eclipse.mylyn.team_feature.feature.group	Eclipse Mylyn
  Mylyn Task List	3.21.0.v20160914-0252	org.eclipse.mylyn_feature.feature.group	Eclipse Mylyn
  Mylyn Task-Focused Interface	3.21.0.v20160815-2336	org.eclipse.mylyn.context_feature.feature.group	Eclipse Mylyn
  Mylyn Tasks Connector: Bugzilla	3.21.0.v20160909-1813	org.eclipse.mylyn.bugzilla_feature.feature.group	Eclipse Mylyn
  Mylyn WikiText	2.10.1.v20161129-1925	org.eclipse.mylyn.wikitext_feature.feature.group	Eclipse Mylyn
  Nebula Oscilloscope Widget	1.2.0.201704251513	org.eclipse.nebula.widgets.oscilloscope.feature.feature.group	Eclipse Nebula
  Remote System Explorer End-User Runtime	3.7.2.201610260947	org.eclipse.rse.feature.group	Eclipse TM Project
  Sloeber	4.0.1.201705071425	io.sloeber.feature.feature.group	Jan Baeyens and Others
  Sloeber, the Eclipse Arduino IDE	4.0.0.201705071425	io.sloeber.product	null
```
