# go-eCharger MODBUS TCP API

| Version | Date       | Author      | Description     |
| ------- | ---------- | ----------- | --------------- |
| 1.0     | 2020-10-01 | Peter Pötzi | Initial version |

# Index

1. [Aktivierung](#1-aktivierung-der-api)
2. [Verbindung](2-verbindung)
3. [Modbus Register](3-modbus-register)
4. [Registertypen](#4-registertypen)

## 1. Aktivierung der API

Um die API zu aktivieren muss in der go-eCharger app unter Erweiterte Einstellungen der<br>
API Zugriff aktiviert werden. Nach der Aktivierung ist ein Neustart der Ladebox notwendig.<br>
Die API steht erst ab Firmware Version 0.40 zur Verfügung.

## 2. Verbindung

Modbus TCP wird über folgende Verbindungen angeboten:

| Verbindung | Port |
| ---------- | ---- |
| WLAN       | 502  |

## 3. Modbus Register

Bei Werten die über mehrere Register verteilt sind, enthält das jeweils niedrigere Register
den niederwertigen Teil (little endian).

| Register                               | Bezeichnung                 | Länge | Beschreibung                                                                                                                                                                                                                                                                                                                                               |
| -------------------------------------- | --------------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 100                                    | CAR_STATE                   | 1     | **Status PWM Signalisierung**<br>0: unbekannt, Ladestation defekt<br>1: Ladestation bereit, kein Fahrzeug<br>2: Fahrzeug lädt<br>3: Warte auf Fahrzeug<br>4: Ladung beendet, Fahrzeug noch<br>verbunden                                                                                                                                                    |
| 101                                    | PP_CABLE                    | 1     | Typ2 **Kabel Ampere codierung**<br>13-32: Ampere Codierung<br>0: kein Kabel                                                                                                                                                                                                                                                                                |
| 105 <br>106                            | FWV                         | 2     | Firmware Version als ASCII                                                                                                                                                                                                                                                                                                                                 |
| 107                                    | ERROR                       | 1     | error:<br>1: RCCB (Fehlerstromschutzschalter)<br>3: PHASE (Phasenstörung)<br>8: NO_GROUND (Erdungserkennung)<br>10, default: INTERNAL (sonstiges)                                                                                                                                                                                                          |
| 108<br>109                             | VOLT_L1                     | 2     | Spannung auf L1 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 110 <br>111                            | VOLT_L2                     | 2     | Spannung auf L2 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 112 <br>113                            | VOLT_L3                     | 2     | Spannung auf L3 in Volt                                                                                                                                                                                                                                                                                                                                    |
| 114<br>115                             | AMP_L1                      | 2     | Ampere auf L1 in 0.1A (123 entspricht 12,3A)                                                                                                                                                                                                                                                                                                               |
| 116<br>117                             | AMP_L2                      | 2     | Ampere auf L2 in 0.1A (123 entspricht 12,3A)                                                                                                                                                                                                                                                                                                               |
| 118 <br>119                            | AMP_L3                      | 2     | Ampere auf L3 in 0.1A (123 entspricht 12,3A)                                                                                                                                                                                                                                                                                                               |
| 120<br>121                             | POWER_TOTAL                 | 2     | Leistung gesamt in 0.01kW (360 entspricht 3,6kW)                                                                                                                                                                                                                                                                                                           |
| 128 <br>129                            | ENERGY_TOTAL                | 2     | Gesamt geladene Energiemenge in 0.1kWh                                                                                                                                                                                                                                                                                                                     |
| 132<br>133                             | ENERGY_CHARGE               | 2     | **Geladene Energiemenge** in<br>Deka-Watt-Sekunden<br>Beispiel:100’000 bedeutet, 1’000’000<br>Ws (=277Wh = 0,277kWh) wurden in<br>diesem Ladevorgang geladen.                                                                                                                                                                                              |
| 144 <br>145                            | VOLT_N                      | 2     | Spannung auf N in Volt                                                                                                                                                                                                                                                                                                                                     |
| 146 <br>147                            | POWER_L1                    | 2     | Leistung auf L1 in 0.1kW (36 entspricht 3,6kW)                                                                                                                                                                                                                                                                                                             |
| 148<br>149                             | POWER_L2                    | 2     | Leistung auf L2 in 0.1kW (36 entspricht 3,6kW)                                                                                                                                                                                                                                                                                                             |
| 150 <br>151                            | POWER_L3                    | 2     | Leistung auf L3 in 0.1kW (36 entspricht 3,6kW)                                                                                                                                                                                                                                                                                                             |
| 152 <br>153                            | POWER_FACTOR_L1             | 2     | Leistungsfaktor auf L1 in %                                                                                                                                                                                                                                                                                                                                |
| 154<br>155                             | POWER_FACTOR_L2             | 2     | Leistungsfaktor auf L2 in %                                                                                                                                                                                                                                                                                                                                |
| 156<br>157                             | POWER_FACTOR_L3             | 2     | Leistungsfaktor auf L3 in %                                                                                                                                                                                                                                                                                                                                |
| 158 <br>159                            | POWER_FACTOR_N              | 2     | Leistungsfaktor auf N in %                                                                                                                                                                                                                                                                                                                                 |
| 200                                    | ALLOW                       | 1     | **allow_charging:** PWM Signal darf<br>anliegen<br>0: nein<br>1: ja                                                                                                                                                                                                                                                                                        |
| 201                                    | ACCESS_STATE                | 1     | **access_state**: Zugangskontrolle.<br>0: Offen<br>1: RFID / App benötigt<br>2: Strompreis / automatisch<br>3: Scheduler                                                                                                                                                                                                                                   |
| 202                                    | ADAPTER_INPUT               | 1     | **adapter_in**: Ladebox ist mit Adapter<br>angesteckt<br>0: NO_ADAPTER<br>1: 16A_ADAPTER                                                                                                                                                                                                                                                                   |
| 203                                    | UNLOCKED_BY                 | 1     | Nummer der RFID Karte, die den<br>jetzigen Ladevorgang freigeschalten<br>hat                                                                                                                                                                                                                                                                               |
| 204                                    | CABLE_LOCK_MODE             | 1     | Kabelverriegelung Einstellung<br>0: Verriegeln solange Auto angesteckt<br>1: Nach Ladevorgang automatisch<br>entriegeln<br>2: Kabel immer verriegelt lassen                                                                                                                                                                                                |
| 205                                    | PHASES                      | 1     | **Phasen** vor und nach dem Schütz<br>binary flags: 0b00ABCDEF<br>A... phase 3, vor dem Schütz<br> B... phase 2 vor dem Schütz<br>C... phase 1 vor dem Schütz<br>D... phase 3 nach dem Schütz<br>E... phase 2 nach dem Schütz<br>F... phase 1 nach dem Schütz<br>pha 0b00001000: Phase 1 ist<br>vorhanden<br>pha 0b00111000: Phase1-3 ist<br>vorhanden<br> |
| 206                                    | LED_BRIGHTNESS              | 1     | **LED Helligkeit** von 0-255<br>0: LED aus<br>255: LED Helligkeit maxima                                                                                                                                                                                                                                                                                   |
| 207                                    | LED_SAVE_ENERGY             | 1     | **led_save_energy**: LED automatisch<br>nach 10 Sekunden abschalten<br>0: Energiesparfunktion deaktiviert<br>1: Energiesparfunktion aktiviert                                                                                                                                                                                                              |
| 208                                    | ELECTRICITY_PRICES_HOURS    | 1     | Minimale **​Anzahl** ​von Stunden in der<br>mit "Strompreis - automatisch" geladen<br>werden muss<br>Beispiel: 2 ("Auto ist nach 2 Stundenvoll genug")                                                                                                                                                                                                     |
| 209                                    | ELECTRICITY_PRICES_FINISHED | 1     | Stunde (**​Uhrzeit​**) in der mit "Strompreis<br>- automatisch" die Ladung mindestens<br>aho ​Stunden gedauert haben muss.<br>Beispiel: 7 ("Fertig bis 7:00, also davor<br>mindestens 2 Stunden geladen")                                                                                                                                                  |
| 210                                    | ELECTRICITY_PRICES_ZONE     | 1     | Awattar Preiszone<br>0: Österreich<br>1: Deutschland                                                                                                                                                                                                                                                                                                       |
| 211                                    | AMPERE_MAX                  | 1     | Absolute max. Ampere: Maximalwert<br>für Ampere Einstellung<br>Beispiel: 20 (Einstellung auf mehr als<br>20A in der App nicht möglich)                                                                                                                                                                                                                     |
| 212                                    | AMPERE_L1                   | 1     | Ampere Level 1 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 213                                    | AMLERE_L2                   | 1     | Ampere Level 2 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 214                                    | AMPERE_L3                   | 1     | Ampere Level 3 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 215                                    | AMPERE_L4                   | 1     | Ampere Level 4 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 216                                    | AMPERE_L5                   | 1     | Ampere Level 5 für Druckknopf am<br>Gerät.<br>6-32: Ampere Stufe aktiviert<br>0: Stufe deaktivert (wird übersprungen)                                                                                                                                                                                                                                      |
| 217                                    | CLOUD_DISABLED              | 1     | **Cloud disabled**<br>0: cloud enabled<br>1: cloud disabled                                                                                                                                                                                                                                                                                                |
| 218                                    | NORWAY_MODE                 | 1     | **Norwegen-Modu**s​ aktiviert<br>0: deaktiviert (Erdungserkennung<br>aktiviert)<br>1: aktiviert (keine Erdungserkennung,<br>nur für IT-Netze gedacht)                                                                                                                                                                                                      |
| 299                                    | AMPERE_VOLATILE             | 1     | Ampere Wert für die PWM<br>Signalisierung in ganzen Ampere von<br>**6-32A**<br><br>Wird nicht im EEPROM gespeichert<br>und wird beim nächsten Bootvorgang<br>auf den zuletzt im EEPROM<br>gespeicherten Wert **​zurückgesetzt​**.<br>Für Energieregelung                                                                                                   |
| 300                                    | AMPERE_EEPROM               | 1     | Ampere Wert für die PWM<br>Signalisierung in ganzen Ampere von<br>**6-32A**<br><br>Wird im EEPROM ​gespeichert ​(max.<br>Schreibzyklen ca. 100.000)                                                                                                                                                                                                        |
| 301<br>302<br>303                      | MAC                         | 3     | MAC Adresse der WLAN Station, binär                                                                                                                                                                                                                                                                                                                        |
| 304<br>305<br>306<br>307<br>308<br>309 | SNR                         | 6     | Seriennummer des go-eCharger, als<br>ASCII                                                                                                                                                                                                                                                                                                                 |
| 310<br>311<br>312<br>313<br>314<br>315 | HOSTNAME                    | 6     | Hostname des go-eCharger, als ASCII                                                                                                                                                                                                                                                                                                                        |
| 315<br>316<br>317<br>318               | IP                          | 4     | IP Adresse des go-eCharger, 1 Bytepro Register                                                                                                                                                                                                                                                                                                             |
| 319<br>320<br>321<br>322               | SUBNET                      | 4     | Subnetzmaske des go-eCharger, 1<br>Byte pro Register                                                                                                                                                                                                                                                                                                       |
| 323<br>324<br>325<br>326               | GATEWAY                     | 4     | Gateway des go-eCharger, 1 Byte proRegister                                                                                                                                                                                                                                                                                                                |

## 4. Registertypen

Register sind als Holding Register oder Input Register ausgeführt. Holding Register erlaubenein Lesen und Schreiben, Input Register nur ein Lesen.

| Register | Bezeichnung                     | Länge             |
| -------- | ------------------------------- | ----------------- |
| 100      | MOD_CAR_STATE                   | Input Register    |
| 101      | MOD_PP_CABLE                    | Input Register    |
| 105      | MOD_FWV                         | Input Register    |
| 107      | MOD_ERROR                       | Input Register    |
| 108      | MOD_VOLT_L1                     | Input Register    |
| 110      | MOD_VOLT_L2                     | Input Register    |
| 112      | MOD_VOLT_L3                     | Input Register    |
| 114      | MOD_AMP_L1                      | Input Register    |
| 116      | MOD_AMP_L2                      | Input Register    |
| 118      | MOD_AMP_L3                      | Input Register    |
| 126      | MOD_POWER_FACTOR_TOTAL          | Input Register    |
| 128      | MOD_ENERGY_TOTAL                | Input Register    |
| 132      | MOD_ENERGY_CHARGE               | Input Register    |
| 144      | MOD_VOLT_N                      | Input Register    |
| 146      | MOD_POWER_L1                    | Input Register    |
| 148      | MOD_POWER_L2                    | Input Register    |
| 150      | MOD_POWER_L3                    | Input Register    |
| 152      | MOD_POWER_FACTOR_L1             | Input Register    |
| 154      | MOD_POWER_FACTOR_L2             | Input Register    |
| 156      | MOD_POWER_FACTOR_L3             | Input Register    |
| 158      | MOD_POWER_FACTOR_N              | Input Register    |
| 200      | MOD_ALLOW                       | Holding Register  |
| 201      | MOD_ACCESS_STATE                | Holding Register  |
| 202      | MOD_ADAPTER_INPUT               | Input Register    |
| 203      | MOD_UNLOCKED_BY                 | Input Register    |
| 204      | MOD_CABLE_LOCK_MODE             | Holding Register  |
| 205      | MOD_PHASES                      | Input Register    |
| 206      | MOD_LED_BRIGHTNESS              | Holding Register  |
| 207      | MOD_LED_SAVE_ENERGY             | Holding Register  |
| 208      | MOD_ELECTRICITY_PRICES_HOURS    | Holding Register  |
| 209      | MOD_ELECTRICITY_PRICES_FINISHED | Holding Register  |
| 210      | MOD_ELECTRICITY_PRICES_ZONE     | Holding Register  |
| 211      | MOD_AMPERE_MAX                  | Holding Registerv |
| 212      | MOD_AMPERE_L1                   | Holding Register  |
| 213      | MOD_AMLERE_L2                   | Holding Register  |
| 214      | MOD_AMPERE_L3                   | Holding Register  |
| 215      | MOD_AMPERE_L4                   | Holding Register  |
| 216      | MOD_AMPERE_L5                   | Holding Register  |
| 217      | MOD_CLOUD_DISABLED              | Holding Register  |
| 218      | MOD_NORWAY_MODE                 | Holding Register  |
| 299      | MOD_AMPERE_VOLATILE             | Holding Register  |
| 300      | MOD_AMPERE_EEPROM               | Holding Register  |
| 301      | MOD_MACI                        | nput Register     |
| 304      | MOD_SNR                         | Input Register    |
| 310      | MOD_HOSTNAME                    | Input Register    |
| 315      | MOD_IP                          | Input Register    |
| 319      | MOD_SUBNET                      | Input Register    |
| 323      | MOD_GATEWAY                     | Input Register    |
