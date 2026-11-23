# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-script Python 3 tool that reads the customer interface (Kundenschnittstelle) of an EVN / Netz NÖ Kaifa MA309 three-phase smart meter over an RJ12-to-USB serial adapter (pinout in `RJ12-Pinout.PNG`), decrypts the DLMS/COSEM push frames, prints the values and optionally publishes them via MQTT. Code comments, variable names and output are in German.

## Commands

```
pip install -r requirements.txt
set -a; . ./smartmeter.env; set +a
python EvnSmartmeterMQTTKaifaMA309.py
```

There are no tests, linter or build step. Running the script needs the physical meter on the serial port (default `/dev/ttyUSB0`) and the customer key from Netz NÖ.

## Configuration

All configuration comes from environment variables, read at the top of `EvnSmartmeterMQTTKaifaMA309.py`: `EVN_KEY` (required), `MQTT_ENABLED`, `MQTT_BROKER` (port fixed to 1883), `MQTT_USER`, `MQTT_PASSWORD`, `PRINT_VALUES`, `SERIAL_PORT`. The README has the defaults. `smartmeter.env.example` is the template, and the local `smartmeter.env` is gitignored. `smartmeter.service` is an example systemd unit that loads `/etc/smartmeter.env` via `EnvironmentFile=`.

## How it works

1. Reads fixed 282-byte chunks from the serial port at 2400 baud, 8N1. There is no frame synchronisation — it assumes each read lines up with one meter push.
2. `GXDLMSTranslator` (gurux-dlms) with `blockCipherKey` set decrypts the frame and converts it to XML.
3. The XML is parsed with BeautifulSoup and all `uint32` / `uint16` elements are collected.
4. Values are extracted by **slicing the string representation of the BeautifulSoup result lists at fixed character offsets** and parsing the hex. Order of fields: uint32 → energy A+, energy A−, power P+, power P−; uint16 → voltage L1–L3 (/10), current L1–L3 (/100), power factor (/1000).
5. Publishes each value to `Smartmeter/<Name>` topics. The MQTT client connects once at startup (exits with code 1 if the broker is unreachable). After that, paho's `loop_start()` thread keeps the connection alive and reconnects on its own. Readings that arrive while it's disconnected are skipped.

## Gotchas

- The offset-slicing in step 4 depends on the exact string output of BeautifulSoup, including tag-name casing. `html.parser` lowercases tags, hence `find_all('uint32')`; switching parser (e.g. back to `lxml`, which failed on the author's machine) or changing the XML format can silently shift every value.
- All direct requirements are pinned. The MQTT client uses the paho-mqtt 2.x API (`CallbackAPIVersion.VERSION2`); a downgrade to 1.x breaks the constructor.
- Production runs on a Raspberry Pi Zero (armv6l, Python 3.9) with piwheels. Packages with native extensions often have no wheel for it and fail to build there (no C compiler/Rust), so avoid adding such dependencies. gurux-dlms decrypts in pure Python, which is why `cryptography` isn't needed.
