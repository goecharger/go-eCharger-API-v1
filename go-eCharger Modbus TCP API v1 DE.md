# GO-E Charger Modbus TCP/IP Dokumentation Deutsch

| Version | Date       | Author      | Description     |
| ------- | ---------- | ----------- | --------------- |
| 1.0     | 2020-10-01 | Peter Pötzi | Initial version |

# Index

* [Aktivierung](#activation)
* [Verbindung](#connection)
* [Modbus Register](#modbusregister)
* [Code Beispiele](#example)

   + [ Verbindung mit dem Charger herstellen ](#connect)
   + [ Daten über den Input Register auslesen ](#getinputregister)
   + [ Daten über den Holding Register auslesen ](#getholdingregister)
   + [ Daten über den Holding Register eintragen ](#setholdingregister)
   + [ Socket schließen ](#close)
   + [ Socket öffnen ](#open)

### Doku zu den verwendeten libraries

* [net](https://nodejs.org/api/net.html)
* [jsmodbus](https://www.npmjs.com/package/jsmodbus)

<a id="activation"></a>
## Aktivierung der API

Um die API zu aktivieren muss in der go-eCharger app unter Erweiterte Einstellungen der<br>
API Zugriff aktiviert werden. Nach der Aktivierung ist ein Neustart der Ladebox notwendig.<br>
Die API steht erst ab Firmware Version 0.40 zur Verfügung.

<a id="connection"></a>
## Verbindung

Modbus TCP wird über folgende Verbindungen angeboten:

| Verbindung | Port |
| ---------- | ---- |
| WLAN       | 502  |

Der go-e Charger hat die Geräte ID 1.

<a id="modbusregister"></a>
## Modbus Register

Die Register ID ist 0-Indiziert. Daher muss für manche Clients die Register ID um 1<br>
erhöht werden.

Register sind als Holding Register oder Input Register ausgeführt.

* Holding Register erlauben ein Lesen und Schreiben
* Input Register nur ein Lesen.

Bei Werten die über mehrere Register gehen sind alle 16-Bit Packete ein big-endian.
Der go-eCharger verwendet standardmäßig Big Endian (Word Swap).


| Register                               | Bezeichnung                 | Register Typ      | Datentyp              | Länge | Beschreibung                                                                                                                                                                                                                                                                                                                                               |
| -------------------------------------- | --------------------------- | ----------------- | --------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 100                                    | CAR_STATE                   | Input Register    | unsigned integer (16) | 1     | **Status PWM Signalisierung**<br>0: unbekannt, Ladestation defekt<br>1: Ladestation bereit, kein Fahrzeug<br>2: Fahrzeug lädt<br>3: Warte auf Fahrzeug<br>4: Ladung beendet, Fahrzeug noch<br>verbunden                                                                                                                                                    |
| 101                                    | PP_CABLE                    | Input Register    | unsigned integer (16) | 1     | Typ2 **Kabel Ampere codierung**<br>13-32: Ampere Codierung<br>0: kein Kabel                                                                                                                                                                                                                                                                                |
| 105 <br>106                            | FWV                         | Input Register    | ascii (4 byte)        | 2     | Firmware Version als ASCII                                                                                                                                                                                                                                                                                                                                 |
| 107                                    | ERROR                       | Input Register    | unsigned integer (16) | 1     | error:<br>1: RCCB (Fehlerstromschutzschalter)<br>3: PHASE (Phasenstörung)<br>8: NO_GROUND (Erdungserkennung)<br>10, default: INTERNAL (sonstiges)                                                                                                                                                                                                          |
| 108<br>109                             | VOLT_L1                     | Input Register    | unsigned integer (32) | 2     | Spannung auf L1 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 110 <br>111                            | VOLT_L2                     | Input Register    | unsigned integer (32) | 2     | Spannung auf L2 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 112 <br>113                            | VOLT_L3                     | Input Register    | unsigned integer (32) | 2     | Spannung auf L3 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 114<br>115                             | AMP_L1                      | Input Register    | unsigned integer (32) | 2     | Ampere auf L1 in 0.1A (123 entspricht 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 116<br>117                             | AMP_L2                      | Input Register    | unsigned integer (32) | 2     | Ampere auf L2 in 0.1A (123 entspricht 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 118 <br>119                            | AMP_L3                      | Input Register    | unsigned integer (32) | 2     | Ampere auf L3 in 0.1A (123 entspricht 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 120<br>121                             | POWER_TOTAL                 | Input Register    | unsigned integer (32) | 2     | Leistung gesamt in 0.01kW (360 entspricht 3, 6kW)                                                                                                                                                                                                                                                                                                           |
| 128 <br>129                            | ENERGY_TOTAL                | Input Register    | unsigned integer (32) | 2     | Gesamt geladene Energiemenge in 0.1kWh                                                                                                                                                                                                                                                                                                                     |
| 132<br>133                             | ENERGY_CHARGE               | Input Register    | unsigned integer (32) | 2     | **Geladene Energiemenge** in<br>Deka-Watt-Sekunden<br>Beispiel:100’000 bedeutet, 1’000’000<br>Ws (=277Wh = 0, 277kWh) wurden in<br>diesem Ladevorgang geladen.                                                                                                                                                                                              |
| 144 <br>145                            | VOLT_N                      | Input Register    | unsigned integer (32) | 2     | Spannung auf N in Volt                                                                                                                                                                                                                                                                                                                                     |
| 146 <br>147                            | POWER_L1                    | Input Register    | unsigned integer (32) | 2     | Leistung auf L1 in 0.1kW (36 entspricht 3, 6kW)                                                                                                                                                                                                                                                                                                             |
| 148<br>149                             | POWER_L2                    | Input Register    | unsigned integer (32) | 2     | Leistung auf L2 in 0.1kW (36 entspricht 3, 6kW)                                                                                                                                                                                                                                                                                                             |
| 150 <br>151                            | POWER_L3                    | Input Register    | unsigned integer (32) | 2     | Leistung auf L3 in 0.1kW (36 entspricht 3, 6kW)                                                                                                                                                                                                                                                                                                             |
| 152 <br>153                            | POWER_FACTOR_L1             | Input Register    | unsigned integer (32) | 2     | Leistungsfaktor auf L1 in %                                                                                                                                                                                                                                                                                                                                |
| 154<br>155                             | POWER_FACTOR_L2             | Input Register    | unsigned integer (32) | 2     | Leistungsfaktor auf L2 in %                                                                                                                                                                                                                                                                                                                                |
| 156<br>157                             | POWER_FACTOR_L3             | Input Register    | unsigned integer (32) | 2     | Leistungsfaktor auf L3 in %                                                                                                                                                                                                                                                                                                                                |
| 158 <br>159                            | POWER_FACTOR_N              | Input Register    | unsigned integer (32) | 2     | Leistungsfaktor auf N in %                                                                                                                                                                                                                                                                                                                                 |
| 200                                    | ALLOW                       | Holding Register  | unsigned integer (16) | 1     | **allow_charging:** PWM Signal darf<br>anliegen<br>0: nein<br>1: ja                                                                                                                                                                                                                                                                                        |
| 201                                    | ACCESS_STATE                | Holding Register  | unsigned integer (16) | 1     | **access_state**: Zugangskontrolle.<br>0: Offen<br>1: RFID / App benötigt<br>2: Strompreis / automatisch<br>3: Scheduler                                                                                                                                                                                                                                   |
| 202                                    | ADAPTER_INPUT               | Input Register    | unsigned integer (16) | 1     | **adapter_in**: Ladebox ist mit Adapter<br>angesteckt<br>0: NO_ADAPTER<br>1: 16A_ADAPTER                                                                                                                                                                                                                                                                   |
| 203                                    | UNLOCKED_BY                 | Input Register    | unsigned integer (16) | 1     | Nummer der RFID Karte, die den<br>jetzigen Ladevorgang freigeschalten<br>hat                                                                                                                                                                                                                                                                               |
| 204                                    | CABLE_LOCK_MODE             | Holding Register  | unsigned integer (16) | 1     | Kabelverriegelung Einstellung<br>0: Verriegeln solange Auto angesteckt<br>1: Nach Ladevorgang automatisch<br>entriegeln<br>2: Kabel immer verriegelt lassen                                                                                                                                                                                                |
| 205                                    | PHASES                      | Input Register    | unsigned integer (16) | 1     | **Phasen** vor und nach dem Schütz<br>binary flags: 0b00ABCDEF<br>A... phase 3, vor dem Schütz<br> B... phase 2 vor dem Schütz<br>C... phase 1 vor dem Schütz<br>D... phase 3 nach dem Schütz<br>E... phase 2 nach dem Schütz<br>F... phase 1 nach dem Schütz<br>pha 0b00001000: Phase 1 ist<br>vorhanden<br>pha 0b00111000: Phase1-3 ist<br>vorhanden<br> |
| 206                                    | LED_BRIGHTNESS              | Holding Register  | unsigned integer (16) | 1     | **LED Helligkeit** von 0-255<br>0: LED aus<br>255: LED Helligkeit maxima                                                                                                                                                                                                                                                                                   |
| 207                                    | LED_SAVE_ENERGY             | Holding Register  | unsigned integer (16) | 1     | **led_save_energy**: LED automatisch<br>nach 10 Sekunden abschalten<br>0: Energiesparfunktion deaktiviert<br>1: Energiesparfunktion aktiviert                                                                                                                                                                                                              |
| 208                                    | ELECTRICITY_PRICES_HOURS    | Holding Register  | unsigned integer (16) | 1     | Minimale **​Anzahl** ​von Stunden in der<br>mit "Strompreis - automatisch" geladen<br>werden muss<br>Beispiel: 2 ("Auto ist nach 2 Stundenvoll genug")                                                                                                                                                                                                     |
| 209                                    | ELECTRICITY_PRICES_FINISHED | Holding Register  | unsigned integer (16) | 1     | Stunde (**​Uhrzeit​**) in der mit "Strompreis<br>- automatisch" die Ladung mindestens<br>aho ​Stunden gedauert haben muss.<br>Beispiel: 7 ("Fertig bis 7:00, also davor<br>mindestens 2 Stunden geladen")                                                                                                                                                  |
| 210                                    | ELECTRICITY_PRICES_ZONE     | Holding Register  | unsigned integer (16) | 1     | Awattar Preiszone<br>0: Österreich<br>1: Deutschland                                                                                                                                                                                                                                                                                                       |
| 211                                    | AMPERE_MAX                  | Holding Register  | unsigned integer (16) | 1     | Absolute max. Ampere: Maximalwert<br>für Ampere Einstellung<br>Beispiel: 20 (Einstellung auf mehr als<br>20A in der App nicht möglich)                                                                                                                                                                                                                     |
| 212                                    | AMPERE_L1                   | Holding Register  | unsigned integer (16) | 1     | Ampere Level 1 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 213                                    | AMLERE_L2                   | Holding Register  | unsigned integer (16) | 1     | Ampere Level 2 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 214                                    | AMPERE_L3                   | Holding Register  | unsigned integer (16) | 1     | Ampere Level 3 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 215                                    | AMPERE_L4                   | Holding Register  | unsigned integer (16) | 1     | Ampere Level 4 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 216                                    | AMPERE_L5                   | Holding Register  | unsigned integer (16) | 1     | Ampere Level 5 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 217                                    | CLOUD_DISABLED              | Holding Register  | unsigned integer (16) | 1     | **Cloud disabled**<br>0: cloud enabled<br>1: cloud disabled                                                                                                                                                                                                                                                                                                |
| 218                                    | NORWAY_MODE                 | Holding Register  | unsigned integer (16) | 1     | **Norwegen-Modu**s​ aktiviert<br>0: deaktiviert (Erdungserkennung<br>aktiviert)<br>1: aktiviert (keine Erdungserkennung, <br>nur für IT-Netze gedacht)                                                                                                                                                                                                      |
| 299                                    | AMPERE_VOLATILE             | Holding Register  | unsigned integer (16) | 1     | Ampere Wert für die PWM<br>Signalisierung in ganzen Ampere von<br>**6-32A**<br><br>Wird nicht im EEPROM gespeichert<br>und wird beim nächsten Bootvorgang<br>auf den zuletzt im EEPROM<br>gespeicherten Wert **​zurückgesetzt​**.<br>Für Energieregelung                                                                                                   |
| 300                                    | AMPERE_EEPROM               | Holding Register  | unsigned integer (16) | 1     | Ampere Wert für die PWM<br>Signalisierung in ganzen Ampere von<br>**6-32A**<br><br>Wird im EEPROM ​gespeichert ​(max.<br>Schreibzyklen ca. 100.000)                                                                                                                                                                                                        |
| 301<br>302<br>303                      | MAC                         | Input Register    | unsigned integer (48) | 3     | MAC Adresse der WLAN Station, binär                                                                                                                                                                                                                                                                                                                        |
| 304<br>305<br>306<br>307<br>308<br>309 | SNR                         | Input Register    | ascii (12 byte)       | 6     | Seriennummer des go-eCharger, als<br>ASCII                                                                                                                                                                                                                                                                                                                 |
| 310<br>311<br>312<br>313<br>314 | HOSTNAME                    | Input Register    | ascii (10 byte)       | 6     | Hostname des go-eCharger, als ASCII                                                                                                                                                                                                                                                                                                                        |
| 315<br>316<br>317<br>318               | IP                          | Input Register    | ascii (8 byte)        | 4     | IP Adresse des go-eCharger, 1 Bytepro Register                                                                                                                                                                                                                                                                                                             |
| 319<br>320<br>321<br>322               | SUBNET                      | Input Register    | unsigned integer (64) | 4     | Subnetzmaske des go-eCharger, 1<br>Byte pro Register                                                                                                                                                                                                                                                                                                       |
| 323<br>324<br>325<br>326               | GATEWAY                     | Input Register    | ascii (4 byte)        | 4     | Gateway des go-eCharger, 1 Byte proRegister                                                                                                                                                                                                                                                                                                                |

<a id="example"></a>
<a id="connect"></a>

## Verbindung mit dem Charger herstellen

``` javascript
// clientG.js
const net = require("net"); // importiert net
const modbus = require("jsmodbus"); // importiert jsmodbus
const socket = new net.Socket(); // instanziert Socket
const client = new modbus.client.TCP(socket, 1); // instanziert eine Client TCP connection mit hilfe vom Socket und der unitId
const options = { // config für die connection
    port: 502, // port
    host: 'xxx.xxx.xxx.xxx' // ip
};
openSocket(socket, options)

socket.on("error", (err) => { // Wird aufgerufen, wenn der Computer einen Fehler beim aufrufen von der Verbindung hat
    console.log(err); // gibt den Fehler in der Konsole aus
})
```

<a id="getinputregister"></a>

## Daten über den Input Register auslesen

``` javascript
/**
 * @function
 * @name getInputRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
 * @param {number} count -  attribut von der Spalte "Länge" von der GOE-E Charger Modbus API
 * 

 * Diese Funktion gibt das JSON Objekt aus, welches vom Charger gesendet wirdc
 * 
 */
function getInputRegisters(socket, client, register, count) {
    socket.on("connect", function() { // Wird aufgerufen, wenn der Computer eine Verbindung herstellt 
        /**
         * @method 
         * @name readInputRegisters - read only Option vom Modbus (Von der GO-E Charger Modbus API -> Spalte Register Typ)
         * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
         * @param {number} count -  attribut von der Spalte "Länge" von der GOE-E Charger Modbus API
         */
        client.readInputRegisters(register, count).then(function(response) {
            console.log(response); // Gibt die response in der Konsole aus
        }).catch(function(err) {
            console.log(err); // Gibt den error in der Konsole aus
        });
    })
}
```

<a id="getholdingregister"></a>

## Daten über den Holding Register auslesen

``` javascript
/**
 * @function
 * @name getHoldingRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
 * @param {number} count -  attribut von der Spalte "Länge" von der GOE-E Charger Modbus API
 * 
 * Diese Funktion gibt das JSON Objekt aus, welches vom Charger gesendet wird
 * 
 */
function getHoldingRegisters(socket, client, register, count) {
    socket.on("connect", function() { // Wird aufgerufen, wenn der Computer eine Verbindung herstellt 
        /**
         * @method 
         * @name readHoldingRegisters - read only Option vom Modbus (Von der GO-E Charger Modbus API -> Spalte Register Typ)
         * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
         * @param {number} count -  attribut von der Spalte "Länge" von der GOE-E Charger Modbus API
         */
        client.readHoldingRegisters(register, count).then(function(response) {
            console.log(response); // Gibt die response in der Konsole aus
        }).catch(function(err) {
            console.log(err); // Gibt den error in der Konsole aus
        });
    })
}
```

<a id="setholdingregister"></a>

## Daten über den Holding Register eintragen

``` javascript
/**
 * @function
 * @name setHoldingRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
 * @param {number} value - attribut um dem gegebenen register einen neuen Wert zuzufügen
 * 
 * Diese Funktion setzt einen gegebenen wert auf eine gegebene register adresse
 * 
 */
function setHoldingRegisters(socket, client, register, value) {
    socket.on("connect", function() {
        /**
         * @method 
         * @name writeSingleRegister - read und write option von Mosbus
         * @param {number} register - attribut von der Spalte "Register" von der GO-E Charger Modbus API
         * @param {number} value - attribut um dem gegebenen register einen neuen Wert zuzufügen
         */
        client.writeSingleRegister(register, value).then(function(response) {
            console.log(response); // gibt den response aus
        }).catch(function(err) {
            console.log(err); // gibt den Fehler aus
            closeSocket(socket);
        })
    })
}
```

<a id="close"></a>

## Socket schließen

``` javascript
/**
 * @function
 * @name closeSocket
 * @param {object} socket 
 * 
 * Diese Funktion schließt den Socket
 * 
 */
function closeSocket(socket) {
    socket.end();
}
```

<a id="open"></a>

## Socket öffnen

``` javascript
/**
 * @function
 * @name openSocket
 * @param {object} socket 
 * @param {object} option - config für die Connection
 * 
 * Diese Funktion öffnet den Socket
 *  
 */
function openSocket(socket, option) {
    socket.connect(options); // Mit der gegebenen config mit dem Socket verbinden
}
```
