# Konzeption und Design eines Smart Desktop Communicators

# Projektziel 

Entwicklung eines Smart Desktop Communicators zur Sprachkommunikation über Ethernet oder WLAN.

Das System ermöglicht Audioein- und -ausgabe über Mikrofon und Lautsprecher und bietet Bedienelemente sowie Statusanzeigen zur Benutzerinteraktion.

Ziel ist die Umsetzung einer Hardwarelösung mit geeignetem Mikrocontroller inklusive Schaltplan- und PCB-Design unter Berücksichtigung von Signalverarbeitung, Energieversorgung und sauberem Hardwarelayout.

# Systemarchitektur

Als zentrale Steuereinheit des Systems wird der STM32L401CC6 verwendet. Im Gegensatz zur Nutzung eines fertigen Entwicklungsboards wird der Mikrocontroller direkt als eigenständiger Chip in das PCB-Design integriert (siehe Abbildung 1).

Dadurch ist die vollständige Minimalbeschaltung des Mikrocontrollers erforderlich, um einen stabilen Betrieb sicherzustellen.

Die implementierte Schaltung umfasst:
• Spannungsversorgung des Mikrocontrollers mit 3,3 V  
• Entkopplungskondensatoren zur Spannungsstabilisierung  
• Reset-Schaltung  
• Boot-Konfiguration  
• SWD-Debug-Schnittstelle zum Programmieren  
• Taktversorgung durch externen Oszillator  
• Verbindung zu den Peripherie-Komponenten  

Der STM32 übernimmt die zentrale Steuerung des Systems. Das vom INMP441 aufgenommene Audiosignal wird an den Mikrocontroller übertragen und dort verarbeitet. Die Audiodaten werden anschließend über UART an den RN 171  weitergegeben, welcher als WLAN-Schnittstelle dient und die Daten an das Netzwerk überträgt.

Für die Audioausgabe wird der PAM8302AAY Class-D Audioverstärker eingesetzt. Dieser verstärkt das Ausgangssignal des Systems und treibt den angeschlossenen 8 Ω 50 mm Lautsprecher (Visaton K 50) zur Wiedergabe des Audiosignals an.

## Stromversorgung und Power-Management

Die Stromversorgung des Systems erfolgt über USB-C sowie einen Akku für den mobilen Betrieb. Ein TP4056 Lade-IC übernimmt das Laden des Akkus sowie die Energieverwaltung zwischen USB-Betrieb und Batterieversorgung.

Ein Spannungsregler stellt eine stabile 3,3 V Versorgung für alle digitalen Komponenten sicher und gewährleistet einen zuverlässigen Betrieb des Gesamtsystems.

## Auswahl der Systemkomponenten

### Mikrocontroller

Als zentrale Steuereinheit wird der STM32 eingesetzt. Dieser bietet eine zuverlässige Echtzeitverarbeitung, was besonders für die Audioverarbeitung im System wichtig ist. Zusätzlich passt er gut zu den im Studium bereits gesammelten Erfahrungen im Umgang mit STM32-basierten Systemen.



### Mikrofon

Für die Audioaufnahme wird das INMP441 MEMS-Mikrofon mit I2S-Schnittstelle verwendet. Dieses ist bereits vollständig integriert und benötigt keine zusätzliche analoge Beschaltung. Dadurch wird das PCB-Design vereinfacht und eine direkte digitale Anbindung an den STM32 ermöglicht.



### Lautsprecher

Für die Audioausgabe wird ein PAM8302AAY Class-D Verstärker eingesetzt. Dieser verstärkt das Ausgangssignal des Systems und ermöglicht den Betrieb eines externen Lautsprechers. Dadurch kann das vom Mikrocontroller bereitgestellte Audiosignal ausreichend verstärkt und sauber wiedergegeben werden.



## Power Management

Das System wird über einen USB-C-Anschluss mit 5 V versorgt. Die CC1- und CC2-Pins sind jeweils über 5,1 kΩ Widerstände mit GND verbunden, wodurch eine korrekte USB-C-Erkennung und Aktivierung der Versorgungsspannung (VBUS) gewährleistet wird.

Zur Energieverwaltung wird ein TP4056 Lade-IC eingesetzt, der das Laden eines angeschlossenen Li-Ion/Akkus über USB-C ermöglicht. Der Ladestrom wird über den PROG-Pin eingestellt. Der TEMP-Pin wird nicht verwendet.

Der Akkubetrieb ist über eine einfache Power-OR-Schaltung mit einer Schottky-Diode realisiert. Dadurch kann das System sowohl über USB als auch über den Akku betrieben werden, ohne Rückspeisung zwischen den Energiequellen.

Die nachgelagerte Spannungsregelung erzeugt eine stabile 3,3 V Versorgung für alle digitalen Komponenten (STM32, ESP32, Audio-Peripherie). Ein Spannungsregler übernimmt diese Aufgabe, während Stützkondensatoren (100 nF und 10 µF) am Ein- und Ausgang für Spannungsstabilität und Störunterdrückung sorgen.

Die USB-Schirmung ist mit der Systemmasse verbunden, um elektromagnetische Störungen zu reduzieren und die Signalintegrität zu verbessern.



## Spannungsversorgung

Alle VDD- und VBAT-Pins sind mit der 3,3 V Versorgung verbunden, während VSS und VSSA auf Masse (GND) gelegt sind.

Zur Stabilisierung der Versorgungsspannung wurden mehrere 100 nF Entkopplungskondensatoren nahe an den Versorgungspins platziert. Diese filtern hochfrequente Störungen und reduzieren Spannungsschwankungen, wodurch eine stabile Versorgung und eine verbesserte Signalintegrität gewährleistet wird.


### Interne Spannungsregelung (VCAP)

Der Pin VCAP1 ist mit einem 2,2 µF Kondensator gegen Masse beschaltet. Dieser ist Teil der internen Spannungsregelung des Mikrocontrollers, welche die reduzierte Kernspannung für den Prozessor bereitstellt.

Der externe Kondensator dient zur Stabilisierung dieser internen Versorgung und gewährleistet einen zuverlässigen Betrieb des Mikrocontrollers.


### Reset-Schaltung (NRST)

Der NRST-Pin dient zum Zurücksetzen des Mikrocontrollers. Er ist über einen Pull-Up-Widerstand mit 3,3 V verbunden und zusätzlich über einen Taster nach GND geführt.

Der Pull-Up sorgt für einen definierten High-Pegel im Normalbetrieb. Beim Betätigen des Tasters wird NRST auf LOW gezogen, wodurch ein Reset des Mikrocontrollers ausgelöst wird.

---

### Taktversorgung

Zur präzisen Takterzeugung wird ein externer Quarz zwischen den Pins PH0 und PH1 verwendet. Ergänzend sind zwei 22 pF Kondensatoren gegen Masse geschaltet.

Diese dienen zur Stabilisierung der Quarzschwingung und gewährleisten einen sauberen und stabilen Systemtakt.

---

### SWD Programmier- und Debug-Schnittstelle

Für Programmierung und Debugging wird die SWD-Schnittstelle verwendet. Dabei werden folgende Signale genutzt:

- PA13 → SWDIO  
- PA14 → SWCLK  
- GND  
- 3,3 V Referenzspannung  

Diese Schnittstelle ermöglicht das direkte Flashen und Debuggen des Mikrocontrollers.

---

### Boot-Konfiguration (BOOT0)

Der BOOT0-Pin ist über einen 10 kΩ Pull-Down-Widerstand mit GND verbunden. Dadurch startet der STM32L401CCU6 nach dem Einschalten immer aus dem internen Flash-Speicher und führt direkt die programmierte Firmware aus.

Diese Beschaltung verhindert einen unbeabsichtigten Start im Bootloader-Modus und erhöht die Zuverlässigkeit des Systems.


## Digitale Steuerung und Zustandsanzeige

Zur Steuerung des Systems wird ein Taster an einen GPIO-Pin des STM32 angeschlossen. Zusätzlich werden mehrere LEDs verwendet, um unterschiedliche Betriebszustände des Systems anzuzeigen.



### Taster (Eingang)

Der Taster ist mit einem Pull-Up-Widerstand beschaltet und gegen GND geschaltet. Dadurch liegt der Eingang im Ruhezustand auf einem definierten High-Pegel. Beim Betätigen des Tasters wird der GPIO-Pin auf GND gezogen, wodurch ein LOW-Pegel erkannt wird.

Diese Pull-Up-Beschaltung verhindert undefinierte Eingangszustände und reduziert die Störanfälligkeit des Eingangs.

Der Taster wird zur Zustandsumschaltung innerhalb der Systemlogik verwendet, sodass zwischen verschiedenen Betriebsmodi gewechselt werden kann.



### LED-Zustandsanzeige

Zur Visualisierung der Systemzustände sind mehrere LEDs an GPIO-Pins des Mikrocontrollers angeschlossen.

- LED D2 ist mit PC13 verbunden und signalisiert den Aufnahmezustand des Systems.  
- LED D1 ist mit PB0 verbunden und zeigt den inaktiven bzw. ausgeschalteten Zustand an.  

Zusätzlich ist ein Fehlerzustand implementiert, der Betriebs- oder Verarbeitungsfehler signalisiert.

Alle LEDs sind jeweils über Vorwiderstände angeschlossen, um den Strom zu begrenzen und die Bauteile vor Beschädigung zu schützen.

## INMP441 MEMS-Mikrofon

Das INMP441 basiert auf einem MEMS-Prinzip (Micro-Electro-Mechanical System). Im Inneren befindet sich eine mikromechanische Membran, die durch eintreffenden Schalldruck in Schwingung versetzt wird.

Diese mechanischen Schwingungen werden intern erfasst und durch integrierte Signalverarbeitung in ein elektrisches Signal umgewandelt. Anschließend erfolgt die direkte Digitalisierung innerhalb des Mikrofonmoduls durch einen integrierten ADC.

Dadurch steht bereits ein vollständig digitales Audiosignal zur Verfügung, sodass keine externe analoge Signalverarbeitung erforderlich ist.

---

### I2S-Schnittstelle

Das INMP441 stellt die Audiodaten über eine digitale I2S-Schnittstelle (Inter-IC Sound) bereit. I2S ist ein serielles Protokoll zur Übertragung von Audiodaten zwischen integrierten Schaltungen.

Über diese Schnittstelle werden die Audiodaten direkt an den STM32 übertragen. Dadurch entfallen externe ADCs sowie analoge Vorverstärker.

---

### Systemintegration

Das Mikrofon wurde wie folgt in das System integriert:

Die Versorgung erfolgt mit 3,3 V, da das Modul für diesen Spannungsbereich ausgelegt ist. Zwischen VDD und GND befindet sich ein Entkopplungskondensator zur Stabilisierung der Versorgungsspannung und zur Reduktion von Störungen.

- **WS (Word Select)** → PB12  
  Dient zur Kanal-Synchronisation (Links/Rechts-Zuordnung der Audiodaten).

- **SCK (Serial Clock)** → PB13  
  Wird vom Mikrocontroller bereitgestellt und definiert den Takt für die Datenübertragung.

- **SD (Serial Data)** → PB15  
  Über diese Leitung sendet das Mikrofon die digitalen Audiodaten an den STM32.

- **L/R-Pin** → GND  
  Konfiguration als linker Audiokanal. Da nur ein Mikrofon verwendet wird, ist diese Einstellung ausreichend.

  ## Audioverstärkung mit dem PAM8302A

Für die Audioausgabe wird der PAM8302A Class-D Verstärker verwendet. Dieser verstärkt das vom STM32L401CCU6 bereitgestellte Audiosignal, da die Ausgangsleistung des Mikrocontrollers nicht ausreicht, um einen Lautsprecher direkt anzusteuern.

Der Verstärker wird mit 3,3 V versorgt. Zur Stabilisierung der Versorgungsspannung sind 100 nF und 10 µF Kondensatoren zwischen VDD und GND platziert, um Störungen und Spannungsschwankungen zu reduzieren.

Das Audiosignal wird über einen 10 nF Koppelkondensator an den Eingang IN+ geführt, um Gleichspannungsanteile zu blockieren. IN− ist mit GND verbunden, wodurch der Verstärker im Single-Ended-Betrieb arbeitet.

Der SD-Pin ist ebenfalls auf GND gelegt, sodass der Verstärker dauerhaft aktiviert ist. Der Ausgang OUT+ und OUT− ist direkt mit dem Lautsprecher verbunden und arbeitet im differentiellen Betrieb für höhere Leistung und bessere Störsicherheit.

### WLAN -schnittstelle 

Für die Drachtlose Kommunikation wird das RN171 WLAN-Modul verwendet. Das Modul bietet eine integriete WLAN - schnittstelle und kann über UART einfach mit dem STM32 verbunden werden 
Die erforderliche Beschaltung und Ansteuerung wurde auf Basis Herstellerdokumentation Umgesetzt . da das Modul ausschließlich für die Netzwerkkommnikation genutzt wird , bleibt die zentral steuerung vollständig bein stm32
#Quelle
123microcontroller.com
diode.com
https://www.snapeda.com/parts/RN-171/Microchip+Technology/view-part/?ref=snap
https://www.snapeda.com/parts/INMP441/TDK/view-part/?ref=snap
