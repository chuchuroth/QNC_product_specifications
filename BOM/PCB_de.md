Nach Durchsicht der vier QNC-Dokumente ist der wichtigste Punkt: Die Spezifikationen definieren sehr klar die **Systemfunktion**, **Protokollgrenzen**, **Baseline-vs.-Extension-Scope**, **Safe-Mode-/Fault-Isolation**, **Konfigurationsgovernance** und **Umgebungsanforderungen**, aber **nicht** die für ein finales PCB nötigen Detaildaten wie exakte Stecker, Pinbelegung, Versorgungsspannung, Portanzahl pro Variante, Strombudget, Isolationsfestigkeit, Gehäuse/IP-Schutz und mechanische Randbedingungen. Deshalb ist der folgende Vorschlag ein **produktionsnaher Referenz-Designplan** auf Basis der Dokumente – mit expliziten Annahmen dort, wo die Unterlagen offen bleiben. [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [DPS](https://www.genspark.ai/api/files/s/MSkOUBCE)

---

## 1) Systeminterpretation und Designannahmen

Ich interpretiere QNC als **software-first Industrial Edge Gateway**, das in der Baseline **IO-Link, Modbus RTU, EtherNet/IP Adapter und Discrete Digital I/O** unterstützt, nordwärts **REST, WebSocket und strukturierte Logs** bereitstellt und für **FastDDS, Modbus TCP, CANopen, OPC UA, MQTT** extensionsfähig sein soll. Das System ist **nicht safety-rated**, muss aber Fault Isolation, Safe Mode, Konfigurationssignaturen, Rollback, Diagnosezugang und robuste industrielle Installation unterstützen. Für Hardware heißt das: Linux-fähige Rechenplattform, getrennte Feldschnittstellen, saubere Fehlercontainment-Zonen, servicefreundliche Diagnosepfade und genügend I/O-/Netzwerkspielraum für Baseline plus optionale Erweiterungen. [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### Empfohlene Hardware-Auslegung für Design Freeze A

| Annahme | Vorschlag |
|---|---|
| Eingangsversorgung | 24 VDC nominal, 18…36 VDC zulässig |
| Rechnerarchitektur | Linux-Compute-Modul + separater Echtzeit-/Supervisory-MCU |
| Ethernet | 2x Gigabit Ethernet, galvanisch zum Kabel entkoppelt über Magnetics |
| Modbus RTU | 2x isolierte RS-485 |
| IO-Link | 4x Master-Port |
| Discrete I/O | 8x DI + 8x DO, bankweise konfigurierbar |
| Extension-Ready | 1x CAN/CANopen, vorbereitet aber optional bestückt |
| Service | USB-C Service/Console, isolierter Debug-Header, Boot-Recovery |
| Montage | DIN-Schiene oder Panel, 0…50 °C Betrieb |
| Zielprodukt | Carrier-/Baseboard mit industriellem Compute-Modul |

Diese Annahmen sind konsistent mit dem dokumentierten Scope, den Umweltangaben 0…50 °C / Pollution Degree 2, der Forderung nach mindestens mehreren gleichzeitigen Devices/Clients, den Baseline-Protokollen und den Erweiterungspfaden. Sie ersetzen aber **nicht** die noch fehlenden Hardware-Festlegungen. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### Architekturentscheidung

Für die erste produktionsfähige Iteration empfehle ich **kein monolithisches Single-Board mit nacktem MPU + DDR**, sondern ein **Baseboard mit industriellem SoM**. Das reduziert DDR-/PMIC-/BGA-Risiko, verkürzt Layout- und SI-Zyklen und ist angesichts der fehlenden low-level Hardwaredaten der robusteste Weg zu EVT/DVT/PVT. Das passt auch zum Dokumentcharakter: Die QNC-Unterlagen spezifizieren primär System- und Interface-Verhalten, nicht eine fest vorgegebene CPU-Plattform. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 2) Schematic-Partitioning und Hierarchie

### Empfohlene Hierarchie der Schaltpläne

1. **Sheet 01 – Power Entry & Protection**  
   24-V-Eingang, Reverse Polarity, Hot-Swap/Inrush, TVS, EMI-Filter, Sicherungen, Versorgungsmonitoring.

2. **Sheet 02 – Compute Core**  
   Industrial SoM, Boot-Straps, Reset, TPM, RTC, Watchdog-Schnittstelle, Status-LEDs.

3. **Sheet 03 – Supervisory / RT MCU**  
   Zustandsüberwachung, Rail-Monitoring, Board-ID, Recovery, Brownout-Handling, Hardware-Watchdog.

4. **Sheet 04 – Northbound Ethernet**  
   2x Ethernet-Port, Magnetics, ESD, Shield/Chassis-Anbindung, optional Link/Act-LEDs.

5. **Sheet 05 – Southbound RS-485 / Modbus RTU**  
   2x isolierte RS-485-Kanäle, Terminierung, Biasing, Schutzbeschaltung, Diagnose.

6. **Sheet 06 – IO-Link Master**  
   4x Master-Port, Portstromversorgung L+, C/Q-Treiber, Kurzschluss-/Thermikschutz, Port-ID.

7. **Sheet 07 – Discrete Digital I/O**  
   8x DI, 8x DO, optional bankweise sourcing/sinking, Isolations-/Schutzebene, Statusdiagnose.

8. **Sheet 08 – CAN / CANopen Extension**  
   Optional bestückte CAN-Schnittstelle mit Bus-Schutz und Terminierung.

9. **Sheet 09 – Service, Debug, Recovery**  
   USB-C Device/Console, UART, SWD/JTAG, Recovery Header, Factory Config DIP.

10. **Sheet 10 – Security & Nonvolatile State**  
    TPM, sichere Identitäts-/Board-Serial-Ablage, optional Secure Element, Event Counter.

11. **Sheet 11 – Test & Production Support**  
    Boundary Testpunkte, Bed-of-Nails-Zugänge, Strommessshunts, Loopback-Optionen.

Diese Partitionierung spiegelt die in den Dokumenten geforderte Trennung zwischen Physical/Protocol Adaptation, Normalization, Control, Diagnostics und Service wider – nur auf Hardwareebene sauber abgebildet. [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### Funktionsblöcke auf Board-Ebene

- **Compute-Domäne**: Linux/QNC Runtime, REST/WebSocket, Logging, DDS optional.
- **Supervisor-Domäne**: deterministische Board-/Power-/Fault-Überwachung.
- **Feldbus-Domäne**: Modbus RTU, CANopen optional, IO-Link, DIO.
- **Service-Domäne**: Update, Diagnose, Recovery.
- **Power-Domäne**: getrennte Rails für Logik, Interface, isolierte Feldseite.

Damit ist Fault Containment auch physikalisch unterstützbar: Ein RS-485- oder IO-Link-Fehler darf nicht die Compute-Domäne oder andere Ports stören; das ist explizit im QNC-Fault-Modell angelegt. [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 3) Detaillierter BOM-Vorschlag

Wichtig: Das ist ein **empfohlener Referenz-BOM** zur Umsetzung der dokumentierten Funktion. Er ist **nicht** in den QNC-Dokumenten vorgegeben. Stecker, einige Leistungsstufen und die finale DIO-Topologie bleiben bis zur Interface-Freigabe offen.

### Kritische aktive Bauteile

| Block | Empfohlenes Teil | Funktion | Kritische Eckdaten | Sourcing-Hinweis |
|---|---|---|---|---|
| Compute SoM | **Toradex Verdin iMX8M Plus Quad 8GB WB IT** | Linux/Edge/DDS/REST/WebSocket | Industrial SoM, reduziert DDR-Risiko | Hersteller/Distributor prüfen, Langzeitverfügbarkeit bevorzugen [Toradex](https://www.toradex.com/computer-on-modules/verdin-arm-family/nxp-imx-8m-plus?srsltid=AfmBOoqKgcokkZBOHE2eLQYVikifUyZdY12uMnYiCEbgnh8hNAinrX0U) |
| Alternative SoM | **UCM-iMX8M-Plus** | Alternative Compute-Plattform | i.MX 8M Plus SoM | Nur als Backup-Source qualifizieren [CompuLab](https://www.compulab.com/products/computer-on-modules/ucm-imx8m-plus-nxp-i-mx-8m-plus-som-system-on-module-computer/) |
| Supervisor MCU | **STM32G474RET6** | Power-/Reset-/Watchdog-/Health-Supervisor | industrial grade, genug ADC/Timer/I/O | Second-source nicht direkt möglich, Lifecycle checken |
| TPM | **OPTIGA-TPM-SLB-9672-FW15** | Secure Boot / Schlüssel / Attestation | SPI TPM 2.0 | Sicherheits-BOM separat versionieren [Infineon](https://www.infineon.com/part/OPTIGA-TPM-SLB-9672-FW15) |
| IO-Link Master | **MAX14819ATM+** x2 | 4 IO-Link Master-Ports (2 Chips à 2 Ports) | integrierte Framer/L+ Control | Kritisches Langzeitteil, früh bevorraten [ADI](https://www.analog.com/en/products/max14819.html) |
| RS-485 | **ISO1410DWR** x2 | 2 isolierte Modbus-RTU Ports | 5 kVrms Isolation, robust EMC | Gute Wahl für Industrial Frontend [TI](https://www.ti.com/product/ISO1410) |
| CAN (Extension) | **TCAN1042HVDRQ1** | CAN/CANopen-ready | fault-protected CAN transceiver | Nur bestücken, wenn Extension freigegeben [TI](https://www.ti.com/product/TCAN1042HV-Q1/part-details/TCAN1042HVDRQ1) |
| Digital Input | **ISO1212DBQR** x4 | 8 isolierte 24-V-DI-Kanäle | 2 Kanäle/IC, industrial input receiver | IEC-61131-nahe Eingangstopologie [TI](https://www.ti.com/product/ISO1212) |
| Digital Output (Sourcing) | **TPS272C45CRHFR** x4 | 8 High-Side-DO-Kanäle | 36 V, 3 A, smart high-side | Nur wenn sourcing final freigegeben [TI](https://www.ti.com/product/TPS272C45) |
| DIO-Alternative | **MAX14900EAGM+CKT** x2 | flexible DIO-Topologie | pro Kanal flexibel, industrial DO/DIO | Prüfen, ob Logikmodell zum ICD passt [ADI](https://www.analog.com/en/products/max14900e.html) |
| Hot-Swap/Inrush | **LM5069MM-2/NOPB** | 24-V-Eingangsschutz | Hot-swap / inrush / fault cutoff | Pflicht für robustes Industrial Power-In [TI DS](https://www.ti.com/lit/ds/symlink/lm5069.pdf) |
| Haupt-Buck | **LM76005RNPR** | 24 V → 5 V Hauptversorgung | 60 V input tolerant, 5 A | Gute Robustheit am 24-V-Bus [TI](https://www.ti.com/product/LM76005/part-details/LM76005RNPR) |
| Isoliertes DC/DC | **NXJ2S0505MC-R13** x2…4 | isolierte Hilfsspannungen für Feldseite | 5 V / 2 W isoliert | Anzahl nach Kanalgruppierung [Murata](https://pim.murata.com/en-us/pim/details/?partNum=NXJ2S0505MC) |
| RTC | **RV-3028-C7** | Zeitstempel / Audit / Recovery | ultra-low power RTC | Optional, aber empfohlen |
| ESD Array | **TPD4E05U06QDRQ1** | ESD-Schutz für Service-/Datenleitungen | low-capacitance ESD | Für Ethernet separate Schutzstrategie nötig |
| TVS 24 V Eingang | **SMBJ33A** | Surge/Transient Clamp | 600 W Klasse | Hersteller nach Surge-Level auswählen |
| Common-Mode Chokes | Würth / TDK industrial families | Leitungsemissionen | differenziell je Schnittstelle | final per Pre-Compliance tunen |
| Ethernet Connector | **TBD nach Gehäuse** | RJ45 shielded oder M12 X-coded | mechanisch offen | Ohne Mechanikfreigabe nicht finalisierbar |
| Feldstecker | **TBD nach Mechanik** | IO-Link / RS-485 / DIO / CAN | Schraubklemme oder M8/M12 | mechanik- und IP-abhängig |

### Passiv-BOM-Regeln mit Toleranzen

| Passivgruppe | Empfehlung |
|---|---|
| 24-V-Pfad | 50 V oder 100 V Spannungsfestigkeit, X7R wo MLCC, Polymer/Elektrolyt an Bulk |
| 5-V-/3V3-Decoupling | 0,1 µF + 1 µF + 10 µF gestaffelt, X7R/X5R, 10 % ausreichend |
| Feedback-/Sense-Netzwerke | 1 % Standard, 0,1 % / 25 ppm für Spannungs- und Strommessung |
| Bus-Terminierungen | RS-485/CAN 120 Ω, 1 %, 0,25 W min.; Ethernet gemäß PHY/Magnetics-Appnote |
| RC-Filter Safety-Critical Diagnostics | 1 % / C0G oder stabile X7R, falls Triggergrenzen eng sind |
| Pull-ups/Pull-downs für Boot/Reset | 1 % oder 5 % je nach Strap-Anforderung |
| Shunts für Strommessung | 1 %, 1 W bis 3 W je nach Portstrom, 50 ppm empfehlenswert |

### Beschaffungsstrategie

- **A-Teile**: SoM, IO-Link, isolierte RS-485, TPM, Power-Front-End frühzeitig LTB/LTS-fähig absichern.
- **B-Teile**: Schutzbauteile, Isolationsmodule, DIO-Treiber dual sourcen.
- **C-Teile**: Passives auf 0402/0603/0805 standardisieren, Herstellerpool zulassen.
- **No-Go**: NCNR-Teile ohne Lebenszyklusfreigabe, nicht autorisierte Broker für Sicherheitsbauteile.

---

## 4) PCB-Stack-up

Da die Compute-Seite als SoM ausgeführt wird, reicht für das Carrier-Board ein **6-Lagen-Stack-up** mit kontrollierter Impedanz. Das ist der beste Kompromiss aus Kosten, Fertigbarkeit und SI/EMI-Sicherheit.

### Empfohlenes Stack-up

| Lage | Funktion | Kupfer | Hinweis |
|---|---|---:|---|
| L1 | Komponenten + kritische Signale | 35 µm | kurze Service-/Mgmt-/Diff-Pairs |
| L2 | durchgehende GND-Plane | 35 µm | Referenzlage für L1/L3 |
| L3 | Power + langsame Signale | 18 µm | 24 V / 5 V / 3V3 Inseln |
| L4 | kontrollierte Signale / Diff-Pairs | 18 µm | Ethernet/USB/MCU-Interconnect |
| L5 | durchgehende GND / Chassis-Kopplung | 35 µm | Schirm- und Rückstromführung |
| L6 | Komponenten + Feldsignale | 35 µm | Transceiver, Stecker, Schutz |

### Material und Fertigungsziel

- **FR-4, Tg ≥ 170 °C**
- Gesamtdicke **1,6 mm**
- Außenlagen **1 oz**, Innenlagen **0,5 oz**  
- Kontrollierte Impedanz-Coupons:
  - 50 Ω single-ended
  - 90 Ω diff für USB 2.0 falls genutzt
  - 100 Ω diff für Ethernet-/High-Speed-Paare
  - 120 Ω nominal für CAN/RS-485-Leitungsmodell auf dem Board nicht nötig, aber kontrollierte Symmetrie sinnvoll

Das passt zu einem industriellen Gateway mit begrenztem, aber vorhandenem High-Speed-Anteil und unterstützt saubere Rückstrompfade sowie EMV-Reserve. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 5) High-Speed- und Sensitive-Routing-Guidelines

### Allgemeine Routingregeln

- Keine Split-GND-Rückwege unter High-Speed-Signalen.
- Shield-/Chassis-Anbindung direkt am Stecker, **nicht** erst tief im Board.
- Feldschnittstellen an den Rand, Compute/TPM/Service in die Mitte.
- Isolationsbarrieren geometrisch sichtbar als Keep-out + Creepage-Kanal ausführen.
- Jede externe Leitung zuerst durch **ESD/Surge/Filter**, dann erst zum IC.

### Ethernet

- Falls das SoM PHYs integriert: MDI-Paare kurz, symmetrisch, ohne Stubs zum Magnetics-Modul.
- 100-Ω-diff, Paarlängendifferenz klein halten, gleiche Layer bevorzugen.
- Keine aggressiven 90°-Ecken; 45°/Bogen.
- Bob-Smith-/Shield-Netz exakt nach Magnetics-/Connector-Appnote.
- Chassis Shield nahe Port mit 360°-Schirmanbindung im Gehäusekonzept vorsehen.

### RS-485 / CAN

- Portnah terminieren; Bias nur einmal je Segment.
- TVS + ggf. Common-Mode Choke portnah.
- Isolatoren/isolierte Transceiver zwischen Logik- und Feldseite platzieren.
- Busleitungen nicht parallel zu aggressiven Schaltknoten des 24-V-Bucks führen.

### IO-Link

- Portkanäle identisch aufbauen, symmetrische Layout-Topologie.
- L+ Strompfad breit, thermisch entlastet, einzeln abgesichert.
- C/Q-Leitungen kurz, störungsarm, mit definierter Rückstromführung.
- Keine gemeinsamen Engstellen für mehrere Port-Rückströme.

### Sensitive Netze

- TPM-SPI, Reset, Boot-Straps, Supervisor-Signale von Feldkabeln fernhalten.
- ADC-/Sense-Leitungen Kelvin führen.
- Quarze/RTC nahe am IC, Guarding bei sehr empfindlichen Netzen.

Diese Regeln sind direkt aus dem dokumentierten Bedarf nach Fault Isolation, Diagnosierbarkeit, Update-Sicherheit und robusten Feldschnittstellen abgeleitet. [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 6) PDN-Design

### Empfohlener Power Tree

| Rail | Quelle | Typische Last | Bemerkung |
|---|---|---:|---|
| 24V_IN | Externer Industrieeingang | systemweit | geschützt, überwacht |
| 24V_FIELD | abgesichert aus 24V_IN | IO-Link/DIO/Feld | je Port/Bank abgesichert |
| 5V_MAIN | LM76005 aus 24V_IN | SoM + Logic | zentrale Logikschiene |
| 5V_ISO_A | isoliertes DC/DC | RS-485/CAN Gruppe A | Feldisolation |
| 5V_ISO_B | isoliertes DC/DC | RS-485/CAN Gruppe B | Feldisolation |
| 3V3_LOGIC | aus 5V_MAIN | MCU/TPM/Glue Logic | lokal gefiltert |
| 1V8_AUX | falls nötig | Level Shifter / Aux | nur wenn Peripherie nötig |

### PDN-Regeln

- 24-V-Eingang mit Hot-Swap, Reverse-Polarity-Schutz, TVS und Pi-Filter.
- SoM auf eigener sauberen 5-V-Insel mit Bulk-Puffer nahe Modul.
- Jeder Feldport mit lokaler Strombegrenzung oder Sicherung.
- Strommessung an 24V_IN, 5V_MAIN und mindestens einer Feldbank.
- Supervisor-MCU überwacht UV/OV, Inrush-Fault, Thermal-Fault und Port-Overcurrent.
- Rails sternförmig aus Power-Zentrum verteilen, nicht als Daisy Chain über die Boardfläche.

### Dimensionierungsvorschlag

- 24V_IN Budget: **2 A dauerhaft** als Startpunkt
- 5V_MAIN: **4–5 A**
- 24V_FIELD: je nach Portkonfiguration **0,5–1 A pro Bank**
- 3V3_LOGIC: **1–2 A**
- Reservemarge: **30 % elektrisch**, **20 % thermisch**

Das unterstützt die dokumentierten Ziele zu Recovery, Fault Handling, Diagnose und Mehrgerätebetrieb. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 7) Grounding- und Shielding-Strategie

Es sollte drei Bezugssysteme geben:

1. **DGND / Logic GND**  
   SoM, MCU, TPM, interne Logik.

2. **FGND / Field-Isolated GND**  
   isolierte RS-485/CAN-/ggf. DIO-Teilbereiche.

3. **CHASSIS / PE**  
   Schirme, Steckergehäuse, Surge-Ableitung.

### Regeln

- Kabelschirm immer zuerst auf **CHASSIS**, nicht direkt auf DGND.
- CHASSIS zu DGND über definierte RC-/HF-Kopplung, nicht flächig hart kurzschließen.
- Isolationsbarrieren klar getrennt mit ausreichender Creepage/Clearance.
- TVS-Rückstrom zur passenden Referenz führen; kein unkontrollierter Sprung auf DGND.
- LED-/Debug-/Service-GND nicht durch Feldstrompfade laufen lassen.

So wird das in den Dokumenten geforderte Fault Containment auch gegen leitungsgebundene Störungen wirksam unterstützt. [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 8) Thermalanalyse und Wärmeabfuhr

### Grobe Verlustabschätzung für Freeze A

| Block | Verlust grob |
|---|---:|
| SoM | 6…10 W |
| Power Front End + Buck | 1,5…3 W |
| IO-Link (4 Ports, ohne Last) | 1…2 W |
| RS-485 / CAN / DIO Logic | 1…2 W |
| Lastabhängige DO-Verluste | stark variabel, 0…6 W |

### Thermisches Konzept

- SoM mit **Heatspreader** an Gehäuse oder Metallträger koppeln.
- Unter Power-ICs, IO-Link-Treibern und High-Side-Switches dichte Thermal-Vias.
- Große Kupferflächen auf L1/L6 plus interne Kupferanbindung.
- Feldstecker nicht direkt neben SoM-Wärmequelle platzieren.
- Falls DIN-Rail-Gehäuse geschlossen: Konvektion vertikal unterstützen.

### Thermische Validierung

- CFD-light/Simulationsabschätzung vor Layout-Freeze.
- Messung bei 24 V, Volllast, 50 °C ambient.
- Hot-Spot-Limit intern < 105 °C für passive Magnetics/ESD-Cluster, < 125 °C Junction für aktive Leistungsteile.
- Worst Case: alle DIO-High-Side-Kanäle aktiv + DDS/Ethernet Traffic + IO-Link Telemetrie.

Die Temperaturspanne 0…50 °C ist in den QNC-Unterlagen genannt; das Board sollte mit mindestens 10 °C Margin ausgelegt werden. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M)

---

## 9) EMI/EMC-Compliance

Die Dokumente nennen keine konkreten Normen. Für ein industrielles Gateway würde ich deshalb folgende **Zielnormen** ansetzen:

- **EN 55032 / CISPR 32 Class A**
- **EN 61000-6-2** (Industrial Immunity)
- **IEC 61000-4-2** ESD
- **IEC 61000-4-4** EFT
- **IEC 61000-4-5** Surge
- **IEC 61000-4-6** Conducted RF Immunity

### Designmaßnahmen

- TVS an allen externen Leitungen.
- Common-Mode Chokes dort, wo Pre-Compliance es rechtfertigt.
- Schaltregler mit kleinem Hot Loop, Shield-GND-Rückführung und Snubber-Footprint.
- Getrennte Feld-/Logikzonen.
- Gehäuse-/Schirmkonzept früh mit Mechanik koppeln.
- Taktquellen kurz und sauber referenziert.
- Optional Ferrit-Beads an Domain-Übergängen, aber nicht als Ersatz für saubere PDN-Topologie.

### EMC-Testplan

- Erst **Pre-Compliance** auf nacktem Board und im Zielgehäuse.
- Danach gezielte Tuning-Runde für CMC, RC-Snubber, Shield-Bonding.
- Kritisch: Ethernet-Emissionen, 24-V-Eingang leitungsgebunden, IO-Link/DIO-Burstfestigkeit.

Da QNC sichere Kommunikation, Diagnose und Verfügbarkeit fordert, ist ein EMV-resistentes Frontend nicht optional, sondern zentral. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 10) DFM/DFA und Fertigungshinweise

### DFM

- Ziel: **IPC Class 2** für Serienprodukt; Class 3 nur falls Kunden-/Branchenvorgabe kommt.
- Min. Design Rule als Startpunkt: **6/6 mil**, Laser-Vias nur wenn SoM-Carrier-Fanout es fordert.
- Via-in-pad vermeiden, außer SoM-Hersteller fordert es.
- 1,6 mm Board, definierte Kupferbalance, Warpage-Kontrolle.
- Fiducials global + lokal.
- Testcoupon für Impedanz.
- Lötstoppmasken-Reliefs für feine QFNs optimieren.

### DFA

- Stecker nur an Außenkante.
- Service-/Recovery-Anschlüsse ohne Demontage des Gesamtsystems erreichbar machen.
- LEDs von vorne lesbar.
- Einbaurichtung für DIN-Rail und Panel gleich mitdenken.
- Schraubklemmen oder M12 nur mit genügend Werkzeugabstand.

### Fabrikationshinweise

- Conformal Coating optional; bei Pollution Degree 2 meist nicht zwingend, aber kundenspezifisch offen.
- Kein Underfill ohne echten Bedarf.
- Selektive Lötschritte vermeiden, wenn nicht mechanisch zwingend.
- Für alle Schutzbauteile Alternativfootprints vorsehen, wo EMC-Risiko hoch ist.

Die Dokumente geben Manufacturing-Standards nicht vor; deshalb sind diese Punkte eine pragmatische Industrial-DFM-Basis. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 11) Testpunkte, Debugging und Validierung

### Pflicht-Testpunkte

- 24V_IN, 24V_FIELD, 5V_MAIN, 3V3_LOGIC, alle isolierten 5-V-Rails
- Reset, Boot Mode, Watchdog Kick, Fault Summary
- RS-485 A/B je Kanal
- CANH/CANL
- IO-Link C/Q + L+ je Port
- DIO-Bank Inputs/Outputs
- UART Console, SWD/JTAG, TPM SPI Header optional nicht bestückt
- Shield-to-Chassis Messpunkt

### Debugging-Strategie

- Supervisor-MCU als **Board Health Monitor** mit eigenem Fault Log.
- Linux-Console separat von Applikationsnetz.
- Recovery-Boot ohne Spezialjig möglich.
- Portweise Enable/Disable per Software + Hard-Mute-Pfad.
- Strommessung pro Bank für schnelle Fault-Lokalisierung.

### Validierungsplan

**EVT**
- Power-up, Brownout, Inrush, ESD-Basistests
- Linux Boot, SoM Health, Supervisor Handshake
- Port Enumeration und Grundkommunikation

**DVT**
- IO-Link Master Tests
- Modbus RTU Langzeittest
- EtherNet/IP Durchsatz / Link Recovery
- Safe Mode Reaktionskette
- Rollback/Update Recovery
- Security: Signatur-/TPM-/Boot-Pfade

**PVT**
- Fertigungstests
- Temperaturzyklen
- 72h Burn-In
- Pre-Compliance + Regression

### QNC-spezifische Abnahmekriterien

- Kritischer Fault → **Safe Mode ≤ 1 s**
- Restart → betriebsbereit **≤ 30 s**
- REST-to-device Latenz im Zielpfad **≤ 50 ms**
- DDS-/Extension-Faults beeinflussen Baseline-Southbound nicht

Diese Kriterien leiten sich direkt aus den dokumentierten NFR-/Fault-/Lifecycle-Anforderungen ab. [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 12) Risiken, Trade-offs und noch fehlende Informationen

### Größte Projektrisiken

| Risiko | Auswirkung | Empfehlung |
|---|---|---|
| Fehlende Stecker-/Pinout-Freigabe | PCB kann nicht finalisiert werden | Mechanik + ICD-Hardware-Anhang nachziehen |
| Unklare Eingangsspannung / Strombudget | Power-Design evtl. falsch dimensioniert | 24-V-Systembudget schriftlich freigeben |
| DIO „sourcing/sinking“ nicht präzise definiert | falsche Ausgangstopologie | bankweise oder kanalweise Konfiguration sofort entscheiden |
| Unklare Isolationsanforderung | Sicherheits-/EMV-/Kostenrisiko | min. Working Voltage + HiPot-Ziel festlegen |
| Unklare Portanzahl | Flächen- und Thermikrisiko | Produktvariantenmatrix definieren |
| Unklarer Steckerstandard (RJ45 vs M12, Klemme vs Rundstecker) | Rework des Mechanik-/EMV-Designs | früh fixieren |
| FastDDS-Portsharing ungeklärt | Netzwerk-Topologie unklar | dedizierter Port vs VLAN vs shared NIC entscheiden |

### Wichtige Trade-offs

- **SoM vs. diskreter MPU**  
  SoM ist schneller, risikoärmer, teurer pro Stück; diskret ist günstiger in Volumen, aber deutlich riskanter in NRE und SI.

- **Vollständige Isolation vs. Teilisolation**  
  Mehr Isolation verbessert Robustheit/Fault Containment, kostet aber Fläche, BOM und Wärme.

- **2 Ethernet vs. 3 Ethernet Ports**  
  2 Ports reichen für Baseline meist aus; 3 Ports entkoppeln DDS/Service/Feld besser, erhöhen aber Kosten und Gehäusekomplexität.

- **4 IO-Link vs. 8 IO-Link Ports**  
  4 Ports sind realistisch für Freeze A; 8 Ports treiben Strom, Wärme und PCB-Fläche stark hoch.

- **RJ45 vs. M12**  
  RJ45 günstiger und labortauglich; M12 robuster industriell, aber mechanisch und kostenmäßig anspruchsvoller.

### Fehlende Informationen für Finalisierung

Bevor die Platine freigegeben werden kann, fehlen aus meiner Sicht mindestens diese Punkte:

1. Exakte **Versorgungsspezifikation**: nominal/min/max, Brownout, Transienten  
2. Endgültige **Steckerfamilien** und **Pinouts**  
3. Ziel-**Portanzahl** je Produktvariante  
4. Erforderliche **Isolationsfestigkeit** und Creepage/Clearance-Ziele  
5. Mechanische Daten: Board-Abmessung, Befestigung, Gehäuse, IP-Ziel  
6. Regulatorisches Zielset: CE/FCC/UL/UKCA, Industrie-EMV-Normen  
7. DIO-Elektrik: Pegel, Lasten, sourcing/sinking-Konzept, Schutzklasse  
8. Ethernet-Topologie: dedizierte Ports für northbound/southbound/DDS ja oder nein  
9. Servicekonzept: lokaler USB-Service, nur Ethernet, oder beides  
10. Fertigungsziel: IPC-Klasse, Coating, Traceability, Serienvolumen

Diese Lücken sind keine Nebensache; sie sind genau die Punkte, die in den QNC-Dokumenten bewusst außerhalb des Hardwaredetail-Scope liegen. [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [DPS](https://www.genspark.ai/api/files/s/MSkOUBCE) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 13) Klare Empfehlung

Wenn ich dieses Projekt in die Produktion führen müsste, würde ich den nächsten Schritt so setzen:

- **Architektur fixieren**: SoM-basiertes Carrier-Board bestätigen  
- **Open Points schließen**: Versorgung, Stecker, Isolationsziel, DIO-Topologie  
- **Design Freeze A** erstellen: 6-Lagen Carrier, 2x GbE, 2x RS-485, 4x IO-Link, 8xDI/8xDO, optional CAN  
- **EVT-Board** mit EMC-/Thermik-Reserve auslegen  
- danach **DVT** mit finalem Gehäuse und Pre-Compliance

---
---
# Komplette Schaltplan-Hierarchie für das QNC-Design

**CAD-taugliche, vollständige Schaltplan-Hierarchie** 

mit **Top-Level-Blockstruktur**, **vorgeschlagenen Blattgrenzen**, **hierarchischen Netznamen**, **Interfaces zwischen den Sheets** und **Signalrichtung pro Blatt**. Wo der Plan bewusst offen bleibt, friere ich die Logik-Schnittstellen ein und lasse nur Steckverbinder-/Mechanikdetails als spätere Freeze-Punkte offen. Das passt zu den Annahmen im Dokument: 24-V-Industrieversorgung, SoM + Supervisor-MCU, 2× GbE, 2× isoliertes RS-485, 4× IO-Link Master, 8× DI + 8× DO, optional CAN, Service/Recovery, TPM/RTC und klar getrennte Fault-Containment-Domänen. 

---

## 1. Hierarchische Gesamtstruktur

```text
00_TOP_QNC_GATEWAY
├── 01_PWR_ENTRY_PROTECTION
├── 02_COMPUTE_CORE
├── 03_SUPERVISOR_MCU
├── 04_ETHERNET_PORTS
├── 05_RS485_MODBUS_RTU
├── 06_IOLINK_MASTER_4PORT
├── 07_DISCRETE_IO_8DI_8DO
├── 08_CAN_CANOPEN_EXT
├── 09_SERVICE_DEBUG_RECOVERY
├── 10_SECURITY_NVM_RTC
└── 11_TEST_PRODUCTION_SUPPORT
```

### Empfohlene hierarchische Sheet-Namen in CAD

| Blatt | Sheet-Name | Funktion |
|---|---|---|
| 00 | `TOP_QNC_GATEWAY` | Root-Sheet mit allen Hierarchical Ports |
| 01 | `PWR_ENTRY_PROTECTION` | 24-V-Eingang, Schutz, Sequencing, Hauptwandler |
| 02 | `COMPUTE_CORE` | SoM, Boot, Status, Host-I/O |
| 03 | `SUPERVISOR_MCU` | Power-Monitoring, Reset, Watchdog, Safe-Mode |
| 04 | `ETHERNET_PORTS` | 2× Northbound Ethernet |
| 05 | `RS485_MODBUS_RTU` | 2× isoliertes RS-485 |
| 06 | `IOLINK_MASTER_4PORT` | 4× IO-Link Master |
| 07 | `DISCRETE_IO_8DI_8DO` | 8 Eingänge, 8 Ausgänge |
| 08 | `CAN_CANOPEN_EXT` | optionales CAN/CANopen |
| 09 | `SERVICE_DEBUG_RECOVERY` | USB-C, UART, SWD/JTAG, Recovery |
| 10 | `SECURITY_NVM_RTC` | TPM, EEPROM, RTC, Board Identity |
| 11 | `TEST_PRODUCTION_SUPPORT` | Testpunkte, Loopbacks, Produktionshilfen |

Diese Blattaufteilung folgt direkt der im PCB-Plan vorgeschlagenen funktionalen Partitionierung in Power-, Compute-, Supervisor-, Feldbus-, Service- und Security-Domänen. 

---

## 2. Top-Level-Blockdiagramm

```text
                 +----------------------------------+
24V_IN / PE ---> | 01 POWER ENTRY & PROTECTION      |
                 | TVS / RP / HotSwap / EMI / Buck  |
                 +--+-----------+---------+---------+
                    |           |         |
                    |           |         +--> 24V_FIELD_A/B/C
                    |           +------------> 5V_ISO_A/B
                    +------------------------> 5V_MAIN / 3V3_LOGIC
                                  |
                                  v
                    +------------------------------+
                    | 03 SUPERVISOR MCU            |
                    | rail monitor / reset / WDOG  |
                    +----+-------------+-----------+
                         |             |
                         |             +--> enables, safe mode, reset tree
                         |
                         v
              +------------------------------+
              | 02 COMPUTE CORE (SoM)        |
              | Linux runtime / northbound   |
              +--+---------+--------+--------+
                 |         |        | \
                 |         |        |  \
                 v         v        v   v
        +-----------+ +---------+ +---------+ +------------------+
        |04 ETH x2  | |05 RS485 | |06 IOLink| |07 DIO 8DI / 8DO  |
        +-----------+ +---------+ +---------+ +------------------+
                 \         |            |              /
                  \        |            |             /
                   \       v            v            /
                    \   +---------------------------+
                     -->| 08 CAN EXT (optional)    |
                        +---------------------------+

              +------------------+    +------------------+
              |09 Service/Debug  |    |10 Security/NVM   |
              +------------------+    +------------------+

                        +---------------------------+
                        |11 Test / Production       |
                        +---------------------------+
```

---

## 3. Globale Netznamen-Konvention

Damit die Schaltplanhierarchie sauber bleibt, würde ich die Netze so normieren:

### Power / Ground
- `24V_IN_RAW`
- `24V_IN_PROT`
- `24V_FIELD_A`
- `24V_FIELD_B`
- `24V_FIELD_C`
- `5V_MAIN`
- `5V_ISO_A`
- `5V_ISO_B`
- `3V3_LOGIC`
- `1V8_AUX`
- `VBAT_RTC`
- `DGND`
- `FGND_A`
- `FGND_B`
- `CHASSIS`

### Reset / Supervision / Safety
- `PWR_GOOD`
- `SYS_RESET_N`
- `SOM_RESET_N`
- `MCU_RESET_N`
- `SAFE_MODE_REQ`
- `SAFE_MODE_ACK`
- `WDOG_KICK`
- `WDOG_FAIL_N`
- `BOOT_RECOVERY_N`
- `FORCE_RECOVERY_N`
- `24V_UV_WARN`
- `24V_OV_WARN`
- `PWR_FAULT_N`
- `THERM_WARN_N`

### Management / Service
- `I2C_SEC_SCL`, `I2C_SEC_SDA`
- `I2C_MGMT_SCL`, `I2C_MGMT_SDA`
- `SPI_TPM_SCLK`, `SPI_TPM_MOSI`, `SPI_TPM_MISO`, `SPI_TPM_CS_N`
- `UART_DBG_TX`, `UART_DBG_RX`
- `USB0_D_P`, `USB0_D_N`
- `USB0_CC1`, `USB0_CC2`
- `SWDIO`, `SWCLK`, `NRST_MCU`

### Ethernet
- `ETH1_MDI0_P/N ... ETH1_MDI3_P/N`
- `ETH2_MDI0_P/N ... ETH2_MDI3_P/N`
- `ETH1_LED_LINK_N`, `ETH1_LED_ACT_N`
- `ETH2_LED_LINK_N`, `ETH2_LED_ACT_N`
- `ETH1_SHIELD`, `ETH2_SHIELD`

### RS-485
- `RS485A_TXD`, `RS485A_RXD`, `RS485A_DE`, `RS485A_RE_N`, `RS485A_TERM_EN`
- `RS485B_TXD`, `RS485B_RXD`, `RS485B_DE`, `RS485B_RE_N`, `RS485B_TERM_EN`
- `RS485A_A`, `RS485A_B`
- `RS485B_A`, `RS485B_B`

### IO-Link
- `IOL_SPI_SCLK`, `IOL_SPI_MOSI`, `IOL_SPI_MISO`
- `IOL_CS0_N`, `IOL_CS1_N`
- `IOL_INT0_N`, `IOL_INT1_N`
- `IOL_RESET_N`
- `IOL0_CQ`, `IOL1_CQ`, `IOL2_CQ`, `IOL3_CQ`
- `IOL0_LPLUS ... IOL3_LPLUS`
- `IOL0_OC_N ... IOL3_OC_N`
- `IOL0_TSD_N ... IOL3_TSD_N`

### DIO
- `DI_SENSE[7:0]`
- `DO_CMD[7:0]`
- `DO_FLT_N[7:0]`
- `DO_EN[7:0]`
- `DIO_BANK0_MODE`, `DIO_BANK1_MODE`
- Extern logisch: `DI0_IN ... DI7_IN`, `DO0_OUT ... DO7_OUT`

### CAN
- `CAN1_TXD`, `CAN1_RXD`, `CAN1_STB_N`, `CAN1_TERM_EN`
- `CANH`, `CANL`
- `CAN1_FAULT_N`

### Test / Manufacturing
- `TP_24V_IN_RAW`, `TP_5V_MAIN`, `TP_3V3_LOGIC`, `TP_SYS_RESET_N`
- `TP_RS485A_A`, `TP_RS485A_B`, `TP_CANH`, `TP_CANL`
- `TP_ETH1_MDI0_P/N`, usw.
- `LB_RS485A_EN`, `LB_CAN1_EN`, `LB_IOL_SIM_EN`

---

## 4. Richtungsdefinition

Zur Eindeutigkeit gilt in allen Tabellen:

- `IN` = kommt **in dieses Sheet hinein**
- `OUT` = geht **von diesem Sheet heraus**
- `BIDI` = bidirektional
- `MON` = Mess-/Statussignal, funktional nicht treibend
- `PWR` = Energieversorgung
- `REF` = Masse/Schirm/Referenz

---

# 5. Blattdefinitionen im Detail

---

## 01 – [PWR_ENTRY_PROTECTION]

### Blockdiagramm

```text
J1_24V_IN / PE
   -> Fuse / Reverse Polarity / TVS
   -> Hot-Swap / Inrush / EMI PI Filter
   -> 24V_IN_PROT
      -> Buck 5V_MAIN
      -> Buck/LDO 3V3_LOGIC
      -> Iso DC/DC 5V_ISO_A
      -> Iso DC/DC 5V_ISO_B
      -> eFuse 24V_FIELD_A
      -> eFuse 24V_FIELD_B
      -> eFuse 24V_FIELD_C
   -> current/voltage sense -> Supervisor
```

### Hauptblöcke
- J1 Eingangsversorgung 24 VDC + PE
- F1 Eingangssicherung
- D1 Reverse-Polarity / ideal diode
- TVS1 Surge Clamp
- U1 Hot-Swap / Inrush Controller
- EMI-Filter / Common-Mode + Pi
- U2 Hauptbuck `24V_IN_PROT -> 5V_MAIN`
- U3 Sekundärregler `5V_MAIN -> 3V3_LOGIC`
- U4/U5 isolierte Wandler `-> 5V_ISO_A/B`
- U6/U7/U8 eFuse/Current Limit `-> 24V_FIELD_A/B/C`

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `24V_IN_RAW` | IN | J1 | roher 24-V-Eingang |
| `PE_CHASSIS` | IN | J1 / Chassis | Schutzleiter / Gehäuse |
| `EN_5V_MAIN` | IN | 03 | Supervisor schaltet 5-V-Hauptschiene frei |
| `EN_3V3_LOGIC` | IN | 03 | optional getrenntes Enable |
| `EN_ISO_A`, `EN_ISO_B` | IN | 03 | Freigabe isolierter Busversorgungen |
| `EN_FIELD_A/B/C` | IN | 03 | Freigabe Feldspannungszweige |
| `24V_IN_PROT` | OUT | 03/06/07 | geschützter 24-V-Bus |
| `5V_MAIN` | OUT | 02/03/09/10/11 | zentrale 5-V-Logikversorgung |
| `3V3_LOGIC` | OUT | 02/03/09/10/11 | 3,3-V-Logik |
| `5V_ISO_A`, `5V_ISO_B` | OUT | 05/08 | isolierte Busversorgungen |
| `24V_FIELD_A/B/C` | OUT | 06/07 | abgesicherte Feldversorgung |
| `PWR_GOOD` | OUT | 02/03 | Gesamtsystem Power Good |
| `24V_UV_WARN` | OUT | 03 | Unterspannung erkannt |
| `24V_OV_WARN` | OUT | 03 | Überspannung erkannt |
| `PWR_FAULT_N` | OUT | 03 | latched Power Fault |
| `I_MON_24V` | OUT | 03 | Analog-/ADC-Messwert Gesamtstrom |
| `TEMP_PWR_WARN` | OUT | 03 | Temperaturwarnung Powertree |
| `DGND` | REF | alle Logik-Sheets | Logikmasse |
| `CHASSIS` | REF | 04/05/08/09 | Schirm-/Gehäusebezug |

**Kommentar:** Blatt 01 ist die einzige Quelle der Primärstrompfade. Alle Feld- und Isolationsdomänen werden von hier aus sternförmig versorgt; das entspricht der im Plan geforderten Fault-Containment- und PDN-Strategie. 

---

## 02 – [COMPUTE_CORE]

### Blockdiagramm

```text
5V_MAIN / 3V3_LOGIC
   -> Industrial SoM
   -> boot straps / reset / status LEDs
   -> host interfaces
      -> ETH x2
      -> RS485 x2
      -> IO-Link SPI
      -> DIO control/status
      -> CAN
      -> USB/UART service
      -> TPM / EEPROM / RTC
```

### Hauptblöcke
- X1 Industrial SoM
- Boot-Strap-Netzwerk
- Reset-Logik
- Host-Status-LEDs
- Level-Translation, falls nötig
- lokale Bulk-/HF-Decoupling-Inseln

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | Hauptversorgung SoM |
| `3V3_LOGIC` | IN | 01 | I/O-/Hilfsspannung |
| `PWR_GOOD` | IN | 01 | Freigabe Boot-Sequenz |
| `SYS_RESET_N` | IN | 03 | globaler Reset vom Supervisor |
| `SOM_RESET_N` | IN | 03 | separater SoM-Reset |
| `SAFE_MODE_REQ` | IN | 03 | Boot/Runtime in Safe Mode |
| `FORCE_RECOVERY_N` | IN | 09/03 | Recovery-Boot erzwingen |
| `WDOG_KICK` | OUT | 03 | Lebenszeichen/Watchdog |
| `SAFE_MODE_ACK` | OUT | 03 | SoM hat Safe Mode angenommen |
| `SOM_HEARTBEAT` | OUT | 03 | periodischer Alive-Puls |
| `ETH1_MDI0..3_P/N` | BIDI | 04 | Ethernet Port 1 Datenpaare |
| `ETH2_MDI0..3_P/N` | BIDI | 04 | Ethernet Port 2 Datenpaare |
| `ETH1_LED_LINK_N`, `ETH1_LED_ACT_N` | OUT | 04 | Port-LED-Ansteuerung |
| `ETH2_LED_LINK_N`, `ETH2_LED_ACT_N` | OUT | 04 | Port-LED-Ansteuerung |
| `RS485A_TXD` | OUT | 05 | UART TX Kanal A |
| `RS485A_RXD` | IN | 05 | UART RX Kanal A |
| `RS485A_DE`, `RS485A_RE_N` | OUT | 05 | Richtungskontrolle Kanal A |
| `RS485A_TERM_EN` | OUT | 05 | Abschluss zuschaltbar |
| `RS485B_TXD` | OUT | 05 | UART TX Kanal B |
| `RS485B_RXD` | IN | 05 | UART RX Kanal B |
| `RS485B_DE`, `RS485B_RE_N` | OUT | 05 | Richtungskontrolle Kanal B |
| `RS485B_TERM_EN` | OUT | 05 | Abschluss zuschaltbar |
| `IOL_SPI_SCLK`, `IOL_SPI_MOSI` | OUT | 06 | IO-Link Host-SPI |
| `IOL_SPI_MISO` | IN | 06 | IO-Link Host-SPI |
| `IOL_CS0_N`, `IOL_CS1_N` | OUT | 06 | Select der IO-Link-Master-ICs |
| `IOL_INT0_N`, `IOL_INT1_N` | IN | 06 | IO-Link Interrupts |
| `IOL_RESET_N` | OUT | 06 | Reset IO-Link-Controller |
| `DI_SENSE[7:0]` | IN | 07 | digitale Eingänge an Host |
| `DO_CMD[7:0]` | OUT | 07 | Schaltbefehle Ausgänge |
| `DO_EN[7:0]` | OUT | 07 | separates Kanal-Enable |
| `DO_FLT_N[7:0]` | IN | 07 | Fehler-/Kurzschlussstatus |
| `CAN1_TXD` | OUT | 08 | CAN TX |
| `CAN1_RXD` | IN | 08 | CAN RX |
| `CAN1_STB_N` | OUT | 08 | CAN Standby |
| `CAN1_TERM_EN` | OUT | 08 | Zuschaltbarer CAN-Abschluss |
| `USB0_D_P/N` | BIDI | 09 | USB-C Device/Console |
| `UART_DBG_TX` | OUT | 09 | serielle Debug-Konsole |
| `UART_DBG_RX` | IN | 09 | serielle Eingabe |
| `BOOT_MODE[2:0]` | IN | 09 | Boot-Strap von DIP/Jumper |
| `SPI_TPM_*` | BIDI | 10 | TPM 2.0 |
| `I2C_SEC_SCL/SDA` | BIDI | 10 | EEPROM / RTC / Board-ID |
| `RTC_INT_N` | IN | 10 | Alarm / Zeitbasis |
| `BOARD_ID[3:0]` | IN | 10 | feste Variantenkennung |

**Kommentar:** Das Blatt 02 ist bewusst auf Host-/Datenpfad konzentriert; alle harten Schutz-, Sequenz- und Recovery-Funktionen bleiben außerhalb im Supervisor- und Power-Zweig. Das folgt der SoM- plus Supervisor-Architektur des Plans. 

---

## 03 – [SUPERVISOR_MCU]

### Blockdiagramm

```text
rail sense + fault flags + current sense
      -> MCU ADC / GPIO / timer watchdog
      -> reset tree / enables / safe mode
      -> status LEDs / event logging / service hooks
```

### Hauptblöcke
- U30 Supervisor-MCU
- ADC-Messnetz für Rails und Temperaturen
- Reset-/Power-Sequencing
- Hardware-Watchdog-Fenster
- Event-/Fault-Latch
- Board-Status-LEDs

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | MCU-Versorgung |
| `3V3_LOGIC` | IN | 01 | MCU-I/O |
| `24V_UV_WARN`, `24V_OV_WARN` | IN | 01 | Spannungswarnungen |
| `PWR_FAULT_N` | IN | 01 | Power Fault |
| `I_MON_24V` | IN | 01 | Gesamtstrom-Monitor |
| `TEMP_PWR_WARN` | IN | 01 | Temperaturwarnung Power |
| `SOM_HEARTBEAT` | IN | 02 | SoM-Aktivität |
| `WDOG_KICK` | IN | 02 | Watchdog-Service |
| `SAFE_MODE_ACK` | IN | 02 | Host im Safe Mode |
| `DO_FLT_N[7:0]` | IN | 07 | DO-Fehlerstatus |
| `IOL0_OC_N..IOL3_OC_N` | IN | 06 | IO-Link Überstrom |
| `IOL0_TSD_N..IOL3_TSD_N` | IN | 06 | IO-Link Thermik |
| `CAN1_FAULT_N` | IN | 08 | CAN-Fehler |
| `RS485_FAULT_SUM_N` | IN | 05 | RS-485 Sammelfehler |
| `EN_5V_MAIN` | OUT | 01 | Freigabe Hauptversorgung |
| `EN_3V3_LOGIC` | OUT | 01 | Freigabe 3,3 V |
| `EN_ISO_A`, `EN_ISO_B` | OUT | 01 | Freigabe isolierte Wandler |
| `EN_FIELD_A/B/C` | OUT | 01 | Freigabe Feldzweige |
| `SYS_RESET_N` | OUT | 02/09/11 | globaler Reset |
| `SOM_RESET_N` | OUT | 02 | gezielter SoM-Reset |
| `SAFE_MODE_REQ` | OUT | 02 | Safe Mode anfordern |
| `FORCE_RECOVERY_N` | OUT | 02/09 | Recovery-Boot auslösen |
| `STATUS_LED_G`, `STATUS_LED_Y`, `STATUS_LED_R` | OUT | 09/front panel | Systemstatus |
| `SWDIO`, `SWCLK`, `NRST_MCU` | BIDI | 09 | MCU-Debug |
| `EVENT_LOG_INT` | OUT | 10 | optionale Secure-Event-Zählung |

**Kommentar:** Blatt 03 ist die Trennlinie zwischen „System lebt noch“ und „Host-Software hängt“. Damit bekommt das Design eine echte Hardware-Supervision, wie im PCB-Plan für Fault Isolation, Brownout Handling, Recovery und Safe Mode vorgesehen. 

---

## 04 – [ETHERNET_PORTS]

### Blockdiagramm

```text
SoM Ethernet pairs
   -> ESD / CMC / optional PHY-side conditioning
   -> magnetics
   -> RJ45 or M12 X-coded
   -> shield to CHASSIS
```

### Freeze-Annahme
Ich friere die **Sheet-Schnittstelle am Port-seitigen Ethernet-Datenpfad** ein. Wenn das gewählte SoM statt MDI-Paaren nur MAC/RGMII liefert, bleibt die Außenwelt identisch; in Blatt 04 werden dann zusätzlich PHY-Unterblöcke eingefügt.

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `ETH1_MDI0..3_P/N` | BIDI | 02 | Port 1 Datenpaare |
| `ETH2_MDI0..3_P/N` | BIDI | 02 | Port 2 Datenpaare |
| `ETH1_LED_LINK_N`, `ETH1_LED_ACT_N` | IN | 02 | LED-Steuerung |
| `ETH2_LED_LINK_N`, `ETH2_LED_ACT_N` | IN | 02 | LED-Steuerung |
| `3V3_LOGIC` | IN | 01 | LED-/Hilfsspannung |
| `DGND` | REF | 01 | Logikreferenz |
| `CHASSIS` | REF | 01 | Schirm-/Gehäuseanschluss |
| `ETH1_SHIELD` | OUT | J2 | Port-1-Schirm |
| `ETH2_SHIELD` | OUT | J3 | Port-2-Schirm |
| `ETH1_TXRX_PORT` | BIDI | J2 | externer Ethernet-Port 1 |
| `ETH2_TXRX_PORT` | BIDI | J3 | externer Ethernet-Port 2 |

### Externe Stecker
- `J2_ETH1`
- `J3_ETH2`

**Kommentar:** Das Blatt bildet die geforderte galvanische Entkopplung über Magnetics, ESD-Schutz und saubere Schirmankopplung zum Chassis ab; genau diese Trennung ist im PCB-Plan für die Northbound-Ethernet-Domäne vorgesehen. 

---

## 05 – [RS485_MODBUS_RTU]

### Blockdiagramm

```text
Host UART / DE-RE
   -> isolated RS-485 transceiver A
   -> TVS / bias / termination / connector
   -> MBUS_A

Host UART / DE-RE
   -> isolated RS-485 transceiver B
   -> TVS / bias / termination / connector
   -> MBUS_B
```

### Hauptblöcke
- Kanal A komplett isoliert
- Kanal B komplett isoliert
- optional zuschaltbarer Abschluss
- Bias-Netzwerk
- Surge-/ESD-Schutz
- Diagnose-/Fault-Ausgang

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | Logikseite RS-485 |
| `5V_ISO_A` | IN | 01 | isolierte Feldversorgung Kanal A |
| `5V_ISO_B` | IN | 01 | isolierte Feldversorgung Kanal B |
| `DGND` | REF | 01 | Host-Referenz |
| `FGND_A`, `FGND_B` | REF | lokal / Feld | Feldreferenzen |
| `RS485A_TXD` | IN | 02 | Host TX A |
| `RS485A_RXD` | OUT | 02 | Host RX A |
| `RS485A_DE`, `RS485A_RE_N` | IN | 02 | Richtung A |
| `RS485A_TERM_EN` | IN | 02 | Abschluss A |
| `RS485B_TXD` | IN | 02 | Host TX B |
| `RS485B_RXD` | OUT | 02 | Host RX B |
| `RS485B_DE`, `RS485B_RE_N` | IN | 02 | Richtung B |
| `RS485B_TERM_EN` | IN | 02 | Abschluss B |
| `RS485_FAULT_SUM_N` | OUT | 03 | Sammelfehler beider Kanäle |
| `RS485A_A`, `RS485A_B` | BIDI | J4 | externer Bus A |
| `RS485B_A`, `RS485B_B` | BIDI | J5 | externer Bus B |
| `CHASSIS` | REF | 01 | Schirm / ggf. Gehäusebezug |

### Externe Stecker
- `J4_MBUS_A`
- `J5_MBUS_B`

**Kommentar:** Blatt 05 setzt die im Plan geforderte isolierte, diagnostizierbare Modbus-RTU-Domäne um. Jeder Kanal ist elektrisch eigenständig, damit ein Feldfehler nicht in die Compute-Domäne zurückkoppelt. 

---

## 06 – [IOLINK_MASTER_4PORT]

### Blockdiagramm

```text
Host SPI
   -> IO-Link Master IC #0 -> Port 0 / Port 1
   -> IO-Link Master IC #1 -> Port 2 / Port 3

24V_FIELD_A/B
   -> per-port protected L+
   -> CQ driver / current limit / thermal feedback
   -> M12 port connectors
```

### Interne Unterteilung
- `PORT0_BLOCK`
- `PORT1_BLOCK`
- `PORT2_BLOCK`
- `PORT3_BLOCK`

Alle vier Portblöcke müssen **topologisch identisch** sein.

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `24V_FIELD_A` | IN | 01 | Feldversorgung Portgruppe 0/1 |
| `24V_FIELD_B` | IN | 01 | Feldversorgung Portgruppe 2/3 |
| `5V_MAIN` | IN | 01 | Hilfsspannung |
| `3V3_LOGIC` | IN | 01 | Logikversorgung |
| `DGND` | REF | 01 | Logikreferenz |
| `IOL_SPI_SCLK`, `IOL_SPI_MOSI` | IN | 02 | Host-SPI |
| `IOL_SPI_MISO` | OUT | 02 | Host-SPI zurück |
| `IOL_CS0_N`, `IOL_CS1_N` | IN | 02 | Chip Select |
| `IOL_INT0_N`, `IOL_INT1_N` | OUT | 02 | Interrupts |
| `IOL_RESET_N` | IN | 02 | Reset für Master-ICs |
| `IOL0_OC_N..IOL3_OC_N` | OUT | 03 | Überstrom je Port |
| `IOL0_TSD_N..IOL3_TSD_N` | OUT | 03 | Temperaturfault je Port |
| `IOL0_CQ` | BIDI | J6 | IO-Link Port 0 C/Q |
| `IOL0_LPLUS` | OUT | J6 | IO-Link Port 0 L+ |
| `IOL0_LMINUS` | REF | J6 | IO-Link Port 0 L- |
| `IOL1_CQ` | BIDI | J7 | IO-Link Port 1 C/Q |
| `IOL1_LPLUS` | OUT | J7 | IO-Link Port 1 L+ |
| `IOL1_LMINUS` | REF | J7 | IO-Link Port 1 L- |
| `IOL2_CQ` | BIDI | J8 | IO-Link Port 2 C/Q |
| `IOL2_LPLUS` | OUT | J8 | IO-Link Port 2 L+ |
| `IOL2_LMINUS` | REF | J8 | IO-Link Port 2 L- |
| `IOL3_CQ` | BIDI | J9 | IO-Link Port 3 C/Q |
| `IOL3_LPLUS` | OUT | J9 | IO-Link Port 3 L+ |
| `IOL3_LMINUS` | REF | J9 | IO-Link Port 3 L- |
| `CHASSIS` / `FE_IOL` | REF | J6..J9 | Funktionserde / Schirm falls verwendet |

### Externe Stecker
- `J6_IOL0`
- `J7_IOL1`
- `J8_IOL2`
- `J9_IOL3`

**Kommentar:** Das Blatt setzt die im Plan definierte 4-Port-IO-Link-Master-Domäne mit separatem L+-Pfad, Kurzschluss-/Thermoschutz und identischer Kanalstruktur um. 

---

## 07 – [DISCRETE_IO_8DI_8DO]

### Blockdiagramm

```text
24V_FIELD_C
   -> DI front-end x8  -> DI_SENSE[7:0] -> Host
   -> DO high-side / configurable bank x8 <- DO_CMD[7:0] from Host
                         -> DO_FLT_N[7:0] -> Host/Supervisor
```

### Struktur
- `DI_BANK0` = DI0..DI3
- `DI_BANK1` = DI4..DI7
- `DO_BANK0` = DO0..DO3
- `DO_BANK1` = DO4..DO7

Wenn die endgültige Produktentscheidung statt fester DO-Treiber auf flexible DIO-Bausteine geht, bleibt die **Sheet-Schnittstelle identisch**; nur die Kanalimplementierung ändert sich.

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `24V_FIELD_C` | IN | 01 | Feldversorgung DIO |
| `3V3_LOGIC` | IN | 01 | Logikversorgung |
| `DGND` | REF | 01 | Logikreferenz |
| `DI_SENSE[7:0]` | OUT | 02 | digitale Eingänge zum Host |
| `DO_CMD[7:0]` | IN | 02 | Schaltkommandos |
| `DO_EN[7:0]` | IN | 02 | getrenntes Aktivieren |
| `DO_FLT_N[7:0]` | OUT | 02/03 | Kurzschluss / OC / OT |
| `DIO_BANK0_MODE` | IN | 03/09 | Bankmodus 0 |
| `DIO_BANK1_MODE` | IN | 03/09 | Bankmodus 1 |
| `DI0_IN ... DI7_IN` | IN | J10 | externe 24-V-Eingänge |
| `DO0_OUT ... DO7_OUT` | OUT | J11 | externe 24-V-Ausgänge |
| `DI_COM_A/B` | REF | J10 | Eingangscommon |
| `DO_COM_A/B` | REF | J11 | Ausgangsreturn / common |
| `FE_DIO` | REF | J10/J11 | funktionale Erde / Schirm optional |

### Externe Stecker
- `J10_DI_BANK`
- `J11_DO_BANK`

**Kommentar:** Das Blatt bildet die im Plan genannte 8DI/8DO-Funktion ab, lässt aber die noch offene sourcing/sinking-/flex-DIO-Entscheidung hardware-architektonisch sauber offen. 

---

## 08 – [CAN_CANOPEN_EXT]

### Blockdiagramm

```text
Host CAN TX/RX
   -> CAN transceiver
   -> protection / optional isolation / termination
   -> CAN connector
```

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | Logikversorgung |
| `5V_ISO_B` | IN | 01 | optional isolierte Feldseite |
| `DGND` | REF | 01 | Host-Referenz |
| `FGND_B` | REF | Feldseite | Bus-Referenz |
| `CAN1_TXD` | IN | 02 | Host TX |
| `CAN1_RXD` | OUT | 02 | Host RX |
| `CAN1_STB_N` | IN | 02 | Standby-Freigabe |
| `CAN1_TERM_EN` | IN | 02 | 120-Ohm-Abschluss |
| `CAN1_FAULT_N` | OUT | 03 | Fault-Status |
| `CANH`, `CANL` | BIDI | J12 | CAN-Differenzpaar |
| `CHASSIS` | REF | J12 / 01 | Schirm / Gehäuse |

### Externer Stecker
- `J12_CAN_EXT`

**Kommentar:** Dieses Blatt bleibt optional bestückbar, wie im PCB-Plan vorgesehen. Die Schnittstelle zum Host bleibt aber vollständig definiert, damit das Layout extension-ready ist. 

---

## 09 – [SERVICE_DEBUG_RECOVERY]

### Blockdiagramm

```text
USB-C service + ESD + CC
UART debug header
SWD/JTAG header
Recovery button + boot DIP
Status LEDs
```

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | Service-Versorgung |
| `3V3_LOGIC` | IN | 01 | Debug-Logik |
| `USB0_D_P/N` | BIDI | 02 / J13 | USB-C Daten |
| `USB0_CC1`, `USB0_CC2` | BIDI | J13 | USB-C CC |
| `UART_DBG_TX` | IN | 02 | TX vom SoM zur Konsole |
| `UART_DBG_RX` | OUT | 02 | RX zum SoM |
| `SWDIO`, `SWCLK`, `NRST_MCU` | BIDI | 03 / J15 | MCU-Debug |
| `BOOT_MODE[2:0]` | OUT | 02 | Bootstraps aus DIP/Jumper |
| `FORCE_RECOVERY_N` | OUT | 02/03 | Recovery-Taste/Jumper |
| `STATUS_LED_G/Y/R` | IN | 03 | Frontpanel-Status |
| `DGND` | REF | 01 | Logikreferenz |
| `CHASSIS` | REF | 01 / J13 | USB-Schirm auf Gehäuse |

### Externe Stecker / Bedienelemente
- `J13_USB_C_SERVICE`
- `J14_UART_DEBUG`
- `J15_SWD_HEADER`
- `S1_RECOVERY`
- `SW1_BOOT_MODE_DIP`

**Kommentar:** Blatt 09 implementiert genau den Service-/Recovery-Zweig, den der PCB-Plan für Update, Diagnose und Boot-Wiederherstellung fordert. 

---

## 10 – [SECURITY_NVM_RTC]

### Blockdiagramm

```text
SoM SPI -> TPM 2.0
SoM I2C -> EEPROM / Board-ID / RTC
Optional tamper / secure event counter
```

### Hauptblöcke
- TPM 2.0
- Board-EEPROM / Seriennummer / Variantendaten
- RTC mit Backup
- optional Secure Element oder monotonic counter

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | Logikversorgung |
| `VBAT_RTC` | IN | lokale Backup-Quelle | RTC Backup |
| `DGND` | REF | 01 | Referenz |
| `SPI_TPM_SCLK`, `SPI_TPM_MOSI` | IN | 02 | TPM SPI |
| `SPI_TPM_MISO` | OUT | 02 | TPM SPI zurück |
| `SPI_TPM_CS_N` | IN | 02 | TPM Select |
| `TPM_IRQ_N` | OUT | 02 | TPM Interrupt optional |
| `I2C_SEC_SCL/SDA` | BIDI | 02 | EEPROM / RTC / ID |
| `RTC_INT_N` | OUT | 02 | Alarm / Wake |
| `BOARD_ID[3:0]` | OUT | 02 | Varianten-ID |
| `EVENT_LOG_INT` | IN | 03 | Supervisor-Event zählen |
| `TAMPER_IN` | IN | optional ext./switch | Manipulationserkennung |

**Kommentar:** Dieses Blatt stützt die im Plan geforderten Funktionen für sichere Identität, Signaturen, Rollback-/Recovery-Unterstützung und persistente Board-Identität ab. 

---

## 11 – [TEST_PRODUCTION_SUPPORT]

### Blockdiagramm

```text
global rail test points
bus loopbacks
current shunts / measurement pads
bed-of-nails access
factory straps
```

### Sheet-Interfaces

| Interface / Net | Dir. | Gegenblatt / Ziel | Funktion |
|---|---:|---|---|
| `TP_24V_IN_RAW` | MON | 01 | Eingangsspannung |
| `TP_24V_IN_PROT` | MON | 01 | geschützte 24 V |
| `TP_5V_MAIN` | MON | 01 | Haupt-5V |
| `TP_3V3_LOGIC` | MON | 01 | Logik-3V3 |
| `TP_PW_GOOD` | MON | 01/03 | Sequencing-Prüfung |
| `TP_SYS_RESET_N` | MON | 03 | Reset-Verhalten |
| `TP_WDOG_KICK` | MON | 02/03 | Watchdog |
| `TP_RS485A_A/B`, `TP_RS485B_A/B` | MON | 05 | Busprüfung |
| `TP_CANH`, `TP_CANL` | MON | 08 | CAN-Prüfung |
| `TP_IOL0_CQ ... TP_IOL3_CQ` | MON | 06 | IO-Link-Bed-of-Nails |
| `TP_DI[7:0]`, `TP_DO[7:0]` | MON | 07 | DIO-Verifikation |
| `LB_RS485A_EN`, `LB_RS485B_EN` | IN | Strap/Jumper | Loopback für Test |
| `LB_CAN1_EN` | IN | Strap/Jumper | CAN-Testpfad |
| `LB_IOL_SIM_EN` | IN | Strap/Jumper | IO-Link Simulation |
| `FACTORY_ID_PROG` | BIDI | 10 | Seriennummer / EEPROM Programmierung |

**Kommentar:** Das Blatt 11 ist die formale Heimat für alle Testpunkte, Loopbacks, Serienprogrammierung und Pogo-Pad-Zugänge. Das passt zur im Plan genannten DFT-/Produktionstest-Strategie über EVT, DVT und PVT. 

---

# 6. Intersheet-Backbone: empfohlene hierarchische Ports im Top-Sheet

Wenn du das in Altium/KiCad/OrCAD anlegst, würde ich im Root-Sheet diese logischen Backbones führen:

### Power Backbone
- `24V_IN_PROT`
- `24V_FIELD_A/B/C`
- `5V_MAIN`
- `5V_ISO_A/B`
- `3V3_LOGIC`
- `DGND`
- `CHASSIS`

### Safety / Supervision Backbone
- `PWR_GOOD`
- `SYS_RESET_N`
- `SOM_RESET_N`
- `SAFE_MODE_REQ`
- `SAFE_MODE_ACK`
- `WDOG_KICK`
- `PWR_FAULT_N`
- `24V_UV_WARN`
- `24V_OV_WARN`

### Host Peripheral Backbone
- Ethernet-Port-Paare
- `RS485A_*`, `RS485B_*`
- `IOL_*`
- `DI_SENSE[7:0]`
- `DO_CMD[7:0]`, `DO_EN[7:0]`, `DO_FLT_N[7:0]`
- `CAN1_*`
- `USB0_*`
- `UART_DBG_*`
- `SPI_TPM_*`
- `I2C_SEC_*`

---

# 7. Externe Interface-Definitionen auf Produktebene

## Power
- **J1_POWER_IN**
  - `24V_IN+`
  - `0V_IN`
  - `PE_CHASSIS`

## Ethernet
- **J2_ETH1**
- **J3_ETH2**

## Serial / Fieldbus
- **J4_MBUS_A**: `RS485A_A`, `RS485A_B`, optional `FGND_A`, Shield
- **J5_MBUS_B**: `RS485B_A`, `RS485B_B`, optional `FGND_B`, Shield
- **J12_CAN_EXT**: `CANH`, `CANL`, optional `FGND_B`, Shield

## IO-Link
- **J6..J9**
  - `L+`
  - `L-`
  - `C/Q`
  - optional `P2`
  - optional FE/Schirm

## DIO
- **J10_DI_BANK**
  - `DI0..DI7`
  - `DI_COM_A/B`
- **J11_DO_BANK**
  - `DO0..DO7`
  - `DO_COM_A/B`

## Service
- **J13_USB_C_SERVICE**
- **J14_UART_DEBUG**
- **J15_SWD_HEADER**

---

# 8. Was ich ausdrücklich als „noch offen, aber jetzt sauber eingefroren“ definieren würde

Diese Punkte bleiben laut Plan offen; die Schaltplanstruktur oben ist trotzdem vollständig genug, um mit dem Capture zu beginnen:

1. **Ethernet-Steckverbinderfamilie**  
   RJ45 vs. M12 X-coded.

2. **Feldsteckverbinder**  
   Schraubklemme vs. M8/M12.

3. **DIO-Endtopologie**  
   fest sourcing, sink/source-bankweise oder flexible DIO-ICs.

4. **CAN-Bestückung**  
   optional DNI oder Serienfeature.

5. **Isolationsgrenzen im Detail**  
   endgültige Prüfspannung, Creepage/Clearance.

6. **Exakte Strombudgets pro Feldbank**  
   besonders IO-Link L+ und DO-Lastecken.

7. **PE/Chassis-Kopplungsnetzwerk**  
   RC-/HF-Kopplung und Gehäusekonzept.

8. **SoM-Ethernet-Boundary**  
   MDI direkt vs. MAC/RGMII + PHY auf Carrier.

Diese Offenpunkte sind im PCB-Plan selbst benannt beziehungsweise impliziert;


---
---
Ich habe die Schaltplan-Hierarchie in eine **normalisierte Sheet-to-Sheet-Verbindungsmatrix** überführt. Dabei habe ich die zuvor beschriebenen Blätter 01–11 beibehalten und zwei potenziell mehrdeutige Steuerpfade für die CAD-Umsetzung sauber auf **Single-Source-Signale** normalisiert:  
`FORCE_RECOVERY_REQ_N` kommt von **Sheet 09** zum **Supervisor in Sheet 03**; daraus erzeugt **Sheet 03** das eigentliche `FORCE_RECOVERY_N` zum **Compute-Core in Sheet 02**.  
Ebenso kommt die Bank-Konfiguration logisch aus dem Service-/Config-Pfad, wird aber über **Sheet 03** an **Sheet 07** als `DIO_BANK0_MODE` und `DIO_BANK1_MODE` ausgegeben. Das vermeidet Multi-Driver-Netze im Schaltplan. 

---

# Sheet-to-Sheet-Verbindungsmatrix

## A. Versorgungs-, Referenz- und Supervisor-Backbone

| Nr. | Netzname | Quelle (Sheet.Block) | Ziel (Sheet.Block) | Richtung | Signaltyp | Zweck |
|---:|---|---|---|---|---|---|
| A001 | `24V_IN_PROT` | 01.PowerTree | 03.Supervisor_ADC | 01 → 03 | PWR/MON | geschützter 24-V-Bus zur Überwachung |
| A002 | `24V_FIELD_A` | 01.FieldPower_A | 06.IOLink_PortGroup_0_1 | 01 → 06 | PWR | Feldversorgung IO-Link Port 0/1 |
| A003 | `24V_FIELD_B` | 01.FieldPower_B | 06.IOLink_PortGroup_2_3 | 01 → 06 | PWR | Feldversorgung IO-Link Port 2/3 |
| A004 | `24V_FIELD_C` | 01.FieldPower_C | 07.DIO_FieldPower | 01 → 07 | PWR | Feldversorgung DI/DO |
| A005 | `5V_MAIN` | 01.MainBuck | 02.SoM_Power | 01 → 02 | PWR | Hauptversorgung Compute |
| A006 | `5V_MAIN` | 01.MainBuck | 03.Supervisor_Power | 01 → 03 | PWR | Hauptversorgung Supervisor |
| A007 | `5V_MAIN` | 01.MainBuck | 06.IOLink_LogicAux | 01 → 06 | PWR | Hilfsspannung IO-Link |
| A008 | `5V_MAIN` | 01.MainBuck | 09.ServicePower | 01 → 09 | PWR | Service-/Debug-Versorgung |
| A009 | `3V3_LOGIC` | 01.LogicReg | 02.SoM_IO | 01 → 02 | PWR | Logikversorgung Compute |
| A010 | `3V3_LOGIC` | 01.LogicReg | 03.Supervisor_IO | 01 → 03 | PWR | Logikversorgung Supervisor |
| A011 | `3V3_LOGIC` | 01.LogicReg | 04.Eth_LED_ESD | 01 → 04 | PWR | LED-/Porthilfsspannung |
| A012 | `3V3_LOGIC` | 01.LogicReg | 05.RS485_LogicSide | 01 → 05 | PWR | Logikseite RS-485 |
| A013 | `3V3_LOGIC` | 01.LogicReg | 06.IOLink_Logic | 01 → 06 | PWR | IO-Link Logikversorgung |
| A014 | `3V3_LOGIC` | 01.LogicReg | 07.DIO_Logic | 01 → 07 | PWR | DIO Logikversorgung |
| A015 | `3V3_LOGIC` | 01.LogicReg | 08.CAN_Logic | 01 → 08 | PWR | CAN Logikversorgung |
| A016 | `3V3_LOGIC` | 01.LogicReg | 09.DebugLogic | 01 → 09 | PWR | Debug-/Boot-DIP-Versorgung |
| A017 | `3V3_LOGIC` | 01.LogicReg | 10.SecurityCore | 01 → 10 | PWR | TPM/EEPROM/RTC Versorgung |
| A018 | `5V_ISO_A` | 01.IsoDCDC_A | 05.RS485_FieldSide_A | 01 → 05 | PWR | isolierte Versorgung Kanal A |
| A019 | `5V_ISO_B` | 01.IsoDCDC_B | 05.RS485_FieldSide_B | 01 → 05 | PWR | isolierte Versorgung Kanal B |
| A020 | `5V_ISO_B` | 01.IsoDCDC_B | 08.CAN_FieldSide | 01 → 08 | PWR | optionale isolierte Versorgung CAN |
| A021 | `PWR_GOOD` | 01.PowerSeq | 02.SoM_ResetBoot | 01 → 02 | MON | Compute darf sauber booten |
| A022 | `PWR_GOOD` | 01.PowerSeq | 03.Supervisor_GPIO | 01 → 03 | MON | Supervisor kennt Power-Status |
| A023 | `24V_UV_WARN` | 01.PowerMonitor | 03.Supervisor_ADC | 01 → 03 | MON | Unterspannungswarnung |
| A024 | `24V_OV_WARN` | 01.PowerMonitor | 03.Supervisor_ADC | 01 → 03 | MON | Überspannungswarnung |
| A025 | `PWR_FAULT_N` | 01.PowerMonitor | 03.Supervisor_GPIO | 01 → 03 | MON | gelatchter Power-Fehler |
| A026 | `I_MON_24V` | 01.CurrentSense | 03.Supervisor_ADC | 01 → 03 | MON | Gesamtstrommessung |
| A027 | `TEMP_PWR_WARN` | 01.ThermalSense | 03.Supervisor_GPIO | 01 → 03 | MON | Temperaturwarnung Powertree |
| A028 | `DGND` | 01.LogicGround | 02.LogicGround | REF | REF | Logikreferenz |
| A029 | `DGND` | 01.LogicGround | 03.LogicGround | REF | REF | Logikreferenz |
| A030 | `DGND` | 01.LogicGround | 04.LogicGround | REF | REF | Logikreferenz |
| A031 | `DGND` | 01.LogicGround | 05.LogicGround | REF | REF | Logikreferenz |
| A032 | `DGND` | 01.LogicGround | 06.LogicGround | REF | REF | Logikreferenz |
| A033 | `DGND` | 01.LogicGround | 07.LogicGround | REF | REF | Logikreferenz |
| A034 | `DGND` | 01.LogicGround | 08.LogicGround | REF | REF | Logikreferenz |
| A035 | `DGND` | 01.LogicGround | 09.LogicGround | REF | REF | Logikreferenz |
| A036 | `DGND` | 01.LogicGround | 10.LogicGround | REF | REF | Logikreferenz |
| A037 | `CHASSIS` | 01.ChassisBond | 04.PortShieldBond | REF | REF | Ethernet-Schirm/Gehäuse |
| A038 | `CHASSIS` | 01.ChassisBond | 05.PortShieldBond | REF | REF | RS-485 Schirm/Gehäuse |
| A039 | `CHASSIS` | 01.ChassisBond | 08.PortShieldBond | REF | REF | CAN Schirm/Gehäuse |
| A040 | `CHASSIS` | 01.ChassisBond | 09.USBShieldBond | REF | REF | USB-/Service-Schirm |

Diese Matrix bündelt die globalen Versorgungs- und Referenznetze sowie alle vom Power-Sheet kommenden Supervisor-Monitoring-Signale. Sie entspricht der im Plan beschriebenen sternförmigen PDN-Topologie mit klarer Trennung von Logic-, Field- und Chassis-Domänen. 

---

## B. Steuer-, Daten- und Managementverbindungen zwischen den Sheets

| Nr. | Netzname | Quelle (Sheet.Block) | Ziel (Sheet.Block) | Richtung | Signaltyp | Zweck |
|---:|---|---|---|---|---|---|
| B001 | `EN_5V_MAIN` | 03.PowerSequencer | 01.MainBuck_Enable | 03 → 01 | CTRL | Freigabe Hauptversorgung |
| B002 | `EN_3V3_LOGIC` | 03.PowerSequencer | 01.LogicReg_Enable | 03 → 01 | CTRL | Freigabe 3V3 |
| B003 | `EN_ISO_A` | 03.PowerSequencer | 01.IsoDCDC_A_Enable | 03 → 01 | CTRL | Freigabe isolierte Versorgung A |
| B004 | `EN_ISO_B` | 03.PowerSequencer | 01.IsoDCDC_B_Enable | 03 → 01 | CTRL | Freigabe isolierte Versorgung B |
| B005 | `EN_FIELD_A` | 03.PowerSequencer | 01.FieldPower_A_Enable | 03 → 01 | CTRL | Freigabe IO-Link Gruppe 0/1 |
| B006 | `EN_FIELD_B` | 03.PowerSequencer | 01.FieldPower_B_Enable | 03 → 01 | CTRL | Freigabe IO-Link Gruppe 2/3 |
| B007 | `EN_FIELD_C` | 03.PowerSequencer | 01.FieldPower_C_Enable | 03 → 01 | CTRL | Freigabe DIO-Feldversorgung |
| B008 | `SYS_RESET_N` | 03.ResetTree | 02.SoM_ResetBoot | 03 → 02 | CTRL | globaler Systemreset |
| B009 | `SOM_RESET_N` | 03.ResetTree | 02.SoM_ResetBoot | 03 → 02 | CTRL | gezielter Compute-Reset |
| B010 | `SAFE_MODE_REQ` | 03.SafeModeCtrl | 02.SafeModeInput | 03 → 02 | CTRL | Safe-Mode anfordern |
| B011 | `FORCE_RECOVERY_N` | 03.RecoveryCtrl | 02.RecoveryBoot | 03 → 02 | CTRL | Recovery-Boot erzwingen |
| B012 | `SOM_HEARTBEAT` | 02.RuntimeMonitor | 03.WatchdogInput | 02 → 03 | MON | SoM-Lebenszeichen |
| B013 | `WDOG_KICK` | 02.RuntimeMonitor | 03.WatchdogInput | 02 → 03 | MON | Watchdog-Service |
| B014 | `SAFE_MODE_ACK` | 02.SafeModeOutput | 03.SafeModeCtrl | 02 → 03 | MON | Safe-Mode bestätigt |
| B015 | `STATUS_LED_G` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | grüne Status-LED |
| B016 | `STATUS_LED_Y` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | gelbe Status-LED |
| B017 | `STATUS_LED_R` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | rote Status-LED |
| B018 | `SWDIO` | 09.DebugHeader | 03.MCU_Debug | BIDI | DBG | SWD Daten |
| B019 | `SWCLK` | 09.DebugHeader | 03.MCU_Debug | 09 → 03 | DBG | SWD Clock |
| B020 | `NRST_MCU` | 09.DebugHeader | 03.MCU_Debug | BIDI | DBG | MCU Debug-Reset |
| B021 | `FORCE_RECOVERY_REQ_N` | 09.RecoveryButton | 03.RecoveryArbiter | 09 → 03 | CTRL | Recovery-Anforderung vom Serviceblatt |
| B022 | `EVENT_LOG_INT` | 03.EventCtrl | 10.SecureEventCounter | 03 → 10 | CTRL | Supervisor-Ereignis an Security/NVM |
| B023 | `DIO_BANK0_MODE` | 03.DIO_ModeCtrl | 07.DO_Bank0 | 03 → 07 | CTRL | DIO/DO Bankmodus 0 |
| B024 | `DIO_BANK1_MODE` | 03.DIO_ModeCtrl | 07.DO_Bank1 | 03 → 07 | CTRL | DIO/DO Bankmodus 1 |
| B025 | `ETH1_MDI0_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 0+ |
| B026 | `ETH1_MDI0_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 0− |
| B027 | `ETH1_MDI1_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 1+ |
| B028 | `ETH1_MDI1_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 1− |
| B029 | `ETH1_MDI2_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 2+ |
| B030 | `ETH1_MDI2_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 2− |
| B031 | `ETH1_MDI3_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 3+ |
| B032 | `ETH1_MDI3_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | Ethernet Port 1 Paar 3− |
| B033 | `ETH2_MDI0_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 0+ |
| B034 | `ETH2_MDI0_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 0− |
| B035 | `ETH2_MDI1_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 1+ |
| B036 | `ETH2_MDI1_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 1− |
| B037 | `ETH2_MDI2_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 2+ |
| B038 | `ETH2_MDI2_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 2− |
| B039 | `ETH2_MDI3_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 3+ |
| B040 | `ETH2_MDI3_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | Ethernet Port 2 Paar 3− |
| B041 | `ETH1_LED_LINK_N` | 02.SoM_ETH1 | 04.ETH1_LED | 02 → 04 | CTRL | Link-LED Port 1 |
| B042 | `ETH1_LED_ACT_N` | 02.SoM_ETH1 | 04.ETH1_LED | 02 → 04 | CTRL | Activity-LED Port 1 |
| B043 | `ETH2_LED_LINK_N` | 02.SoM_ETH2 | 04.ETH2_LED | 02 → 04 | CTRL | Link-LED Port 2 |
| B044 | `ETH2_LED_ACT_N` | 02.SoM_ETH2 | 04.ETH2_LED | 02 → 04 | CTRL | Activity-LED Port 2 |
| B045 | `RS485A_TXD` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | DATA | Modbus RTU A TX |
| B046 | `RS485A_RXD` | 05.RS485A_LogicIF | 02.UART_A | 05 → 02 | DATA | Modbus RTU A RX |
| B047 | `RS485A_DE` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | CTRL | Driver Enable A |
| B048 | `RS485A_RE_N` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | CTRL | Receiver Enable A |
| B049 | `RS485A_TERM_EN` | 02.UART_A | 05.RS485A_Termination | 02 → 05 | CTRL | Zuschaltbarer Abschluss A |
| B050 | `RS485B_TXD` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | DATA | Modbus RTU B TX |
| B051 | `RS485B_RXD` | 05.RS485B_LogicIF | 02.UART_B | 05 → 02 | DATA | Modbus RTU B RX |
| B052 | `RS485B_DE` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | CTRL | Driver Enable B |
| B053 | `RS485B_RE_N` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | CTRL | Receiver Enable B |
| B054 | `RS485B_TERM_EN` | 02.UART_B | 05.RS485B_Termination | 02 → 05 | CTRL | Zuschaltbarer Abschluss B |
| B055 | `RS485_FAULT_SUM_N` | 05.FaultAggregator | 03.BusFaultInput | 05 → 03 | MON | Sammelfehler RS-485 |
| B056 | `IOL_SPI_SCLK` | 02.SPI_IOL | 06.IOLink_SPI | 02 → 06 | DATA | IO-Link SPI Clock |
| B057 | `IOL_SPI_MOSI` | 02.SPI_IOL | 06.IOLink_SPI | 02 → 06 | DATA | IO-Link SPI MOSI |
| B058 | `IOL_SPI_MISO` | 06.IOLink_SPI | 02.SPI_IOL | 06 → 02 | DATA | IO-Link SPI MISO |
| B059 | `IOL_CS0_N` | 02.SPI_IOL | 06.IOLink_Master0 | 02 → 06 | CTRL | Chip Select IO-Link IC 0 |
| B060 | `IOL_CS1_N` | 02.SPI_IOL | 06.IOLink_Master1 | 02 → 06 | CTRL | Chip Select IO-Link IC 1 |
| B061 | `IOL_INT0_N` | 06.IOLink_Master0 | 02.GPIO_IOL | 06 → 02 | MON | Interrupt IO-Link IC 0 |
| B062 | `IOL_INT1_N` | 06.IOLink_Master1 | 02.GPIO_IOL | 06 → 02 | MON | Interrupt IO-Link IC 1 |
| B063 | `IOL_RESET_N` | 02.GPIO_IOL | 06.IOLink_Reset | 02 → 06 | CTRL | Reset IO-Link-Master |
| B064 | `IOL0_OC_N` | 06.Port0_Fault | 03.PortFaults | 06 → 03 | MON | Überstrom Port 0 |
| B065 | `IOL1_OC_N` | 06.Port1_Fault | 03.PortFaults | 06 → 03 | MON | Überstrom Port 1 |
| B066 | `IOL2_OC_N` | 06.Port2_Fault | 03.PortFaults | 06 → 03 | MON | Überstrom Port 2 |
| B067 | `IOL3_OC_N` | 06.Port3_Fault | 03.PortFaults | 06 → 03 | MON | Überstrom Port 3 |
| B068 | `IOL0_TSD_N` | 06.Port0_Fault | 03.PortFaults | 06 → 03 | MON | Thermofehler Port 0 |
| B069 | `IOL1_TSD_N` | 06.Port1_Fault | 03.PortFaults | 06 → 03 | MON | Thermofehler Port 1 |
| B070 | `IOL2_TSD_N` | 06.Port2_Fault | 03.PortFaults | 06 → 03 | MON | Thermofehler Port 2 |
| B071 | `IOL3_TSD_N` | 06.Port3_Fault | 03.PortFaults | 06 → 03 | MON | Thermofehler Port 3 |
| B072 | `DI_SENSE0` | 07.DI_Channel0 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 0 |
| B073 | `DI_SENSE1` | 07.DI_Channel1 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 1 |
| B074 | `DI_SENSE2` | 07.DI_Channel2 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 2 |
| B075 | `DI_SENSE3` | 07.DI_Channel3 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 3 |
| B076 | `DI_SENSE4` | 07.DI_Channel4 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 4 |
| B077 | `DI_SENSE5` | 07.DI_Channel5 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 5 |
| B078 | `DI_SENSE6` | 07.DI_Channel6 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 6 |
| B079 | `DI_SENSE7` | 07.DI_Channel7 | 02.GPIO_DI | 07 → 02 | DATA | Digitaleingang 7 |
| B080 | `DO_CMD0` | 02.GPIO_DO | 07.DO_Channel0 | 02 → 07 | CTRL | Schaltbefehl Ausgang 0 |
| B081 | `DO_CMD1` | 02.GPIO_DO | 07.DO_Channel1 | 02 → 07 | CTRL | Schaltbefehl Ausgang 1 |
| B082 | `DO_CMD2` | 02.GPIO_DO | 07.DO_Channel2 | 02 → 07 | CTRL | Schaltbefehl Ausgang 2 |
| B083 | `DO_CMD3` | 02.GPIO_DO | 07.DO_Channel3 | 02 → 07 | CTRL | Schaltbefehl Ausgang 3 |
| B084 | `DO_CMD4` | 02.GPIO_DO | 07.DO_Channel4 | 02 → 07 | CTRL | Schaltbefehl Ausgang 4 |
| B085 | `DO_CMD5` | 02.GPIO_DO | 07.DO_Channel5 | 02 → 07 | CTRL | Schaltbefehl Ausgang 5 |
| B086 | `DO_CMD6` | 02.GPIO_DO | 07.DO_Channel6 | 02 → 07 | CTRL | Schaltbefehl Ausgang 6 |
| B087 | `DO_CMD7` | 02.GPIO_DO | 07.DO_Channel7 | 02 → 07 | CTRL | Schaltbefehl Ausgang 7 |
| B088 | `DO_EN0` | 02.GPIO_DO | 07.DO_Channel0 | 02 → 07 | CTRL | Kanal-Enable Ausgang 0 |
| B089 | `DO_EN1` | 02.GPIO_DO | 07.DO_Channel1 | 02 → 07 | CTRL | Kanal-Enable Ausgang 1 |
| B090 | `DO_EN2` | 02.GPIO_DO | 07.DO_Channel2 | 02 → 07 | CTRL | Kanal-Enable Ausgang 2 |
| B091 | `DO_EN3` | 02.GPIO_DO | 07.DO_Channel3 | 02 → 07 | CTRL | Kanal-Enable Ausgang 3 |
| B092 | `DO_EN4` | 02.GPIO_DO | 07.DO_Channel4 | 02 → 07 | CTRL | Kanal-Enable Ausgang 4 |
| B093 | `DO_EN5` | 02.GPIO_DO | 07.DO_Channel5 | 02 → 07 | CTRL | Kanal-Enable Ausgang 5 |
| B094 | `DO_EN6` | 02.GPIO_DO | 07.DO_Channel6 | 02 → 07 | CTRL | Kanal-Enable Ausgang 6 |
| B095 | `DO_EN7` | 02.GPIO_DO | 07.DO_Channel7 | 02 → 07 | CTRL | Kanal-Enable Ausgang 7 |
| B096 | `DO_FLT_N0` | 07.DO_Channel0 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 0 an Host |
| B097 | `DO_FLT_N1` | 07.DO_Channel1 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 1 an Host |
| B098 | `DO_FLT_N2` | 07.DO_Channel2 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 2 an Host |
| B099 | `DO_FLT_N3` | 07.DO_Channel3 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 3 an Host |
| B100 | `DO_FLT_N4` | 07.DO_Channel4 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 4 an Host |
| B101 | `DO_FLT_N5` | 07.DO_Channel5 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 5 an Host |
| B102 | `DO_FLT_N6` | 07.DO_Channel6 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 6 an Host |
| B103 | `DO_FLT_N7` | 07.DO_Channel7 | 02.GPIO_DO_Fault | 07 → 02 | MON | Fehler Ausgang 7 an Host |
| B104 | `DO_FLT_N0` | 07.DO_Channel0 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 0 an Supervisor |
| B105 | `DO_FLT_N1` | 07.DO_Channel1 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 1 an Supervisor |
| B106 | `DO_FLT_N2` | 07.DO_Channel2 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 2 an Supervisor |
| B107 | `DO_FLT_N3` | 07.DO_Channel3 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 3 an Supervisor |
| B108 | `DO_FLT_N4` | 07.DO_Channel4 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 4 an Supervisor |
| B109 | `DO_FLT_N5` | 07.DO_Channel5 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 5 an Supervisor |
| B110 | `DO_FLT_N6` | 07.DO_Channel6 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 6 an Supervisor |
| B111 | `DO_FLT_N7` | 07.DO_Channel7 | 03.PortFaults | 07 → 03 | MON | Fehler Ausgang 7 an Supervisor |
| B112 | `CAN1_TXD` | 02.CAN_IF | 08.CAN_Transceiver | 02 → 08 | DATA | CAN TX |
| B113 | `CAN1_RXD` | 08.CAN_Transceiver | 02.CAN_IF | 08 → 02 | DATA | CAN RX |
| B114 | `CAN1_STB_N` | 02.CAN_IF | 08.CAN_Transceiver | 02 → 08 | CTRL | CAN Standby-Freigabe |
| B115 | `CAN1_TERM_EN` | 02.CAN_IF | 08.CAN_Termination | 02 → 08 | CTRL | CAN-Abschluss zuschalten |
| B116 | `CAN1_FAULT_N` | 08.CAN_Fault | 03.BusFaultInput | 08 → 03 | MON | CAN-Fehlerstatus |
| B117 | `USB0_D_P` | 02.USB_Device | 09.USB_C_Service | BIDI | HS_DIFF | USB 2.0 D+ |
| B118 | `USB0_D_N` | 02.USB_Device | 09.USB_C_Service | BIDI | HS_DIFF | USB 2.0 D− |
| B119 | `UART_DBG_TX` | 02.UART_Debug | 09.UART_Header | 02 → 09 | DBG | Debug-Konsole TX |
| B120 | `UART_DBG_RX` | 09.UART_Header | 02.UART_Debug | 09 → 02 | DBG | Debug-Konsole RX |
| B121 | `BOOT_MODE0` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | Bootmodus Bit 0 |
| B122 | `BOOT_MODE1` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | Bootmodus Bit 1 |
| B123 | `BOOT_MODE2` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | Bootmodus Bit 2 |
| B124 | `SPI_TPM_SCLK` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | DATA | TPM SPI Clock |
| B125 | `SPI_TPM_MOSI` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | DATA | TPM SPI MOSI |
| B126 | `SPI_TPM_MISO` | 10.TPM_SPI | 02.SecuritySPI | 10 → 02 | DATA | TPM SPI MISO |
| B127 | `SPI_TPM_CS_N` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | CTRL | TPM Chip Select |
| B128 | `TPM_IRQ_N` | 10.TPM_IRQ | 02.SecurityIRQ | 10 → 02 | MON | TPM Interrupt |
| B129 | `I2C_SEC_SCL` | 02.SecurityI2C | 10.EEPROM_RTC_I2C | BIDI | DATA | I²C Clock Security/NVM |
| B130 | `I2C_SEC_SDA` | 02.SecurityI2C | 10.EEPROM_RTC_I2C | BIDI | DATA | I²C Data Security/NVM |
| B131 | `RTC_INT_N` | 10.RTC | 02.RTC_Wakeup | 10 → 02 | MON | RTC Alarm/Wakeup |
| B132 | `BOARD_ID0` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | Varianten-ID Bit 0 |
| B133 | `BOARD_ID1` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | Varianten-ID Bit 1 |
| B134 | `BOARD_ID2` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | Varianten-ID Bit 2 |
| B135 | `BOARD_ID3` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | Varianten-ID Bit 3 |

Diese Tabelle enthält die eigentlichen Betriebs- und Managementpfade zwischen den Funktionsblättern: Sequencing, Reset, Safe Mode, Ethernet, Modbus RTU, IO-Link, DIO, CAN, Service/Debug sowie Security/NVM. Die Zuordnung folgt direkt der im Designplan beschriebenen Trennung von Compute-, Supervisor-, Fieldbus-, Service- und Security-Domäne. 

---

## C. Test-, Produktions- und Verifikationsmatrix

| Nr. | Netzname | Quelle (Sheet.Block) | Ziel (Sheet.Block) | Richtung | Signaltyp | Zweck |
|---:|---|---|---|---|---|---|
| C001 | `TP_24V_IN_RAW` | 01.InputEntry | 11.TestPads_Power | 01 → 11 | MON | Prüfpunkt roher Eingang |
| C002 | `TP_24V_IN_PROT` | 01.PowerTree | 11.TestPads_Power | 01 → 11 | MON | Prüfpunkt geschützte 24 V |
| C003 | `TP_5V_MAIN` | 01.MainBuck | 11.TestPads_Power | 01 → 11 | MON | Prüfpunkt 5V_MAIN |
| C004 | `TP_3V3_LOGIC` | 01.LogicReg | 11.TestPads_Power | 01 → 11 | MON | Prüfpunkt 3V3_LOGIC |
| C005 | `TP_PWR_GOOD` | 01.PowerSeq | 11.TestPads_Power | 01 → 11 | MON | Prüfpunkt PWR_GOOD |
| C006 | `TP_SYS_RESET_N` | 03.ResetTree | 11.TestPads_Reset | 03 → 11 | MON | Prüfpunkt Systemreset |
| C007 | `TP_WDOG_KICK` | 02.RuntimeMonitor | 11.TestPads_Reset | 02 → 11 | MON | Prüfpunkt Watchdog-Kick |
| C008 | `TP_RS485A_A` | 05.PortA_Field | 11.TestPads_Bus | 05 → 11 | MON | Prüfpunkt RS-485 A_A |
| C009 | `TP_RS485A_B` | 05.PortA_Field | 11.TestPads_Bus | 05 → 11 | MON | Prüfpunkt RS-485 A_B |
| C010 | `TP_RS485B_A` | 05.PortB_Field | 11.TestPads_Bus | 05 → 11 | MON | Prüfpunkt RS-485 B_A |
| C011 | `TP_RS485B_B` | 05.PortB_Field | 11.TestPads_Bus | 05 → 11 | MON | Prüfpunkt RS-485 B_B |
| C012 | `TP_CANH` | 08.CAN_Field | 11.TestPads_Bus | 08 → 11 | MON | Prüfpunkt CANH |
| C013 | `TP_CANL` | 08.CAN_Field | 11.TestPads_Bus | 08 → 11 | MON | Prüfpunkt CANL |
| C014 | `TP_IOL0_CQ` | 06.Port0_Field | 11.TestPads_IOL | 06 → 11 | MON | Prüfpunkt IO-Link Port 0 |
| C015 | `TP_IOL1_CQ` | 06.Port1_Field | 11.TestPads_IOL | 06 → 11 | MON | Prüfpunkt IO-Link Port 1 |
| C016 | `TP_IOL2_CQ` | 06.Port2_Field | 11.TestPads_IOL | 06 → 11 | MON | Prüfpunkt IO-Link Port 2 |
| C017 | `TP_IOL3_CQ` | 06.Port3_Field | 11.TestPads_IOL | 06 → 11 | MON | Prüfpunkt IO-Link Port 3 |
| C018 | `TP_DI0` | 07.DI_Channel0 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI0 |
| C019 | `TP_DI1` | 07.DI_Channel1 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI1 |
| C020 | `TP_DI2` | 07.DI_Channel2 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI2 |
| C021 | `TP_DI3` | 07.DI_Channel3 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI3 |
| C022 | `TP_DI4` | 07.DI_Channel4 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI4 |
| C023 | `TP_DI5` | 07.DI_Channel5 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI5 |
| C024 | `TP_DI6` | 07.DI_Channel6 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI6 |
| C025 | `TP_DI7` | 07.DI_Channel7 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DI7 |
| C026 | `TP_DO0` | 07.DO_Channel0 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO0 |
| C027 | `TP_DO1` | 07.DO_Channel1 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO1 |
| C028 | `TP_DO2` | 07.DO_Channel2 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO2 |
| C029 | `TP_DO3` | 07.DO_Channel3 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO3 |
| C030 | `TP_DO4` | 07.DO_Channel4 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO4 |
| C031 | `TP_DO5` | 07.DO_Channel5 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO5 |
| C032 | `TP_DO6` | 07.DO_Channel6 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO6 |
| C033 | `TP_DO7` | 07.DO_Channel7 | 11.TestPads_DIO | 07 → 11 | MON | Prüfpunkt DO7 |
| C034 | `LB_RS485A_EN` | 11.LoopbackCtrl | 05.RS485A_TestMux | 11 → 05 | TEST | interner Loopback RS-485 A |
| C035 | `LB_RS485B_EN` | 11.LoopbackCtrl | 05.RS485B_TestMux | 11 → 05 | TEST | interner Loopback RS-485 B |
| C036 | `LB_CAN1_EN` | 11.LoopbackCtrl | 08.CAN_TestMux | 11 → 08 | TEST | interner CAN-Loopback |
| C037 | `LB_IOL_SIM_EN` | 11.LoopbackCtrl | 06.IOLink_TestMux | 11 → 06 | TEST | IO-Link Simulationspfad |
| C038 | `FACTORY_ID_PROG` | 11.FactoryProg | 10.EEPROM_ID_Prog | BIDI | TEST/CFG | Seriennummer / Board-ID Programmierung |

Die Test- und Produktionsmatrix folgt dem im PCB-Plan skizzierten DFT-/Bring-up-Ansatz: Power-Rails, Reset-/Watchdog-Pfade, Feldbusse, IO-Link, DIO sowie Serienprogrammierung sind über ein separates Test-/Factory-Blatt erschlossen. 

---

# D. Kompakte Blatt-zu-Blatt-Übersicht

Falls du zusätzlich eine schnelle Navigationssicht willst, ist hier noch die verdichtete **Sheet-Paar-Matrix** mit den jeweiligen Netzgruppen:

| Quelle | Ziel | Netzgruppen |
|---|---|---|
| 01 | 02 | `5V_MAIN`, `3V3_LOGIC`, `PWR_GOOD`, `DGND` |
| 01 | 03 | `24V_IN_PROT`, `5V_MAIN`, `3V3_LOGIC`, `PWR_GOOD`, `24V_UV_WARN`, `24V_OV_WARN`, `PWR_FAULT_N`, `I_MON_24V`, `TEMP_PWR_WARN`, `DGND` |
| 01 | 04 | `3V3_LOGIC`, `DGND`, `CHASSIS` |
| 01 | 05 | `3V3_LOGIC`, `5V_ISO_A`, `5V_ISO_B`, `DGND`, `CHASSIS` |
| 01 | 06 | `24V_FIELD_A`, `24V_FIELD_B`, `5V_MAIN`, `3V3_LOGIC`, `DGND` |
| 01 | 07 | `24V_FIELD_C`, `3V3_LOGIC`, `DGND` |
| 01 | 08 | `3V3_LOGIC`, `5V_ISO_B`, `DGND`, `CHASSIS` |
| 01 | 09 | `5V_MAIN`, `3V3_LOGIC`, `DGND`, `CHASSIS` |
| 01 | 10 | `3V3_LOGIC`, `DGND` |
| 03 | 01 | `EN_5V_MAIN`, `EN_3V3_LOGIC`, `EN_ISO_A/B`, `EN_FIELD_A/B/C` |
| 03 | 02 | `SYS_RESET_N`, `SOM_RESET_N`, `SAFE_MODE_REQ`, `FORCE_RECOVERY_N` |
| 02 | 03 | `SOM_HEARTBEAT`, `WDOG_KICK`, `SAFE_MODE_ACK` |
| 03 | 07 | `DIO_BANK0_MODE`, `DIO_BANK1_MODE` |
| 03 | 09 | `STATUS_LED_G/Y/R` |
| 03 | 10 | `EVENT_LOG_INT` |
| 09 | 03 | `FORCE_RECOVERY_REQ_N`, `SWDIO`, `SWCLK`, `NRST_MCU` |
| 02 | 04 | `ETH1_*`, `ETH2_*`, `ETHx_LED_*` |
| 02 | 05 | `RS485A_*`, `RS485B_*` |
| 05 | 03 | `RS485_FAULT_SUM_N` |
| 02 | 06 | `IOL_SPI_*`, `IOL_CS*`, `IOL_RESET_N` |
| 06 | 02 | `IOL_SPI_MISO`, `IOL_INT0_N`, `IOL_INT1_N` |
| 06 | 03 | `IOLx_OC_N`, `IOLx_TSD_N` |
| 07 | 02 | `DI_SENSE0..7`, `DO_FLT_N0..7` |
| 02 | 07 | `DO_CMD0..7`, `DO_EN0..7` |
| 07 | 03 | `DO_FLT_N0..7` |
| 02 | 08 | `CAN1_TXD`, `CAN1_STB_N`, `CAN1_TERM_EN` |
| 08 | 02 | `CAN1_RXD` |
| 08 | 03 | `CAN1_FAULT_N` |
| 02 | 09 | `USB0_D_P/N`, `UART_DBG_TX` |
| 09 | 02 | `UART_DBG_RX`, `BOOT_MODE0..2` |
| 02 | 10 | `SPI_TPM_SCLK/MOSI/CS_N`, `I2C_SEC_SCL/SDA` |
| 10 | 02 | `SPI_TPM_MISO`, `TPM_IRQ_N`, `RTC_INT_N`, `BOARD_ID0..3` |
| 01/02/03/05/06/07/08 | 11 | `TP_*` Monitoring-Netze |
| 11 | 05/06/08/10 | `LB_*`, `FACTORY_ID_PROG` |
