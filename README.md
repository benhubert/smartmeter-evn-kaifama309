# Alle Infos zu diesem Projekt befinden sich auf meinem Blog
https://www.michaelreitbauer.at/blog


# SmartMeterEVN
Dieses Projekt ermöglicht es den Smartmeter der EVN (Netz Niederösterreich) über die Kundenschnittstelle auszulesen.
Smart Meter werden von der Netz NÖ GmbH eingebaut, auf Basis der gesetzlichen Forderungen.

## Getting Started

### Voraussetzungen Hardware


* Passwort für die Kundenschnittstelle
  * Alle folgenden Informationen sind aus dem Folder der EVN. (https://www.netz-noe.at/Download-(1)/Smart-Meter/218_9_SmartMeter_Kundenschnittstelle_lektoriert_14.aspx)
  * Wenn bereits ein Smart Meter in der Kundenanlage eingebaut ist, kann hier das der Schlüssel angefordert werden: smartmeter@netz-noe.at
    * Kundennummer oder Vertragskontonummer
    * Zählernummer
    * Handynummer

### Voraussetzungen Software

* Python 3, plus Libraries (siehe "requirements.txt")
    
    Installation der Python Libraries:
    ```
    pip install -r requirements.txt
    ```

### Konfiguration

Die Konfiguration erfolgt über Umgebungsvariablen (siehe `smartmeter.env.example`):

| Variable | Standard | Beschreibung |
|---|---|---|
| `EVN_KEY` | – (Pflicht) | Schlüssel der Kundenschnittstelle |
| `MQTT_ENABLED` | `false` | MQTT verwenden |
| `MQTT_BROKER` | `192.168.1.10` | IP Adresse des Brokers, ohne Port (Port 1883) |
| `MQTT_USER` / `MQTT_PASSWORD` | leer | MQTT Zugangsdaten |
| `PRINT_VALUES` | `true` | Aktuelle Werte auf der Console ausgeben |
| `SERIAL_PORT` | `/dev/ttyUSB0` | Serielle Schnittstelle |

Manuell starten:
```
cp smartmeter.env.example smartmeter.env   # ausfüllen, wird von git ignoriert
set -a; . ./smartmeter.env; set +a
python EvnSmartmeterMQTTKaifaMA309.py
```

Als systemd Service: `smartmeter.service` nach `/etc/systemd/system/` kopieren, Pfade und Benutzer anpassen, die Werte in `/etc/smartmeter.env` eintragen (`chmod 600`) und mit `systemctl enable --now smartmeter` starten.

### Zähler Hersteller
* Kaifa Drehstromzähler MA309


### Unterstützung
Spendenlink: https://www.paypal.me/greenMikeEU

## License

This project is licensed under the GNU General Public License v3.0 License - see the LICENSE.md file for details