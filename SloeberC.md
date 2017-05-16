### Basic Infos

#### Hardware

```
Hardware:     ESP12, Adafruit Huzzah Breakout and Feather Boards  
Core Version: 2.3.0
Computer:     Dell, Intel Core i7 CPU, 870 @ 2.93Ghz,  16.0 GB RAM
```

#### OS, IDE, JAVA

``` 
OS:   Windows 10 Professional   
IDE:  Identified by '.tar' file

Latest nightly build
     io.sloeber.product-4.0.1-20170507.142757-56-win32.win32.x86_64

Current stable release
      V4.0_win64.2017-01-17_15-14-48

Last successful install
      io.sloeber.product-4.0.1-20170401.212426-33-win32.win32.x86_64

JAVA: Version 8 Update 131 (Build 1.8.0_131-b11)

```

### Description

After un-successfully updating my last successful install of Sloeber, I decided for time expediency just to re-install using the latest nightly build. Much to my surprise, I got the following 'Problem Occurred' message just after starting the install:

```
Unable to download http://arduino.esp8266.com/stable/package_esp8266com_index.json
Server returned HTTP response code: 403 for URL: http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

The install then continues until 'make.zip' is being processed, then terminates prematurely. This results in the board libraries for the ESP8266 part not being installed. The work/error log is included at the end of this resport.

Next, I tried to re-install using the current stable version, with similar results, but WITHOUT the 'Problem Occurred' message. The work/error logs were almost identical, with slight differences in line numbers show for Java exceptions. 

Lastly, I re-installed the Sloeber version I had been using, with results identical to the stable build install (work/error log not included, as it contained no new information).

### Problem WorkAround

I was able to complete the install manually for all fore-mentioned IDE versions by:

1) Using the IDE's internal browser to download 'http://arduino.esp8266.com/stable/package_esp8266com_index.json' to the install's 'arduinoPlugin' (internal browser was used to see if it would choke as well):
```
a) Access the 'Window > Show View > Other' menu.
b) Expand 'General' catagory, then select 'Internal Web Browser'.
c) Click the web-browser tab, then paste in the URL for the missing
   'package_esp8266com_index.json' file, followed by return.
d) Click 'Save' on the 'File Download' pop-up window.
e) Navigate to the 'arduinoPlugin' folder using the 'Save As'
   dialog, then click 'Save'.
f) Close the 'Download Complete' dialog box.
g) Access 'File > Restart' to restart the Sloeber IDE.
```

2) Selecting and installing an Esp8266 SDK version:
```
a) Access the 'Window > Preferences' menu.
b) Expand the 'Arduino' sub-menu, and click on 'Platforms and Boards'.
c) Expand, then check the 'Esp8266/Esp8266/V2.3' selection,
   then wait for the SDK install to finish.
d) Access 'File > Restart' to again restart the Sloeber IDE.
```

The SDK install was tested by creating the 'blink' sketch example, then compiling and uploading it to the Huzzah Feather module.

### Discussion

First some prior history...

My very first attempts at installing the Sloeber IDE were un-successful; the ESP8266 board selections were always missing. [Per advice found on the Arduino forum][wait_besure], I even waited upwards to 20 minutes after the install seeming completed to insure that it had.

[wait_besure]: https://forum.arduino.cc/index.php?topic=413731.msg3020434#msg3020434

Noting that the Windows firewall was interrupting the install to ask for permissions, I then turned off the firewall reasoning that the interruption was the problem, and achieved a SUCCESSFUL INSTALL. Attempts to repeat this failure/recovery were fruitless, however, as EVERY install thereafter completed successfully; obviously some hidden variable(s) had changed.

Now back to the present...

Following the un-successful update attempt, I got several messages warning of missing files/downloads. Wanting to get on with my work, I decided a re-install with the latest nightly build would be the fastest remedy.

Given that I cannot now re-install what worked before, it's likely that another 'hidden' variable has changed. One cannot conclude, however, that the update itself caused the problem. Unfortunately, I didn't record any update warning messages before discarding it, which may have provided useful clues.

Regarding possible firewall interactions, I can offer only the following observations:
```
1) During an install, I no longer get prompts for permissions
   to allow 'sloeber-ide.exe' to continue.
2) When I check to see what apps are allowed to communicate,
   I no longer see any 'sloeber-ide.exe' entries.
3) Turning off the firewall before installation now has no effect.
4) Pre-enabling the 'sloeber-ide.exe' firewall permissions before
   installation also has no effect.
```

** Let me know if I can do any testing to help resolve this issue. **


#### Work/Error Log for Latest Nightly Build Install
```
eclipse.buildId=unknown
java.version=1.8.0_131
java.vendor=Oracle Corporation
BootLoader constants: OS=win32, ARCH=x86_64, WS=win32, NL=en_US
Framework arguments:  -perspective io.sloeber.application.perspective
Command-line arguments:  -os win32 -ws win32 -arch x86_64 -perspective io.sloeber.application.perspective

!ENTRY org.eclipse.egit.ui 2 0 2017-05-15 04:57:56.188
!MESSAGE Warning: The environment variable HOME is not set. The following directory will be used to store the Git
user global configuration and to define the default location to store repositories: 'C:\Users\DELL'. If this is
not correct please set the HOME environment variable and restart Eclipse. Otherwise Git for Windows and
EGit might behave differently since they see different configuration options.
This warning can be switched off on the Team > Git > Confirmations and Warnings preference page.

!ENTRY io.sloeber.core 2 0 2017-05-15 04:57:57.282
!MESSAGE Failed to download url http://arduino.esp8266.com/stable/package_esp8266com_index.json
!STACK 0
java.io.IOException: Server returned HTTP response code: 403 for URL: http://arduino.esp8266.com/stable/package_esp8266com_index.json
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(Unknown Source)
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream(Unknown Source)
	at java.net.URL.openStream(Unknown Source)
	at io.sloeber.core.managers.Manager.myCopy(Manager.java:793)
	at io.sloeber.core.managers.Manager.loadJson(Manager.java:217)
	at io.sloeber.core.managers.Manager.loadJsons(Manager.java:172)
	at io.sloeber.core.managers.Manager.startup_Pluging(Manager.java:89)
	at io.sloeber.core.Activator$2.run(Activator.java:198)
	at org.eclipse.core.internal.jobs.Worker.run(Worker.java:55)

!ENTRY io.sloeber.core 4 0 2017-05-15 04:57:57.282
!MESSAGE Unable to download http://arduino.esp8266.com/stable/package_esp8266com_index.json
!STACK 0
java.io.IOException: Server returned HTTP response code: 403 for URL: http://arduino.esp8266.com/stable/package_esp8266com_index.json
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(Unknown Source)
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream(Unknown Source)
	at java.net.URL.openStream(Unknown Source)
	at io.sloeber.core.managers.Manager.myCopy(Manager.java:793)
	at io.sloeber.core.managers.Manager.loadJson(Manager.java:217)
	at io.sloeber.core.managers.Manager.loadJsons(Manager.java:172)
	at io.sloeber.core.managers.Manager.startup_Pluging(Manager.java:89)
	at io.sloeber.core.Activator$2.run(Activator.java:198)
	at org.eclipse.core.internal.jobs.Worker.run(Worker.java:55)
```

#### Work/Error Log for Current Stable Build Install
```
!SESSION 2017-05-16 02:11:03.692 -----------------------------------------------
eclipse.buildId=unknown
java.version=1.8.0_131
java.vendor=Oracle Corporation
BootLoader constants: OS=win32, ARCH=x86_64, WS=win32, NL=en_US
Framework arguments:  -perspective io.sloeber.application.perspective
Command-line arguments:  -os win32 -ws win32 -arch x86_64 -perspective io.sloeber.application.perspective

!ENTRY org.eclipse.egit.ui 2 0 2017-05-16 02:11:38.535
!MESSAGE Warning: The environment variable HOME is not set. The following directory will be used to store the Git
user global configuration and to define the default location to store repositories: 'C:\Users\DELL'. If this is
not correct please set the HOME environment variable and restart Eclipse. Otherwise Git for Windows and
EGit might behave differently since they see different configuration options.
This warning can be switched off on the Team > Git > Confirmations and Warnings preference page.

!ENTRY io.sloeber.core 2 0 2017-05-16 02:11:39.410
!MESSAGE Failed to download url http://arduino.esp8266.com/stable/package_esp8266com_index.json
!STACK 0
java.io.IOException: Server returned HTTP response code: 403 for URL: http://arduino.esp8266.com/stable/package_esp8266com_index.json
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(Unknown Source)
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream(Unknown Source)
	at java.net.URL.openStream(Unknown Source)
	at io.sloeber.core.managers.Manager.myCopy(Manager.java:890)
	at io.sloeber.core.managers.Manager.loadPackageIndex(Manager.java:236)
	at io.sloeber.core.managers.Manager.loadIndices(Manager.java:188)
	at io.sloeber.core.managers.Manager.startup_Pluging(Manager.java:92)
	at io.sloeber.core.Activator$2.run(Activator.java:199)
	at org.eclipse.core.internal.jobs.Worker.run(Worker.java:55)
```
