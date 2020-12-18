# go-eCharger API Specification (english version)

# Index

1. [Connection](#1-connection-)
   - [Rate Limiting](#rate-limiting)
2. [API: status](#2-api-status-)
   - [Request Path](#request-path)
   - [Return Format](#return-format)
   - [Parameter](#parameter)
3. [Commands](#3-commands-) 
   - [Set parameter](#set-parameter) 
   - [Path](#path) 
4. [Return Values](#4-return-values-) 
   - [Local WiFi / Hotspot](#local-wifi--hotspot) 
   - [Cloud: REST Api](#cloud-rest-api) 
5. [Cloud REST Api Workflow](#5-cloud-rest-api-workflow-) 
6. [Custom MQTT Server](#6-custom-mqtt-server-) 

# 1. Connection:

The go-eCharger offers two WLAN interfaces, one of which always serves as a mobile hotspot
and another that can connect to an existing WLAN network to establish an Internet connection.
The following connections are offered for the API:

The following connections are offered for the API:

| Connection         | Path                                                             |
| ------------------ | ---------------------------------------------------------------- |
| WiFi Hotspot       | http://192.168.4.1/                                              |
| WiFi local network | http://x.x.x.x/ The IP address is retrieved from the DHCP server |
| Cloud: REST Api    | https://api.go-e.co/                                             |
| Custom MQTT Server |                                                                  |

Authentication:

| Connection         | Authentication                                                                                     |
| ------------------ | -------------------------------------------------------------------------------------------------- |
| WiFi Hotspot       | None (Hotspot WPA key must be known)                                                               |
| WiFi local network | None (device must be in the same WLAN and the HTTP Api must be activated with the go-eCharger app) |
| Cloud: REST Api    | go-eCharger Cloud Token                                                                            |
| Custom MQTT Server | None                                                                                               |

### Rate Limiting

| Verbindung         | Limit                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WiFi Hotspot       | None (5 second delay recommended)                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| WiFi local network | None (5 second delay recommended)                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Cloud: REST Api    | **Fair use Limit** : 50MB per month, about 500'000 requests. In case<br> of planned exceeding, please contact go-e GmbH! <br><br> **/api_status** <br> **Fair use Limit**: 50MB per month, about 25'000 requests. In case of<br> planned exceeding, please contact go-e GmbH! |
|Custom MQTT Server  | None (5 second delay recommended)                                                                                               |

# 2. API: status:

Returns all relevant parameters as a JSON object.

Example (incomplete):

```
{"version":"B","rbc":"251","rbt":"2208867","car":"1","amp":"10","err":"0","ast"
:"0","alw":"1","stp":"0","cbl":"0","pha":"8","tmp":"30","dws":"0","dwo":"0","ad
i":"1","uby":"0","eto":"120","wst":"3","nrg":[2,0,0,235,0,0,0,0,0,0,0,0,0,0,0,0
],"fwv":"020-rc1","sse":"000000","wss":"goe","wke":"","wen":"1","tof":"101","td
s":"1","lbr":"255","aho":"2","afi":"8","ama":"32","al1":"11","al2":"12","al3":"
15","al4":"24","al5":"31","cid":"255","cch":"65535","cfi":"65280","lse":"0","us
t":"0","wak":"","r1x":"2","dto":"0","nmo":"0","eca":"0","ecr":"0","ecd":"0","ec
4":"0","ec5":"0","ec6":"0","ec7":"0","ec8":"0","ec9":"0","ec1":"0","rca":"","rc
r":"","rcd":"","rc4":"","rc5":"","rc6":"","rc7":"","rc8":"","rc9":"","rc1":"","
rna":"","rnm":"","rne":"","rn4":"","rn5":"","rn6":"","rn7":"","rn8":"","rn9":""
,"rn1":""}
```

### Request Path

| Connection         | Path                                                                                    |
| ------------------ | --------------------------------------------------------------------------------------- |
| WiFi Hotspot       | http://192.168.4.1/status                                                               |
| WiFi local network | http://x.x.x.x/status                                                                   |
| Cloud: REST Api    | https://api.go-e.co/api_status?token=TOKEN [&wait=0] <br>The wait parameter is optional |

### Return Format

| Connection         | Path                                                              |
| ------------------ | ----------------------------------------------------------------- |
| WiFi Hotspot       | Plain STATUS_OBJECT                                               |
| WiFi local network | Plain STATUS_OBJECT                                               |
| Cloud: REST Api    | `{"success":true,"age":AGE_IN_MILLISECONDS,"data":STATUS_OBJECT}` |

### Parameter

In addition to these parameters, other parameters may also be added without prior notice and depending on the type of connection.

**Explanation Format:** All parameters are sent in the JSON object as a string (in quotation
marks). Most of these parameters can be converted to an integer format. The data type
specified in the format shows the expected size. If the string is not converted to the specified
data type, a communication error should be displayed.

| Parameter                                                           | Format     | Explanation |
| ------------------------------------------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| version                                                             | String (1) | **JSON Format.**<br>"B": normal case <br>"C": When end-to-end encryption is enabled |
| rbc                                                                 | uint32_t   | **reboot_counter:** Counts the number of boot operations. Sent with end-to-end encryption as protection against replay attacks. |
| rbt                                                                 | uint32_t   | **reboot_timer:** Counts the milliseconds since the last boot. Sent with end-to-end encryption as protection against replay attacks.<br>Expires after 49 days, increasing the reboot_counter. |
| car                                                                 | uint8_t    | **Status PWM Signaling**<br>1: charging station ready, no vehicle<br>2: vehicle loads<br>3: Waiting for vehicle<br>4: Charge finished, vehicle still connected |
| amp                                                                 | uint8_t    | Ampere value for the PWM signaling in whole ampere of **6-32A** |
| amx                                                                 | uint8_t    | Ampere value for the PWM signaling in whole ampere of **6-32A**  Will not be written on flash but acts like you set amp instead. Only on the next reboot, the amp value will be restored to the last value set with amp. Recommended for PV charging |
| err                                                                 | uint8_t    | **error:**<br>1: RCCB (Residual Current Device)<br>3: PHASE (phase disturbance)<br>8: NO_GROUND (earthing detection)<br>10, default: INTERNAL (other) |
| ast                                                                 | uint8_t    | **access_state:** Access control.<br>0: open<br>1: RFID / App needed<br>2: electricity price / automatic |
| alw                                                                 | uint8_t    | **allow_charging:** PWM signal may be present<br>0: no<br>1: yes |
| stp                                                                 | uint8_t    | **stop_state:** Automatic shutdown<br>0: deactivated<br>2: switch off after kWh |
| cbl                                                                 | uint8_t    | **Typ2 Cable Ampere encoding**<br>13-32: Ampere Codierung<br>0: no cable |
| pha                                                                 | uint8_t    | **Phasen** before and after the contactor<br>binary flags: 0b00ABCDEF<br>A ... phase 3, in front of the contactor<br>B ... phase 2 in front of the contactor<br>C ... phase 1 in front of the contactor<br>D ... phase 3 after the contactor<br>E ... phase 2 after the contactor<br>F ... phase 1 after the contactor<br>Example: 0b00001000: Phase 1 is available<br>Example: 0b00111000: Phase1-3 is available |
| tmp                                                                 | uint8_t    | **Temperature** of the controller in °C |
| dws                                                                 | uint32_t   | **Charged energy** in deca-watt seconds<br>Example: 100'000 means, 1'000'000 Ws (= 277Wh = 0.277kWh)<br>were charged during this charging process. |
| dwo                                                                 | uint16_t   | **Abschaltwert** in 0.1kWh if stp==2, for dws parameter<br>Example: 105 for 10,5kWh<br>Charging station logic: if(dwo!=0 && dws/36000>=dwo)alw=0 |
| adi                                                                 | uint8_t    | **adapter_in:** Charging box is plugged in with adapter<br>0: NO_ADAPTER<br>1: 16A_ADAPTER |
| uby                                                                 | uint8_t    | **unlocked_by:** Number of the RFID card that has activated the<br>current charging process |
| eto                                                                 | uint32_t   | **energy_total:** Total charged energy in 0.1kWh<br>Example: 130 means 13kWh charged |
| wst                                                                 | uint8_t    | **wifi_state:** Wi-Fi connection status<br>3: connected<br>default: not connected                                            |
| nrg                                                                 | array[15]  | Array with values of the current and voltage sensor<br>nrg [0]: voltage on L1 in volts<br>nrg [1]: voltage on L2 in volts<br>nrg [2]: voltage on L3 in volts<br>nrg [3]: voltage to N in volts<br>nrg [4]: Ampere on L1 in 0.1A (123 equals 12.3A)<br>nrg [5]: Ampere on L2 in 0.1A<br>nrg [6]: Ampere on L3 in 0.1A<br>nrg [7]: power on L1 in 0.1kW (36 equals 3.6kW)<br>nrg [8]: power on L2 in 0.1kW<br>nrg [9]: power at L3 in 0.1kW<br>nrg [10]: power at N in 0.1kW<br>nrg [11]: Total power in 0.01kW (360 equals 3.6kW)<br>nrg [12]: power factor on L1 in%<br>nrg [13]: power factor on L2 in%<br>nrg [14]: power factor on L3 in%<br>nrg [15]: Power factor on N in%<br><br> App logic:<br>if(Math.floor(pha/8) ==1 &&<br>parseInt(nrg[3])>parseInt(nrg[0])){<br>nrg[0]=nrg[3]<br>nrg[7]=nrg[10]<br>nrg[12]=nrg[15]<br>} |
| fwv                                                                 | String     | **Firmware Version**<br>Example: "020-rc1" |
| sse                                                                 | String     | **Serial number** number formatted as %06d<br> Example: "000001" |
| wss                                                                 | String     | WiFi **SSID**<br> Example: "My home network" |
| wke                                                                 | String     | WiFi **Key**<br>Example: "**\*\*\*\***" for fwv after 020<br>Example: "password" for fwv before 020 |
| wen                                                                 | uint8_t    | **wifi_enabled:** Wi-Fi enabled<br>0: deactivated<br>1: activated |
| tof                                                                 | uint8_t    | **ime_offset:** Time zone in hours for internal battery-powered clock +100<br>Example: 101 is GMT +1 |
| tds                                                                 | uint8_t    | **Daylight saving time offset** (Summer time) in hours<br>Example: 1 for Central Europe |
| lbr                                                                 | uint8_t    | **LED brightness** from 0-255 <br>0: LED off<br>255: LED brightness maximum |
| aho                                                                 | uint8_t    | Minimum number of hours in which to load with "electricity price - automatic" <br>Example: 2 ("Car is full enough after 2 hours") |
| afi                                                                 | uint8_t    | Hour **(time)** in which with "electricity price - automatically" the charge must have lasted at least aho hours.<br>Example: 7 ("Done until 7:00, so before at least 2 hours loaded") |
| azo                                                                 | uint8_t    | Awattar price zone<br>0: Austria<br>1: Germany |
| ama                                                                 | uint8_t    | Absolute max. Ampere: Maximum value for ampere setting<br>Example: 20 (can not be set to more than 20A in the app) |
| al1                                                                 | uint8_t    | Ampere Level 1 for push button on the device.<br>6-32: Ampere level activated<br>0: level deactivated (is skipped) |
| al2                                                                 | uint8_t    | Ampere Level 2 for push button on the device.<br>Must be either 0 or >al1 |
| al3                                                                 | uint8_t    | Ampere Level 3 for push button on the device.<br>Must be either 0 or >al2 |
| al4                                                                 | uint8_t    | Ampere Level 4 for push button on the device.<br>Must be either 0 or >al3 |
| al5                                                                 | uint8_t    | Ampere Level 5 for push button on the device.<br>Must be either 0 or >al4 |
| cid                                                                 | uint24_t   | Color idle: **color value for standby** (no car plugged in) as a number<br>Example: parseInt ("# 00FFFF"): 65535 (blue / green, default) |
| cch                                                                 | uint24_t   | Color charging: **color value for charging active,** as a number<br>Example: parseInt ("# 0000FF"): 255 (blue, default) |
| cfi                                                                 | uint24_t   | Color idle: **color value completed for charging,** as a number<br>Example: parseInt ("# 00FF00"): 65280 (green, default) |
| lse                                                                 | uint8_t    | **led_save_energy:** Turn off the LED automatically after 10 seconds<br>0: Energy saving function deactivated<br>1: Energy saving function activated  |
| ust                                                                 | uint8_t    | **unlock_state:** Cable lock adjustment<br>0: lock as long as the car is plugged in<br>1: Automatically unlock after charging<br>2: Always leave the cable locked |
| wak                                                                 | String     | WiFi **Hotspot Password**<br>Example: "abdef0123456" |
| r1x                                                                 | uint8_t    | **Flags** <br>0b1: HTTP Api in the WLAN network activated (0: no, 1: yes)<br>0b10: End-to-end encryption enabled (0: no, 1: yes) |
| dto                                                                 | uint8_t    | **Remaining time** in milliseconds remaining on activation by<br>electricity prices<br>App logic:<br>if(json.car==1)message = "Zuerst Auto anstecken"<br>else message = "Restzeit: …" |
| nmo                                                                 | uint8_t    | **Norway mode** activated<br>0: deactivated (earthing detection activated)<br>1: activated (no earthing detection, intended only for IT grids) |
| eca <br>ecr<br>ecd<br>ec4<br>ec5<br>ec6<br>ec7<br>ec8<br>ec9<br>ec1 | uint32_t   | Charged **energy per RFID card** from 1-10<br>Example: eca == 1400: 140kWh charged on card 1<br> Example: ec7 == 1400: 140kWh charged on board 7<br>Example: ec1 == 1400: 140kWh charged on card 10 |
| rca<br>rcr<br>rcd<br>rc4<br>rc5<br>rc6<br>rc7<br>rc8<br>rc9<br>rc1  | String     | **RFID Card ID** from 1-10 as a string<br>Format and length: variable, depending on the version |
| rna<br>rnm<br>rne<br>rn4<br>rn5<br>rn6<br>rn7<br>rn8<br>rn9<br>rn1  | String     | **RFID Card Name** from 1-10<br>Maximum length: 10 characters |
| tme                                                                 | String     | **Current time**, formatted as ddmmyyhhmm<br>0104191236 corresponds to 01.04.2019 12:36 |
| sch                                                                 | String     | **Scheduler settings** (base64 encoded)<br>Functions for encode and decode are here:<br>https://gist.github.com/peterpoetzi/6cd2fad2a915a2498776912c5aa137a8<br> The settings can be set in this way:<br>r21=Math.floor(encode(1))<br>r31=Math.floor(encode(2))<br>r41=Math.floor(encode(3))<br>Direct setting of sch = is not supported |
| sdp                                                                 | uint8_t    | **Scheduler double press:** Activates charge after double pressing the<br>button if the load has just been interrupted by the scheduler<br>0: Function disabled<br> 1: Allow charge immediately |
| upd                                                                 | uint8_t    | **Update available** (only available if connected via go-e server)<br>0: no update available<br>1: Update available |
| cdi                                                                 | uint8_t    | **Cloud disabled**<br>0: cloud enabled<br>1: cloud disabled |
| loe                                                                 | uint8_t    | **Load balancing enabled**<br>0: load balancing disabled<br>1: Load balancing activated via cloud |
| lot                                                                 | uint8_t    | **Load balancing group total ampere** |
| lom                                                                 | uint8_t    | **Load balancing minimum amperage** |
| lop                                                                 | uint8_t    | **Lastmanagement priority** |
| log                                                                 | String     | **Lastmanagement group ID** |
| lon                                                                 | uint8_t    | **Lastmanagement:expected number of charging stations<br>(currently not supported)** |
| lof                                                                 | uint8_t    | **Load balancing fallback amperage** |
| loa                                                                 | uint8_t    | **Load balancing Ampere** (current permitted charging current)<br>is automatically controlled by the load balancing) |
| lch                                                                 | uint32_t   | **Load balancing: seconds since the last current flow while the<br>car is still plugged in**<br>0 when charging is in progress |
| mce                                                                 | uint8_t    | **MQTT custom enabled**<br>Connect to your own MQTT server<br>0: Function disabled<br>1: Function activated |
| mcs                                                                 | String(63) | **MQTT custom Server**<br>Hostname without protocol specification (z.B. test.mosquitto.org) |
| mcp                                                                 | uint16_t   | **MQTT custom Port**<br>i.e. 1883 |
| mcu                                                                 | String(16) | **MQTT custom Username** |
| mck                                                                 | String(16) | **MQTT custom key**<br>For MQTT authentication |
| mcc                                                                 | uint8_t    | **MQTT custom connected**<br>0: not connected<br>1: connected |

# 3. Commands:

The following parameters can only be read:

```
version rbc rbt car err cbl pha tmp dws adi uby eto wst nrg fwv sse eca ecr
ecd ec4 ec5 ec6 ec7 ec8 ec9 ec1 rca rcr rcd rc4 rc5 rc6 rc7 rc8 rc9 rc1
```

The following parameters can be set:

```
amp amx ast alw stp dwo wss wke wen tof tds lbr aho afi ama al1 al2 al3 al4 al5
cid cch cfi lse ust wak r1x dto nmo rna rnm rne rn4 rn5 rn6 rn7 rn8 rn9 rn1
```

### Set parameter

For all parameters that can be set, the format is for the command:

| Method | Payload                                                            |
| ------ | ------------------------------------------------------------------ |
| SET    | [param]=[value]<br>Example: amp=16<br>Example: wss=my home network |

### Path

| Connection            | Path                                                                                                 |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| WiFi Hotspot          | http://192.168.4.1/mqtt?payload=                                                                     |
| WiFi local network    | http://x.x.x.x/mqtt?payload=                                                                         |
| Cloud: REST Api       | https://api.go-e.co/api?token=TOKEN&payload=MESSAGE                                                  |

# 4. Return Values:

### Local WiFi / Hotspot

| Connection            | Response                                               |
| --------------------- | ------------------------------------------------------ |
| WiFi Hotspot          | Complete status JSON object with already changed value |
| WiFi local network    | Complete status JSON object with already changed value |

For every status request and every command, the status JSON object is returned. An<br>unsuccessful command can be recognized by the fact that the value in the status object has not changed.

### Cloud: REST Api

Responses for /api

| Condition                      | Response                                      |
| ------------------------------ | --------------------------------------------- |
| Token not specified            | `{"success":false,"error":"no token"}`        |
| Payload not specified          | `{"success":false,"error":"no payload"}`      |
| Token not found in database    | `{"success":false,"error":"wrong token"}`     |
| Rate limit exception           | `{"success":false,"error":"rate limiting"}`   |
| Success                        | `{"success":true,"payload":original_payload}` |

Return values for /api_status

| Condition                      | Response                                                          |
| ------------------------------ | ----------------------------------------------------------------- |
| Token not specified            | `{"success":false,"error":"no token"}`                            |
| Token not found in database    | `{"success":false,"error":"wrong token"}`                         |
| Rate limit exception           | `{"success":false,"error":"rate limiting"}`                       |
| Status not available           | `{"success":false,"error":"other"}`                               |
| Success                        | `{"success":true,"age":AGE_IN_MILLISECONDS,"data":STATUS_OBJECT}` |

Response time for /api_status

| Condition                      | Response time                                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| Last Status <10 seconds old    | ~300 milliseconds                                                                                                               |
| Last Status 10 seconds old     | **If wait=1:**<br>~300 bis ~3500 milliseconds<br><br>**If wait=0:**<br>~300 milliseconds<br><br>**Explanation:** If wait=1 **(default)** API Server sends ping to load box and waits up to 3 seconds for a new status object. If no new<br>status arrives after 3 seconds, the last received status will be sent. |
| Status nicht abrufbar          | < 1000 milliseconds                                                                                                                                                                                                                                                                                    |

# 5. Cloud REST Api Workflow:

Examples:

|Action |Set charging current to 16A|
|---|---|
|URL |https://api.go-e.co/api?payload=amp=16&token=__________|
|Response|`` {"success":true,"payload":"amp=16"}``|

|Action |Deactivate charging|
|---|---|
|URL| https://api.go-e.co/api?payload=alw=0&token=__________|
|Response |  ``{"success":true,"payload":"alw=0"}``|

|Action| Activate charging|
|---|---|
|URL| https://api.go-e.co/api?payload=alw=1&token=__________|
|Response |``{"success":true,"payload":"alw=1"}``|

|Action |Get Status|
|---|---|
|URL |https://api.go-e.co/api_status?token=__________ &wait=0|
|Response<br>(simplified)|``{"success":true,"age":1234,"data":{"version":"B", [...] ,<br>"car":"1","amp":"16","err":"0", [...] }}``|

# 6. Custom MQTT Server:

From firmware version 030 on it is possible to use a separate MQTT server in addition to the<br>go-e cloud.<br>
<br>
Commands are accepted via this topic:<br>**go-eCharger/000000/cmd/req**<br>Where 000000 must be replaced by the respective serial number.<br>
<br>
The status object is output every 5 seconds via the following topic:<br>**go-eCharger/000000/status**<br>
<br>
It is not necessary to activate the sending, the go-eCharger continuously sends data to /status.
