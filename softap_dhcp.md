### Basic Infos

#### Hardware

```
Hardware:     ESP12, Adafruit Huzzah Breakout and Feather Boards  
Core Version: 2.3.0
```

#### OS, IDE, and SDK

``` 
OS:   Windows 10 Professional and Linux Ubuntu 16.04LTS   
IDE:  1.6.12  
SDK:  1.5.3_16_04_18
```

### Description

While in AP+STA mode, it was observed that DHCP addresses assigned by the AP side sometimes differed from those specified by the [network parameters passed in Wifi.softAPConfig()][wifi_softAPConfig]. These variations were traced to the [debug level][debugging_md] chosen for the 'Generic ESP8266 Module' board setting, with erroneous addresses being generated for levels **not** containing a 'Wifi' entry. Incorrect addresses were also seen for the 'Adafruit HUZZAH ESP8266' board setting, which doesn't expose any debug levels at all.

Further tests revealed that when a [specific DEBUG\_WIFI() call is NOT made within Wifi.softAPConfig()][softAP_crit_debugwifi], the results of a [puzzling (perhaps incomplete) DCHP address calculation][softAP_addrcalc] are used. With the 'Wifi' flag set, however, the critical DEBUG_WIFI() call is made, and the correct DHCP addresses are somehow calculated and set; _where and how is not readily apparent_.

Also noted was that some address ranges caused Wifi.softAPConfig() to fail due to the calculated starting/ending DHCP address range being inverted (end > start); an example is included in the attached sketch.

Additional items:

* Tried the same tests for just AP mode; the results were the same.
* The title for library file [ESP8266WiFiAP.cpp][title_line] should be changed from "ESP8266WiFiSTA.cpp" to "ESP8266WiFiAP.cpp".

[debugging_md]: https://github.com/esp8266/Arduino/blob/master/doc/Troubleshooting/debugging.md  "Debug Features Tutorial"
[wifi_softAPConfig]: https://github.com/esp8266/Arduino/blob/2.3.0/libraries/ESP8266WiFi/src/ESP8266WiFiAP.cpp#L173#L179  "Sets IP, Gateway, Subnet"
[softAP_crit_debugwifi]: https://github.com/esp8266/Arduino/blob/2.3.0/libraries/ESP8266WiFi/src/ESP8266WiFiAP.cpp#L180  "1st DEBUG_WIFI()"
[softAP_addrcalc]: https://github.com/esp8266/Arduino/blob/2.3.0/libraries/ESP8266WiFi/src/ESP8266WiFiAP.cpp#L202#L215  "DCHP address calculation"
[title_line]: https://github.com/esp8266/Arduino/blob/2.3.0/libraries/ESP8266WiFi/src/ESP8266WiFiAP.cpp#L2 "Incorrect title line"


### Settings in IDE

```
Module:          Adafruit HUZZAH ESP8266  
Flash Size:      4MB (3M SPIFFS)  
CPU Frequency:   80Mhz  
Debug Settings:  None presented by IDE  
```

```
Module:          Generic ESP8266 Module  
Flash Size:      4MB (3M SPIFFS)  
CPU Frequency:   80Mhz  
Flash Mode:      qio  
Flash Frequency: 40Mhz  
Upload Using:    Disabled or SERIAL  
Reset Method:    nodemcu  
Debug Port:      None or Serial
Debug Level:     None, Core, or Core+Wifi
```

### Sketch Outline

```
The sketch included at the end of this post is used to:
1. Place the ESP12 in AP + STA mode
2. Configure and start the AP side
3. Statically connect to the home network on the STA side
4. Start a minimal web server to check for connectivity
```

### History

This issue was noticed while developing the attached sketch in 'Generic ESP8266 / Core+Wifi' mode. The debug messages from the soft-AP configuration and startup are:

```
scandone  
del if0  
mode : softAP(5e:cf:7f:c6:9d:bc)  
bcn 0  
del if1  
usl  
add if1  
dhcp server start:(ip:192.168.4.1,mask:255.255.255.0,gw:192.168.4.1)  
bcn 100  
Configuring as AP+STA  
Setting soft-AP configuration ...  
[APConfig] DHCP IP start: 192.168.4.108    << NOTE THESE ADDRESSES  
[APConfig] DHCP IP end: 192.168.4.208  
Ready  
Setting soft-AP ...   
bcn 0  
del if1  
usl  
add if1  
dhcp server start:(ip:192.168.4.9,mask:255.255.255.248,gw:192.168.4.9)  
bcn 100  
Ready  
Soft-AP IP address = 192.168.4.9  
```

The expected DHCP starting and ending addresses expected from the sketch's WiFi.softAPConfig( ip:192.168.4.9, gw:192.168.4.9, mask:255.255.255.248 ) call would be 192.168.4.9 and 182.168.4.14; quite different from reported addresses of 192.168.4.108 and 192.168.4.208.  A check via an 'ipconfig /all' after connecting to the AP, however, shows all but the [subnet mask][subnet_issue] to be in order:

[subnet_issue]: https://github.com/esp8266/Arduino/issues/2578 "Subnet mask always 255.255.255.0"

```
Wireless LAN adapter Wi-Fi 3:  

   Connection-specific DNS Suffix  . :  
   Description . . . . . . . . . . . : 150Mbps Wireless 802.11bgn Nano USB Adapter  
   Physical Address. . . . . . . . . : xx-xx-xx-xx-xx-xx  
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes  
   IPv4 Address. . . . . . . . . . . : 192.168.4.10(Preferred)  
   Subnet Mask . . . . . . . . . . . : 255.255.255.0       << INCORRECT, BUT A KNOWN ISSUE
   ....  
   Default Gateway . . . . . . . . . : 192.168.4.9  
   DHCP Server . . . . . . . . . . . : 192.168.4.9  
   ....  
```

Both the gateway (192.168.4.9) and wireless adapter (192.168.4.10) were also pingable, and the webserver root response was observed after entering '192.168.4.9/' into a browser. Given that everything seemingly works, it was concluded that the DHCP addresses were _mis-reported_.

After debugging, the 'Adafruit HUZZAH ESP8266' configuration was selected and the sketch recompiled. Everything matches the prior debug output, except for the now absent [APConfig] starting and ending DHCP messages. An 'ipconfig /all' check, however, revealed the actual starting DHCP address now matched the "mis-reported" starting address seen earlier:

```
Wireless LAN adapter Wi-Fi 3:  

   Connection-specific DNS Suffix  . :  
   Description . . . . . . . . . . . : 150Mbps Wireless 802.11bgn Nano USB Adapter  
   ....  
   IPv4 Address. . . . . . . . . . . : 192.168.4.108(Preferred)  
   Subnet Mask . . . . . . . . . . . : 255.255.255.0  
   ....  
   Default Gateway . . . . . . . . . : 192.168.4.9  
   DHCP Server . . . . . . . . . . . : 192.168.4.9  
   ....  
```

The wireless adapter (192.168.4.108) was pingable, but the gateway address (192.168.4.9) was not. The webserver root response was no longer observable as well.

### Additional Troubleshooting

The DHCP address calculations are found in [ESP8266WiFiAPClass::softAPConfig()][softAP_addrcalc]:

```cpp
 
    struct dhcps_lease dhcp_lease;              // LINE 202
    IPAddress ip = local_ip;
    ip[3] += 99;
    dhcp_lease.start_ip.addr = static_cast<uint32_t>(ip);
    DEBUG_WIFI("[APConfig] DHCP IP start: %s\n", ip.toString().c_str());

    ip[3] += 100;
    dhcp_lease.end_ip.addr = static_cast<uint32_t>(ip);
    DEBUG_WIFI("[APConfig] DHCP IP end: %s\n", ip.toString().c_str());

    if(!wifi_softap_set_dhcps_lease(&dhcp_lease)) {
        DEBUG_WIFI("[APConfig] wifi_set_ip_info failed!\n");
        ret = false;
    }
```

The errant '192.168.4.108' address arises from the 'ip[3] += 99' addition, i.e., 99 + 9 = 108.

Given correct DHCP addresses were seen while 'Wifi+Core' debug messages were active, while erroneous ones occured for the 'Huzzah' configuration, all other 'Generic' debug message configurations were checked. Erroneous addresses were observed only when the 'Wifi' debug flag was OFF.

Systematic comment/uncommenting of active DEBUG_WIFI() lines isolated the [1st message line in 'softAPConfig()'][softAP_crit_debugwifi] as the critical one; commenting this line out while the 'Wifi' debug level was active caused erroneous DHCP addresses. More expermentation showed that the length of the variable arglist to be important:

```cpp
/**
 * Configure access point
 * @param local_ip      access point IP
 * @param gateway       gateway IP
 * @param subnet        subnet mask
 */
bool ESP8266WiFiAPClass::softAPConfig(IPAddress local_ip, IPAddress gateway, IPAddress subnet) {        // LINE 179

//    1) LINE 180, Critical DEBUG_WIFI(), Correct DHCP address when left in
DEBUG_WIFI("[APConfig] local_ip: %s gateway: %s subnet: %s\n", local_ip.toString().c_str(), gateway.toString().c_str(), subnet.toString().c_str());

//    2) Erroneous DHCP address when commented out
//DEBUG_WIFI("[APConfig] local_ip: %s gateway: %s subnet: %s\n", local_ip.toString().c_str(), gateway.toString().c_str(), subnet.toString().c_str());

//    3) Replace line 180 variables with different ones, then change varlist size

//       a) Behaves like original line, OK when in, error when commented out
IPAddress ipa(10,0,0,1);
DEBUG_WIFI("[APConfig] dummy1: %s dummy2: %s dummy3: %s\n", ipa.toString().c_str(), ipa.toString().c_str(), //ipa.toString().c_str()  );

//       b) Remove (1) argument, OK when in, error when commented out
IPAddress ipa(10,0,0,1);
DEBUG_WIFI("[APConfig] dummy1: %s dummy2: %s\n", ipa.toString().c_str(), ipa.toString().c_str() );

//       c) Remove (2) arguments, erroneous address regardless whether present or not
IPAddress ipa(10,0,0,1);
DEBUG_WIFI("[APConfig] dummy1: %s\n", ipa.toString().c_str() );

//    4) Divide original line into (3) separate lines, then see if we get erroneous result as implied by (3c):
DEBUG_WIFI("[APConfig] local_ip: %s ", local_ip.toString().c_str() );
DEBUG_WIFI("gateway: %s ", gateway.toString().c_str() );
DEBUG_WIFI("subnet: %s\n", subnet.toString().c_str() );

// ** Yes: Erroneous DHCP addresses are now always present; may help to isolate the problem **
```

Lastly, inspection of ['wifi_softap_set_dhcps_lease()'][invaddr_fail] of the LWIP libraries indicated that some IP addresses may cause a 'fail' to be returned by softAPConfig(). To demonstrate this, set the sketch compiler variable 'AP_CFGERR' to a non-zero value and re-compile:

[invaddr_fail]: https://github.com/esp8266/Arduino/blob/2.3.0/tools/sdk/lwip/src/app/dhcpserver.c#L816#L817  "Inverted start/end address fail"

```
scandone
del if0
mode : softAP(5e:cf:7f:c6:da:a4)
bcn 0
del if1
usl
add if1
dhcp server start:(ip:192.168.4.1,mask:255.255.255.0,gw:192.168.4.1)
bcn 100
Configuring as AP+STA
Setting soft-AP configuration ... 
[APConfig] DHCP IP start: 192.168.4.204     << Addresses are inverted; (start - end) > 0x64, generates the 'fail'
[APConfig] DHCP IP end: 192.168.4.48
[APConfig] wifi_set_ip_info failed!
Failed!
Setting soft-AP ... 
bcn 0
del if1
usl
add if1
dhcp server start:(ip:192.168.4.105,mask:255.255.255.248,gw:192.168.4.105)
bcn 100
Ready
Soft-AP IP address = 192.168.4.105
```
An 'ipconfig /all' check showed correct DHCP addresses, a result comfirmed again with pings and a webserver root response:

```
Wireless LAN adapter Wi-Fi 3:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : 150Mbps Wireless 802.11bgn Nano USB Adapter
   ....
   IPv4 Address. . . . . . . . . . . : 192.168.4.106(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   ....
   Default Gateway . . . . . . . . . : 192.168.4.105
   DHCP Server . . . . . . . . . . . : 192.168.4.105
   ....
```

_This result, however, is now obtained regardless of debug mode status or comment state of the 1st DEBUG_WIFI line._

### Sketch

```cpp
/* NodeZm Test Module */

#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>

//---------------------------------------------------------------------------
// Globals

// Home network SSID and password
const char* home_ssid     = "HomeNetwork";
const char* home_password = "xxxxxxxx";

// Soft access point ssid and password
const char * apsf_ssid     = "NodeZm";
const char * apsf_password = "nodeaaaa";
const char * host          = "nodezm";

IPAddress ipnull( 0, 0, 0, 0 );

// Set AP_CFGERR to a non-zero value to see 'softAPConfig()' failure
#define  AP_CFGERR   0
#if (AP_CFGERR != 0)

// AP address which returns an APConfig error
   IPAddress local_IP(192,168,4,105);    // Hex:   C0.A8.04.69
   IPAddress gateway(192,168,4,105);     // ID/BC: 192.168.4.104 / 192.168.4.111
   IPAddress subnet(255,255,255,248);    // Range: 192.168.4.105 - 192.168.4.110

#else

// AP address something other than the default of 192.168.4.1
   IPAddress local_IP(192,168,4,9);      // Hex: C0.A8.04.09
   IPAddress gateway(192,168,4,9);       // ID/BC: 192.168.4.8 / 192.168.4.15
   IPAddress subnet(255,255,255,248);    // Range: 192.168.4.9 - 192.168.4.14

#endif

// Static station address to connect to home network with
IPAddress ipsta( 192,168,1,50 );         // Hex: C0.A8.01.32
IPAddress gwsta( 192,168,1,1 );          // ID/BC: 192.168.1.0 / 192.168.1.255
IPAddress substa( 255,255,255,0 );       // Range: 192.168.1.1 - 192.168.1.254

// Huzzah LED assignments
const int  LedBlu = 2, LedRed = 0, LedOff = 1, LedOn = 0;

ESP8266WebServer server(80);

//---------------------------------------------------------------------------
void handleRoot() {
  
	digitalWrite ( LedBlu, LedOn );
  String message = "Hello from Root\n\n";

	server.send ( 200, "text/html", message );
  digitalWrite ( LedBlu, LedOff );
  
} //handleRoot

//---------------------------------------------------------------------------
void setup ( void ) {
  
  pinMode(LedRed, OUTPUT);         // Red  LED used to indicate network connection 
  pinMode(LedBlu, OUTPUT);         // Blue LED used to indicate server activity
  digitalWrite(LedRed, LedOff);    // Red  LED off for AP connection, on for AP + STA
  digitalWrite(LedBlu, LedOff);    // Blue LED off, no server activity

  Serial.begin(115200);
  while (!Serial) delay(250);      // Wait until Arduino Serial Monitor opens
  Serial.println( " " );
  Serial.println( " " );

  Serial.setDebugOutput(true);     // ? Cryptic WIFI info

// DEBUG: Disable auto-connect, then disconnect WIFI
  WiFi.setAutoConnect( false );
  WiFi.setAutoReconnect( false );
  WiFi.disconnect( true );           // Wifi OFF
  WiFi.softAPdisconnect( false );    // Wifi already OFF  

  Serial.println( "Configuring as AP+STA" );
  WiFi.mode( WIFI_AP_STA );

  Serial.println("Setting soft-AP configuration ... ");
  Serial.println( WiFi.softAPConfig(local_IP, gateway, subnet) ? "Ready" : "Failed!");

  Serial.println("Setting soft-AP ... ");
  bool ok = WiFi.softAP( apsf_ssid, apsf_password );
  Serial.println( ok ? "Ready" : "Failed!" );

  Serial.print("Soft-AP IP address = ");
  Serial.println(WiFi.softAPIP());
  Serial.println( " " );
  
  Serial.print( "Waiting for STA connection... " );
  WiFi.config( ipsta, gwsta, substa );
  WiFi.begin( home_ssid, home_password );
  bool  conn;
  for ( int i = 0; i < 30; i++ )
     {
     delay(500); Serial.print( "." );
     conn = (WiFi.status() == WL_CONNECTED);
     if ( conn ) break;
     } //end-for
   Serial.println( "." );  // End 'dot' feedback

// Connection status
   IPAddress staIP;
   if ( conn )  {

//    Show connection success, get IP address
      digitalWrite(LedRed, LedOn ); 
      staIP = WiFi.localIP();
      Serial.print("STA IP address: ");
      Serial.println( staIP );  // Show IP we're to use 
   
   } else {

//    No network connect
      Serial.println ( "No STA connection " );

   } //end-else

// DEBUG: Set auto-connect flags
  WiFi.setAutoConnect( true );
  WiFi.setAutoReconnect( true );

// Connect handlers and start server
	server.on ( "/", handleRoot );
	server.begin();
	Serial.println ( "HTTP server started" );
 
} //setup

//---------------------------------------------------------------------------
void loop ( void ) {
	server.handleClient();
} //loop

//---------------------------------------------------------------------------
//---------------------------------------------------------------------------
```
