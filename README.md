# ESPHome Smart Meter Reader für gPlugE (WT32-ETH01)

Dieses Repository enthält eine ESPHome-Konfiguration für das **gPlugE** Modul (basierend auf dem **WT32-ETH01** ESP32-Board). Es ist speziell für das Auslesen von Smart Metern über die P1-Schnittstelle mittels des DSMR-Protokolls konzipiert.

Ergänzen: Wieso esphome verwendet wurde:
- Besser in Home Assistant integriert
- Kein Umweg über MQTT
- Verständlichere Scripting Möglichkeiten
- Grosse Communiy, viele Unterstützte Geräte.

## ⚡ Unterstützte & Getestete Hardware
Die Konfiguration wurde mit folgender Hardware getestet:
* **Stromzähler:** [Ensor eRS801](https://www.ensor.com/produkte)
* **Schnittstelle:** P1-Kundenschnittstelle (RJ12)
* **Modul:** [gPlugE (Ethernet)](https://gplug.ch/)
* **Protokoll:** DSMR (115.200 Baud)

## 🔍 Migration von Tasmota zu ESPHome

### P1-Port Identifizierung
In Tasmota wurde die Schnittstelle oft über ein SML-Skript definiert. Eine typische Zeile sah so aus:
`>M 1 +1,5,o,0,115200,z`
* Die Zahl **5** steht für den Hardware-Pin **GPIO 5**.
* In ESPHome wird dieser Pin in der Sektion `uart:` als `rx_pin: GPIO 5` übernommen.


## Funktionsumfang & Anzeige
Die Konfiguration nutzt den modernen **Web-Server (v3)** mit strukturierten Gruppen und Icons für eine übersichtliche Live-Ansicht:

1. **Total:**
   - **Aktueller Verbrauch & Einspeisung:** Anzeige in Watt (W) statt Kilowatt (kW) für bessere Auflösung.
   - **Tagesbezug:** Ein intelligenter Sensor (`total_daily_energy`), der den Energiebezug seit Mitternacht summiert und täglich automatisch zurücksetzt.
   - **Aktueller Tarif:** Automatische Übersetzung des Zählercodes in Klartext ("Hochtarif (HT)" / "Niedertarif (NT)").
   - **Energie Bezug Total:** Berechneter Gesamtzählerstand (HT + NT).

2. **Phase L1 / L2 / L3:**
   - Einzelauswertung von Leistung (W), Spannung (V) und Stromstärke (A) für jede Phase.

3. **Diagnose:**
   - Zähler-Identifikation und P1-Protokoll-Version.
   - Netzwerkdetails: Ethernet IP-Adresse, MAC-Adresse und der vollständige DNS-Hostname (FQDN).
   - ESPHome Firmware-Version und Zeitpunkt des letzten Neustarts.

## Bekannte Probleme
Ergänzen:
* Stromtarif wird zumindest bei meinem Zähler nicht ausgelesen. Es steht weiterhin "Warte auf Daten..."

## Relevante Ressourcen & Links

* **Hersteller gPlug:** [gplug.ch](https://gplug.ch/)
* **Offizielle Installationsanleitung gPlugE:** [Anleitung ansehen](https://gplug.ch/installationsanleitung/gpluge/)
* **Ensor Smart Meter Produkte:** [ensor.com/produkte](https://www.ensor.com/produkte)
* **ESPHome Dokumentation:** [DSMR Sensor](https://esphome.io) | [Ethernet Component](https://esphome.io)
* **Tasmota Dokumentation:** [SML Interface](https://tasmota.github.io) (für Vergleichszwecke)

## 💡 Wichtige Informationen
* **Konnektivität:** Diese Konfiguration ist auf **reinen Ethernet-Betrieb** optimiert. WiFi ist deaktiviert, um die Stabilität zu erhöhen und Funkstörungen zu vermeiden.
* **Einheiten:** Alle Leistungswerte werden mittels Filtern von kW in Watt umgerechnet.

