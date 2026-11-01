# ESPHome Konfiguration um den Stromzähler mit dem gPlugE IoT-Adapter auszulesen.

Dieses Repository enthält ESPHome-Konfiguration für das **gPlugE** Modul. Basierend auf dem **WT32-ETH01** (ESP32 mit nativem Ethernet) ist dieses Setup speziell für das Auslesen von Smart Metern über die P1-Kundenschnittstelle mittels des DSMR-Protokolls konzipiert.

| Konfiguration | gPlug | Schnittstelle / Protokoll | Zähler |
|---|---|---|---|
| [gPlugE-DSMR-P1.yaml](configs/gPlugE-DSMR-P1.yaml) | gPlugE (WT32-ETH01, Ethernet) | P1 / DSMR | Ensor eRS801 u.a. |
| [gPlugM-DLMS-E450.yaml](configs/gPlugM-DLMS-E450.yaml) | gPlugM (ESP32-C3, WLAN) | M-Bus CII / DLMS-COSEM | Landis+Gyr E450 / E570 – siehe [gPlugM](#-gplugm-m-bus--dlms-zb-landisgyr-e450) |

## 🚀 Warum ESPHome statt Tasmota?
Die Migration von Tasmota zu ESPHome bietet signifikante Vorteile für moderne Smart-Home-Umgebungen:

*   **Native Home Assistant Integration:** Nahtlose Anbindung über das ESPHome-Dashboard; Sensoren erscheinen automatisch mit korrekten Einheiten, Icons und Geräteklassen.
*   **Kein MQTT-Umweg:** Direkte Kommunikation über die native API – das reduziert Latenzen, erhöht die Zuverlässigkeit und macht einen MQTT-Broker überflüssig.
*   **Mächtiges Scripting:** Flexiblere Möglichkeiten zur Datenverarbeitung (Lambdas) direkt auf dem Gerät (z. B. für die Berechnung von Gesamtwerten oder Tarif-Übersetzungen).
*   **Modernes Web-Interface:** Nutzung des Web-Servers v3 mit gruppierter Anzeige für eine übersichtliche Live-Kontrolle direkt im Browser.

## ⚡ Unterstützte & Getestete Hardware
*   **Modul:** [gPlugE (Ethernet-Version)](https://gplug.ch) – Hardware-Basis: WT32-ETH01.
*   **Stromzähler:** [Ensor eRS801](https://www.ensor.com) (Schweizer Standard), sowie kompatible DSMR-Zähler (z.B. Landis+Gyr, Kaifa).
*   **Schnittstelle:** P1-Kundenschnittstelle (RJ12 Anschluss).
*   **Protokoll:** DSMR (Voreingestellt: 115.200 Baud, unverschlüsselt).

## 🛠 Installation & Konfiguration

### 1. Konfigurationsdatei anpassen
Die Hauptdatei ist [gPlugE-DSMR-P1.yaml](configs/gPlugE-DSMR-P1.yaml). In der Regel müssen **keine Änderungen** im eigentlichen Code vorgenommen werden; es reicht aus, wenn die Variablen unter `substitutions` im oberen Bereich der Datei angepasst werden.

### 2. secrets.yaml vorbereiten
ESPHome nutzt eine externe `secrets.yaml` für sensible Daten. Erstelle diese Datei in deinem ESPHome-Verzeichnis mit folgendem Inhalt:

```yaml
# Passwort für den Webserver-Zugriff (Browser-Login)
web_server_password: "DeinSicheresPasswort"

# Das aktuell auf dem Gerät befindliche OTA-Passwort 
# (Bei Erstinstallation nach Tasmota identisch mit ota_password setzen)
gpluge_ota_password: "DeinAltesPasswort"

# Das Passwort, das für zukünftige Updates gesetzt werden soll
ota_password: "DeinNeuesPasswort"

# Der Verschlüsselungsschlüssel für die Home Assistant API (32 Zeichen Base64)
api_key: "DEIN_GENERIERTER_API_KEY"
```
*Hinweis: Die Konfiguration enthält eine Logik, die beim Booten automatisch das OTA-Passwort aktualisiert, falls `ota_password` und `gpluge_ota_password` sich unterscheiden.*

### 3. Flashen der Firmware
Es gibt zwei Wege, um von Tasmota zu ESPHome zu wechseln:

*   **Via Tasmota Web-Interface (OTA):** Du kannst die in ESPHome kompilierte `.bin` Datei direkt im Tasmota-Update-Menü hochladen. 
    *   *Wichtig:* Gib dem Gerät nach dem Upload ca. 2-3 Minuten Zeit. Der ESP32 muss das Dateisystem neu organisieren. Trenne in dieser Zeit nicht den Strom!
*   **Via USB (Empfohlen):** Verbinde das Modul per USB-Seriell-Adapter mit deinem Computer und nutze das [ESPHome Web Tool](https://web.esphome.io), um die Firmware direkt aus dem Browser zu installieren. Dies ist der sicherste Weg, um die korrekte Partitionierung sicherzustellen.

## 📋 Angezeigte Informationen & Sensoren

Folgende Informationen werden vom Stromzähler ausgelesen, im Webinterface des gPlugE dargestellt und nativ an Home Assistant übermittelt.

![Screenshot gPlugE Webinterface](img/gPlugE-Screenshot.png)

### 1. Energie-Statistiken
*   **Aktueller Verbrauch (W):** Die aktuelle Last in Watt.
*   **Energie Bezug Total (1.8.0):** Ein berechneter Sensor (Summe aus HT und NT) zur Überwachung des Gesamtzählerstands.
*   **Energie Bezug HT/NT (kWh):** Separate Zählerstände für Hochtarif (1.8.1) und Niedertarif (1.8.2).
*   **Energie Bezug Heute (kWh):** Ein täglicher Summenzähler, der sich automatisch um Mitternacht zurücksetzt.

### 2. Phasen-Analyse (L1, L2, L3)
Für jede der drei Phasen wird separat ausgelesen:
*   **Leistung (W):** Aktuelle Wirkleistung pro Phase.
*   **Spannung (V):** Netzspannung zur Überwachung der Versorgungsqualität.
*   **Strom (A):** Aktuelle Stromstärke pro Phase.

### 3. Netzqualität & Stabilität
*   **Stromausfälle:** Zähler für registrierte lange Stromausfälle.
*   **Spannungsereignisse:** Überwachung von Spannungseinbrüchen (*Sags*) und Spannungsspitzen (*Swells*) pro Phase.
*   **Fehler-Log:** Auslesen des internen Status-Logs des Smart Meters.

### 4. Geräte-Diagnose
*   **Zähler-Identifikation:** Offizielle Identifikationsnummer des Stromzählers.
*   **DSMR-Version:** Version des verwendeten P1-Protokolls.
*   **System-Status:** CPU-Temperatur des ESP32, Uptime des gPlugE sowie Netzwerkdetails (IP- & MAC-Adresse).

## ⚠️ Bekannte Probleme & Hinweise

### 1. Tarif-Erkennung (HT/NT)
Smart Meter geben den aktuellen Tarif über die P1-Schnittstelle oft als numerischen Rohwert aus. Die in dieser Konfiguration hinterlegte Logik übersetzt diese Werte automatisch in Klartext.
*   **Standard-Logik:** In der Regel entspricht Rohwert `0002` dem Hochtarif (HT) und `0001` dem Niedertarif (NT).
*   **Anpassung:** Sollte Ihr Zähler abweichende Werte senden (z. B. `5432` / `5431`), zeigt das Webinterface "Unbekannter Tarif" an. In diesem Fall muss der Rohwert im YAML-Skript unter `text_sensor` im entsprechenden `lambda`-Block angepasst werden (was in meinem Fall für `5432` / `5431` durchgeführt wurde).

### 2. Konnektivität & Stabilität
*   **Reiner Ethernet-Betrieb:** Diese Konfiguration ist auf den Betrieb über das LAN-Kabel optimiert. WLAN ist bewusst deaktiviert, um eine maximale Verbindungsstabilität zu garantieren.

### 3. P1-Schnittstelle & Pegel
*   **Invertierung:** Je nach Zählermodell und verwendetem Lesekopf muss das Signal invertiert werden. Die Konfiguration nutzt den Standard-UART-Modus. Sollten keine Daten ankommen, prüfen Sie, ob Ihr Lesekopf eine Hardware-Invertierung besitzt oder ob der `rx_pin` im YAML auf `inverted: true` gesetzt werden muss.
*   **Baudrate:** Der Standard für DSMR ist 115.200 Baud. Ältere Zähler (z. B. nach DSMR 2.x oder 3.x Standard) benötigen eventuell eine Reduktion auf 9.600 Baud.

## 🔌 gPlugM (M-Bus / DLMS, z.B. Landis+Gyr E450)

> **Stand:** Getestet mit einem Landis+Gyr E450 bei ewb (unverschlüsselt, gespeist vom Zähler). Benötigt **ESPHome 2026.11** oder neuer.

Der [gPlugM](https://gplug.ch/produkte/gplugm/) wird mit dem RJ12-Kabel an die **M-Bus-Kundenschnittstelle (CII)** eines Landis+Gyr E450 oder E570 angeschlossen und von dieser auch gespeist. Der Zähler sendet DLMS/COSEM-Telegramme, die ESPHome mit der eingebauten Komponente [`dlms_meter`](https://esphome.io/components/sensor/dlms_meter/) dekodiert.

### Hardware
| | |
|---|---|
| **Chip** | ESP32-C3 (USB-C, WLAN 2.4 GHz) |
| **Schnittstelle** | M-Bus CII, 2400 Baud, 8E1, RX auf **GPIO7** (invertiert) |
| **RGB-LED** | rot GPIO1, grün GPIO4, blau GPIO3 (blau = Status-LED) |
| **Taster** | GPIO9 |
| **Speisung** | über den M-Bus des Zählers, dauerhaft max. 10 mA bei 24 V (240 mW): CPU mit 80 MHz, WLAN im Stromsparmodus |

### Voraussetzungen
*   **ESPHome 2026.11 oder neuer:** Der E450 sendet ein "selbstbeschreibendes" Telegramm (zuerst alle OBIS-Kennzahlen, dann alle Werte). Erst die mit ESPHome 2026.11 ausgelieferte Version von `dlms_parser` kann es dekodieren – mit älteren Versionen sind alle Werte 0. Die Konfiguration verlangt deshalb `min_version: 2026.11.0`.
*   **Kundenschnittstelle freischalten:** Die CII ist ab Werk deaktiviert und muss beim Netzbetreiber (z.B. Kundendienst ewb) aktiviert werden.
*   **Verschlüsselung:** Je nach Netzbetreiber sind die Daten AES-verschlüsselt. Den Schlüssel (32 Hex-Zeichen) liefert der Netzbetreiber; bei einem gPlugM mit Tasmota steht er im Script als `dKEY="..."`. **Vor dem Flashen notieren** – er geht sonst verloren. Ein leeres `dKEY=""` bedeutet: unverschlüsselt (so bei ewb).

### Konfiguration
Die Datei [gPlugM-DLMS-E450.yaml](configs/gPlugM-DLMS-E450.yaml) ist auf **ewb (Energie Wasser Bern)** voreingestellt. Für andere Netzbetreiber genügen in der Regel die `substitutions`:

| Substitution | Standard (ewb) | Bedeutung |
|---|---|---|
| `obis_energy_channel` | `1.1` | Kanal der Energiezähler. ewb liefert `1-1:1.8.0`, viele andere `1-0:1.8.0` → `1.0` |

Zusätzlich zur `secrets.yaml` oben braucht der gPlugM:

```yaml
# WLAN (der gPlugM hat kein Ethernet)
wifi_ssid: "DeinWLAN"
wifi_password: "DeinWLANPasswort"

# Das aktuell auf dem gPlugM befindliche OTA-Passwort (bei Erstinstallation = ota_password)
gplugm_ota_password: "DeinAltesPasswort"

# Nur bei verschlüsselter Kundenschnittstelle (und `decryption_key` im YAML aktivieren)
gplugm_decryption_key: "0123456789ABCDEF0123456789ABCDEF"
```

### Flashen
Tasmota und ESPHome verwenden auf dem ESP32-C3 eine unterschiedliche Flash-Aufteilung, ein Update über das Tasmota-Webinterface ist deshalb **nicht möglich**. Die Erstinstallation erfolgt einmalig über **USB-C** (der gPlugM hat einen eingebauten USB-Anschluss, kein Adapter nötig):

1.  Schlüssel (`dKEY`) aus Tasmota sichern, siehe oben. Wer zu Tasmota zurück möchte, sichert vorher den ganzen Flash: `esptool read-flash 0 ALL gplugm-tasmota.bin`.
2.  gPlugM vom Zähler trennen und per USB-C mit dem Computer verbinden (Kabel mit Datenleitungen). Wird kein Port gefunden: Taster gedrückt halten, USB einstecken, Taster loslassen (Download-Modus).
3.  `esphome run gPlugM-DLMS-E450.yaml` (oder [ESPHome Web Tool](https://web.esphome.io)) und den USB-Port des gPlugM wählen.
4.  Wieder an den Zähler anschliessen. Alle weiteren Updates laufen über WLAN.

### Angezeigte Informationen & Sensoren
Der E450 sendet alle ~10 s ein Telegramm mit allen Werten:

| Gruppe | Werte |
|---|---|
| Total | Aktueller Verbrauch und Einspeisung (W), Aktueller Tarif (HT/NT), Energie Bezug Heute, Energie Bezug/Einspeisung Total (kWh) |
| Phasen | Verbrauch und Einspeisung L1–L3 (W), Spannung L1–L3 (V), Strom L1–L3 (A) |
| Stromzähler | Zähler Identifikation (z.B. `LGZ1030665999266`) |
| Diagnose | CPU Temperatur, WLAN-Signal, ESPHome-Version, IP & MAC, Taster |

Ebenfalls im Telegramm, aber nicht in der Konfiguration: Blindenergie (`1-1:3.8.0` / `1-1:4.8.0`, varh), Blindleistung pro Phase (`1-0:23.7.0` … `1-0:64.7.0`, var) und die Kundennachricht (`0-0:96.13.0`). Bei Bedarf als weitere `dlms_meter`-Sensoren ergänzen.

### HT/NT-Zählerstände
Der E450 sendet bei ewb **keine getrennten Zählerstände** für Hoch- und Niedertarif (1.8.1/1.8.2), sondern nur das Total und den aktuellen Tarif als Text (`RATE1` = HT, `RATE2` = NT von 22 bis 6 Uhr). Die Aufteilung übernimmt Home Assistant mit einem [Utility Meter](https://www.home-assistant.io/integrations/utility_meter/) (Entity-IDs ggf. anpassen):

```yaml
utility_meter:
  strom_bezug:
    source: sensor.gplugm_smartmeter_energie_bezug_total_1_8_0
    tariffs: [HT, NT]

automation:
  - alias: "Stromtarif umschalten"
    triggers:
      - trigger: state
        entity_id: sensor.gplugm_smartmeter_aktueller_tarif
    actions:
      - action: select.select_option
        target:
          entity_id: select.strom_bezug
        data:
          option: "{{ 'NT' if 'NT' in trigger.to_state.state else 'HT' }}"
```

### Fehlersuche
*   **Alle Werte 0:** ESPHome ist älter als 2026.11 (siehe Voraussetzungen).
*   **Keine Werte:** `log_level` auf `VERBOSE` und unter `logger: logs:` `dlms_meter: VERBOSE` setzen. Das Log zeigt dann jede empfangene OBIS-Kennzahl.
*   **Unlesbare Telegramme:** Die Schnittstelle ist verschlüsselt → `decryption_key` setzen.
*   **Energiewerte fehlen, Leistung kommt:** `obis_energy_channel` von `1.1` auf `1.0` ändern (oder umgekehrt).
*   **Werte kurz auf 0, Energie-Dashboard zeigt tausende kWh:** Der E450 legt um hh:58 mitten im Telegramm eine Pause ein. `receive_timeout: 3s` (im YAML gesetzt) wartet sie ab; ohne wird ein halbes Telegramm als 0 ausgewertet und Home Assistant verbucht den ganzen Zählerstand als Verbrauch. Bereits falsch erfasste Statistiken lassen sich unter *Entwicklerwerkzeuge → Statistiken* korrigieren.
*   **Werte um Faktor 10/100/1000 falsch:** Filter (`multiply`) anpassen.

### Quellen
*   Pinbelegung und Schnittstellenparameter: [jluthiger/esphome-gplug](https://github.com/jluthiger/esphome-gplug)
*   OBIS-Kennzahlen von ewb: [jmesserli/ewb-smartmeter-homeassistant](https://github.com/jmesserli/ewb-smartmeter-homeassistant)
*   Dekodierung des selbstbeschreibenden Telegramms: [esphome-libs/dlms_parser#12](https://github.com/esphome-libs/dlms_parser/issues/12)

## 🤝 Beteiligung
Solltet ihr einen anderen gPlug oder Stromzähler im Einsatz und die ESPHome Konfiguration dazu erstellt haben, könnt/sollt/dürft ihr diese gerne hier teilen. Entweder als Pull-Request oder ihr schickt es mir per Email an `haribert`at`gmx`dot`ch`, dann nehme ich diese unter `configs` auf.

## 🔗 Relevante Ressourcen & Links
*   **Hersteller gPlug:** [gplug.ch](https://gplug.ch)
*   **Ensor Smart Meter Produkte:** [ensor.com](https://www.ensor.com)
*   **ESPHome Dokumentation:** [DSMR Sensor](https://esphome.io) | [Ethernet Component](https://esphome.io) | [DLMS Meter](https://esphome.io/components/sensor/dlms_meter/)
*   **Tasmota Dokumentation:** [SML Interface](https://tasmota.github.io) (für Vergleichszwecke)

