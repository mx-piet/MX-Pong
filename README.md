# MX-Pong
1D LED Pong Game (WS2812)
# MX-Ball – 1D LED Pong für ESP32

Ein Pong-Spiel für zwei Spieler auf einem einzigen WS2812B-LED-Streifen.
Der Ball läuft als Lichtpunkt über den Streifen, jeder Spieler hat einen
Taster und muss den Ball in seiner Rückspielzone zurückschlagen.
Je später man drückt (je näher am Rand), desto schneller fliegt der Ball zurück.

> Diese Ausgabe (**V2.5.2-NS**) ist die Version **ohne Sound**.
> Für den Aufbau braucht man nur ESP32, LED-Streifen und zwei Taster.

## Features

- **1D-Pong** auf einem adressierbaren LED-Streifen (Standard: 100 LEDs)
- **Geschwindigkeit nach Timing:** 10 Speed-Segmente, Ballfarbe zeigt das Tempo
  (Weiß = langsam, Gelb = mittel, Magenta = schnell)
- **Zone-Shrink:** Nach einer einstellbaren Zahl an Ballwechseln rücken die
  Rückspielzonen zur Mitte – das Spielfeld wird kürzer, das Spiel schneller
- **Multi-Ball:** Bei langen, langsamen Ballwechseln kommt ein 2. und
  später ein 3. Ball ins Spiel
- **Match-Modus** mit Punktezählung (Standard: 10 Punkte), Punkteanzeige auf
  dem Streifen, Sieger-Sweep und Lauflicht am Match-Ende
- **Start-Countdown:** schrumpfender Balken Rot → Gelb → Grün-Flash
- **Demo-Modus** mit Regenbogen-Lauflicht; Spielstart, wenn beide Taster
  gedrückt gehalten werden
- **Taster-LEDs** blinken im Demo-Modus und leuchten beim Zurückschlagen kurz auf
- **WLAN-Access-Point mit zwei Weboberflächen:**
  - Hauptseite (Port 80): Live-Spielstand, Spielernamen, Top-100-Bestenliste
  - Konfiguration (Port 7070): alle Spielparameter live einstellbar
- **FreeRTOS Dual-Core:** LED-Ausgabe und Webserver laufen auf Core 0,
  die Spiellogik auf Core 1 – kein Ruckeln durch WLAN

## Hardware

<p align="center">
  <img src="media/mx-ball-hardware.jpg" alt="MX-Ball Aufbau: LED-Streifen im Alu-Profil und zwei Handtaster mit LED" width="300">
  <br><em>LED-Streifen im Alu-Profil und zwei Handtaster mit Arcade-Knopf (rot/blau beleuchtet)</em>
</p>

| Bauteil | Hinweis |
|---|---|
| ESP32 WROOM-32D (DevKit) | andere ESP32-Boards gehen meist auch |
| WS2812B-LED-Streifen | Anzahl über `MX_LED_COUNT` einstellbar |
| 2 × Taster | idealerweise mit eingebauter LED (Arcade-Taster) |
| 5-V-Netzteil | passend zur LED-Anzahl (bis 60 mA pro LED bei Vollweiß) |
| Widerstand 100–470 Ω | in die Datenleitung zum LED-Streifen |
| optional: Kondensator 1000 µF | über 5 V/GND am Streifenanfang |

### Pinbelegung

| GPIO | Funktion | Anschluss |
|---|---|---|
| 5 | LED-Daten (WS2812B) | über Widerstand an DIN des Streifens |
| 18 | Taster links | Taster gegen GND (interner Pull-up) |
| 19 | Taster rechts | Taster gegen GND (interner Pull-up) |
| 26 | Taster-LED links | Active-High |
| 27 | Taster-LED rechts | Active-High |

> ⚠️ Den LED-Streifen **nicht** über den ESP32 versorgen, sondern direkt
> vom 5-V-Netzteil. GND von Netzteil, Streifen und ESP32 verbinden.
> Je nach Taster-LED ggf. einen Vorwiderstand einplanen
> (ESP32-GPIO max. ca. 20 mA).

## Software / Installation

Benötigt:

- [Arduino IDE](https://www.arduino.cc/en/software) 2.x
- **ESP32 Board-Paket von Espressif, Version ≥ 3.0.0**
  (Boardverwalter-URL:
  `https://espressif.github.io/arduino-esp32/package_esp32_index.json`)
- Bibliothek **FastLED** (über den Bibliotheksverwalter)
- `WiFi` und `WebServer` sind im ESP32-Paket enthalten

Schritte:

1. Repository herunterladen (grüner Button **Code → Download ZIP**) oder klonen
2. `MX-Pong/MX-Pong.ino` in der Arduino IDE öffnen
3. Board **ESP32 Dev Module** wählen, COM-Port wählen
4. Bei Bedarf `MX_LED_COUNT` an die Länge deines Streifens anpassen
5. Hochladen

## Bedienung

1. Nach dem Einschalten läuft der **Demo-Modus** (Regenbogen-Lauflicht).
2. **Beide Taster gedrückt halten** → Countdown startet → Spiel beginnt.
3. Drücke deinen Taster, wenn der Ball in **deiner Zone** ist
   (links blau, rechts rot). Je später, desto schneller der Rückschlag.
4. Verpasst ein Spieler den Ball, bekommt der Gegner einen Punkt.
5. Wer zuerst die eingestellten Match-Punkte erreicht, gewinnt.

### Weboberfläche

Mit dem WLAN des Spiels verbinden:

| | |
|---|---|
| SSID | `MX-Ball` |
| Passwort | wie in `MX_WIFI_PASSWORD` eingestellt |
| Hauptseite | <http://192.168.70.1> |
| Konfiguration | <http://192.168.70.1:7070> |

> ⚠️ Vor dem ersten Hochladen in `MX-Pong.ino` das WLAN-Passwort
> `MX_WIFI_PASSWORD` setzen (Platzhalter: `DEIN_PASSWORT`, mindestens 8 Zeichen).

Parameter und Spielernamen lassen sich nur im **Demo-Modus** ändern
(während eines Spiels ist die Seite gesperrt). Einstellungen gelten
bis zum nächsten Neustart.

## Einstellbare Parameter (Auszug)

| Parameter | Standard | Bedeutung |
|---|---|---|
| LED-Helligkeit | 30 % | Gesamthelligkeit |
| Basisgeschwindigkeit | 50 LEDs/s | Grundtempo des Balls |
| Zone links / rechts | 20 LEDs | Größe der Rückspielzonen |
| Vordrück-Fenster | 300 ms | so lange gilt ein Tastendruck vor Ankunft des Balls |
| Ballwechsel bis Shrink | 10 | nach so vielen Wechseln schrumpfen die Zonen |
| Shrink pro Schritt | 10 LEDs | Verschiebung der Zonen pro Shrink |
| 2./3. Ball | an | Multi-Ball nach 6 langsamen Ballwechseln |
| Match-Punkte | 10 | Punkte bis zum Sieg |
| Countdown | 3000 ms | Dauer des Start-Countdowns |
| Speed-Tabelle | 1.0 … 6.0 | Tempo-Multiplikator je Treffer-Segment |

## Lizenz

© 2026 Peter Müller ([@mx-bikepiet](https://github.com/mx-bikepiet)) – veröffentlicht unter der [GNU GPL v3.0](LICENSE).

Nachbauen, Nutzen und Verändern ist ausdrücklich erwünscht. Wer eine
veränderte Version weitergibt, muss sie ebenfalls unter der GPL v3.0
inklusive Quellcode veröffentlichen.
