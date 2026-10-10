# Restoran – Restaurant Ordering & Pager System

An embedded systems project that simulates ordering food in a restaurant and waiting for it to be ready. It uses **two Raspberry Pi Pico W (picoETF) microcontrollers** that talk to each other over Wi-Fi using the **MQTT** protocol.

- **Ordering station (`narucivanje.py`)**: the guest browses the menu on a TFT screen with a rotary encoder, adds items, and confirms the order.
- **Table pager / "zujalica" (`zujalica.py`)**: receives the order, counts down the preparation time on a 7-segment display, then beeps and flashes LEDs until the guest presses a button.

> Project for the course **Ugradbeni sistemi (Embedded Systems)**, Faculty of Electrical Engineering, University of Sarajevo, 2024/25.

---

## Features

- Scrollable menu with prices on a 320×240 ILI9341 TFT display
- Navigation and selection with a KY-040 rotary encoder (rotate to move, click to select)
- YES/NO (DA/NE) confirmation for every item and for the whole order
- Order summary showing item count and total price (in KM)
- Wireless communication between the two boards over MQTT (public broker `broker.emqx.io`)
- Countdown in `MM:SS` format on a multiplexed 4-digit 7-segment display
- Audio and visual alarm (2 kHz buzzer + alternating LEDs) when the order is ready
- Interrupt-driven button to stop the alarm

---

## How it works

1. The ordering station shows a welcome screen. Clicking the encoder opens the menu.
2. The guest rotates the encoder to pick an item and clicks to add it (with a DA/NE confirmation).
3. Choosing **"Zavrsi narudzbu"** (finish order) asks for a final confirmation.
4. On confirmation, the list of ordered items is published over MQTT, e.g. `Pizza, Sok, Kolac`.
5. The pager receives the message and calculates the waiting time: **10 seconds per item**.
6. The time counts down on the 7-segment display.
7. At zero, the buzzer and LEDs turn on and keep going until the guest presses the button.

### Ordering station state machine

The UI is driven by a `screen` variable:

| `screen` | Meaning |
|---|---|
| `0` | Welcome screen |
| `1` | Menu |
| `2` | Confirm adding one item |
| `3` | Final screen (order sent, summary) |
| `4` | Empty-order warning |
| `5` | Confirm the whole order |

---

## Hardware

### Ordering station
| Component | Signal type | Purpose |
|---|---|---|
| Raspberry Pi Pico W (picoETF) | – | Controller |
| ILI9341 TFT-LCD (320×240, SPI) | Digital output | Shows the menu and messages |
| KY-040 rotary encoder | Digital input | Moving through the menu and confirming |

### Pager (zujalica)
| Component | Signal type | Purpose |
|---|---|---|
| Raspberry Pi Pico W (picoETF) | – | Controller |
| 4-digit 7-segment display | Digital output | Shows the remaining time |
| Passive piezo buzzer | Digital output (PWM) | Sound signal when the time runs out |
| LEDs (×8) | Digital output | Visual signal when the time runs out |
| Push button | Digital input | Stops the sound and light signals |

---

## Pinout

### Ordering station (`narucivanje.py`)
| Function | GPIO |
|---|---|
| TFT CLK (SCK) | GP18 |
| TFT MOSI | GP19 |
| TFT MISO | GP16 |
| TFT CS | GP17 |
| TFT RST | GP20 |
| TFT DC | GP15 |
| Encoder CLK | GP0 |
| Encoder DT | GP1 |
| Encoder SW (button) | GP2 |

### Pager (`zujalica.py`)
| Function | GPIO |
|---|---|
| Buzzer (PWM) | GP16 |
| Button (pull-down, rising-edge IRQ) | GP0 |
| 7-segment digits D1–D4 | GP4–GP7 |
| 7-segment segments A–DP | GP8–GP15 |
| LEDs | GP4–GP11 |
| Onboard LED | GP25 |

Full wiring diagrams are in the Fritzing files (`hardware/ekran.fzz`, `hardware/buzzer.fzz`) and their PDF exports.

---

## Repository structure

```
├── narucivanje.py              # Ordering station (TFT + encoder, MQTT publisher)
├── zujalica.py                 # Pager (7-segment, buzzer, LEDs, MQTT subscriber)
├── hardware/
│   ├── ekran.fzz               # Fritzing schematic – ordering station
│   ├── ekran_fritzing.pdf
│   ├── buzzer.fzz              # Fritzing schematic – pager
│   └── buzzer_fritzing.pdf
└── docs/
    ├── dijagram.pdf            # Activity diagram
    ├── Specifikacija_projekta.pdf
    ├── Tehnicka_dokumentacija.pdf
    ├── Korisnicka_uputstva.pdf
    └── Ulazi_i_izlazi_sistema.pdf
```

---

## Getting started

### Requirements
- 2× Raspberry Pi Pico W with **MicroPython** installed
- [Thonny](https://thonny.org/) (or `mpremote`) to upload files
- A 2.4 GHz Wi-Fi network both boards can join

### Libraries
Upload these to the **ordering station** board:
- `ili934xnew.py` – ILI9341 driver
- `tt14.py`, `tt24.py`, `tt32.py` – fonts

Both boards need `umqtt.simple`. If it is missing, install it on the board:
```python
import mip
mip.install("umqtt.simple")
```

### Configuration
Edit the settings at the top of **both** files:
```python
WIFI_SSID     = 'your-network'
WIFI_PASSWORD = 'your-password'
MQTT_BROKER   = 'broker.emqx.io'
MQTT_TOPIC    = b'neda/blue'     # must be the same on both boards
```
The menu can be changed in `narucivanje.py`:
```python
menu_items_base = [
    ("Pizza",   "5"),
    ("Sendvic", "3.5"),
    ("Sok",     "2"),
    ("Kolac",   "2.5"),
]
```

### Running
1. Upload `zujalica.py` to the pager as `main.py` and start it. It subscribes to the topic first.
2. Upload `narucivanje.py` (plus the libraries) to the ordering station as `main.py` and start it.
3. Place an order and watch the countdown start on the pager.

---

## User guide

1. **Click** the encoder on the welcome screen to open the menu.
2. **Rotate** to move up and down the menu.
3. **Click** an item, choose **DA** to add it to the order or **NE** to cancel.
4. Repeat for more items, then select **"Zavrsi narudzbu"** and confirm with **DA**.
5. Go to your table. The pager shows how long you need to wait.
6. When the alarm goes off, your order is ready. **Press the button** on the pager to stop it.
