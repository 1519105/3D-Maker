# 3D Factories Profi3DMaker — Klipper Conversion

A full conversion of the **3D Factories Profi3DMaker** — a closed-source commercial printer with no autoleveling, no web interface, and limited usability — into a modern, fully-featured Klipper-based machine.

![Printer overview](Images/IMG_9366.jpg)

---

## Why This Conversion?

The original Profi3DMaker had significant limitations out of the box:

- No automatic bed leveling
- No web-based control interface
- No input shaper / resonance compensation
- No exclude objects support
- No modern slicer profile support
- Proprietary, closed firmware with no community support

This project replaces the original control system entirely while keeping the robust aluminum extrusion frame and large build volume.

---

## Hardware Overview

| Component | Original | After Conversion |
|---|---|---|
| Control board | Proprietary | **Lerdge Z (STM32F407)** |
| Firmware | Closed proprietary | **Klipper** |
| Web interface | None | **Mainsail (Raspberry Pi 4 2GB)** |
| Extruder | Stock direct | **HGX direct drive extruder** |
| Bed leveling | Manual only | **BLTouch v3.4** |
| Probe (alt.) | None | **Klicky Probe v2** |
| Belts | Stock | **Stock** |
| Enclosure | Partial | **Full plexiglass panels** |
| Lighting | None | **LED strip** |
| Cable management | Hard cables | **Flexible silicone cables** |

---

## Build Volume

| Axis | Size |
|---|---|
| X | 400 mm |
| Y | 260 mm |
| Z | 190 mm |

---

## Electronics Bay

The left section of the frame houses all electronics: PSU, Lerdge Z board, and Raspberry Pi 4 — all separated from main print chamber, with separated ventilation.

![Electronics bay overview](Images/IMG_9367.jpg)
![Electronics detail](Images/IMG_9368.jpg)

### Fusion 360 — Print Head Assembly

STL models includes electronics covers and holder:

![Electronic bay cover stl set](STLs/MB_cover_set.stl)

---

## Print Head

The original print head was completely replaced with a custom-designed assembly modeled in **Autodesk Fusion 360**. It integrates:

- **HGX direct drive extruder** (Ideaformer BDE v1)
- **BLTouch v3.4** for automatic mesh bed leveling
- **Klicky Probe v2** as an alternative probe option
- Dual part cooling fans with custom duct
- Hotend fan
- Cable chain attachment

![Print head close-up](Images/IMG_9369.jpg)

### Fusion 360 — Print Head Assembly

STL models includes the carriage, extruder mount, fan ducts, probe holders, and belt holder:

![Printer head stl set](STLs/PrinterHead_set.stl)

---

## Repository Contents

```
3D-Maker/
├── config/
│   ├── printer.cfg          — Main Klipper configuration
│   ├── macros.cfg           — Custom macros (PRINT_START, PRINT_END, etc.)
│   └── moonraker.conf       — Moonraker / Mainsail config
├── STLs/
│   └── bed_plate.stl        — Custom printed bed plate
│   └── PrinterHead_set.stl  — Custom printer head
├── Orca/
│   └── MP_3DMaker_0_4_nozzle.orca_printer  — Full OrcaSlicer printer bundle
├── Images/
│   ├── IMG_9366.jpg         — Printer overview
│   ├── IMG_9367.jpg         — Electronics bay
│   ├── IMG_9368.jpg         — Electronics detail
│   └── IMG_9369.jpg         — Print head close-up
│   ├── Snímek_obrazovky_2026-04-09_150134.png   — OrcaSlicer profile
│   ├── Snímek_obrazovky_2026-04-09_151301.png   — Enclosure CAD front
│   ├── Snímek_obrazovky_2026-04-09_151306.png   — Enclosure CAD rear
│   ├── Snímek_obrazovky_2026-04-09_151631.png   — Print head CAD front
│   ├── Snímek_obrazovky_2026-04-09_151637.png   — Print head CAD rear
│   ├── Snímek_obrazovky_2026-04-09_151726.png   — Print head CAD bottom
│   └── Snímek_obrazovky_2026-04-09_151839.png   — Print head CAD exploded
└── README.md
```

---

## OrcaSlicer Profile

A complete **OrcaSlicer printer bundle** (`.orca_printer`) is included with print macros.

![OrcaSlicer profile](Images/Snímek_obrazovky_2026-04-09_150134.png)

**Printer:** MP 3DMaker — 0.4 mm nozzle  
**Build area:** 400 × 260 mm  
**Max height:** 190 mm

### Included Filament Profiles

| Filament | Notes |
|---|---|
| Generic PET | Template profile |
| Kingroon PLA | Standard PLA |
| PLA BODY SUNLU | Body/structural parts |
| PLA SATEN WHITE HS | High-speed satin PLA |
| PLA SATEN BLACK HS | High-speed satin PLA |
| PLA SILK GOLD / GREEN / RED | Silk PLA variants |
| PLABasic Alzament | Budget PLA |
| PETG Nebula | PETG for vases / fast prints |
| RecPET Magnesia Blue | Recycled PET |
| Prusament PCCF | Carbon-fibre reinforced PC |
| TPU 96A | Flexible filament |

### Included Process Profiles

| Profile | Layer height |
|---|---|
| 0.2 standard @for all | 0.2 mm — general purpose |
| 0.1 vase | 0.1 mm — vase / detail mode |
| 0.2mm PCCF Strength | 0.2 mm — tuned for PC-CF |

### How to Import

1. Open **OrcaSlicer**
2. Go to **File → Import → Import Configs**
3. Select `MP_3DMaker_0_4_nozzle.orca_printer`
4. All printer, filament, and process profiles will be imported at once

---

## Klipper Features Enabled

- ✅ Automatic mesh bed leveling (BLTouch)
- ✅ Web interface via Mainsail
- ✅ Web camera
- ✅ Input shaper / resonance compensation
- ✅ Exclude objects mid-print
- ✅ Pressure advance tuning
- ✅ Custom `PRINT_START` / `PRINT_END` macros
- ✅ Filament runout sensor support
- ✅ Real-time config editing without reflashing
- ✅ Z-tilt screw function
- ✅ Emergency PSU shutdown

---

## Control Board — Lerdge Z

The Lerdge Z board uses an **STM32F407** MCU. Klipper firmware must be compiled with the following settings and then "crypted" with provided Lerdge FW tool or python script:

- Micro-controller: **STM32F407**
- Bootloader offset: **32KiB**
- Clock reference: **25 MHz crystal**
- Communication: **UART on PA9/PA10**

Flash via original firmware way with attached display.

---

## KlipperScreen on Android tablet and xserver

### Instalation of launch script a setting up system service

```bash
set -e

# 1) Závislosti
sudo apt update
sudo apt install -y adb x11-xserver-utils

# 2) Přepiš launch skript vlastní verzí (ADB forward + XSDL detekce portu)
cat > "$HOME/KlipperScreen/launch_klipperscreen.sh" << 'LAUNCH_EOF'
#!/bin/bash
echo "Čekám na ADB zařízení..."
timeout=0
while ! adb devices | grep -q "device$" && [ $timeout -lt 30 ]; do
    sleep 1
    timeout=$((timeout+1))
done
if ! adb devices | grep -q "device$"; then
    echo "ADB nenalezeno, konec."
    exit 1
fi
adb shell am start -n x.org.server/.MainActivity
sleep 3
XSDL_PORT=""
timeout=0
echo "Hledám XSDL port..."
while [ -z "$XSDL_PORT" ] && [ $timeout -lt 20 ]; do
    for port in 6000 6001 6002 6003 6004 6005 6006 6007 6008 6009 6010; do
        echo "  Zkouším port $port..."
        adb forward tcp:6100 tcp:$port 2>/dev/null
        if timeout 2 bash -c "DISPLAY=:100 xset -q > /dev/null 2>&1"; then
            XSDL_PORT=$port
            echo "Nalezen port: $XSDL_PORT"
            break
        fi
    done
    timeout=$((timeout+1))
    sleep 1
done
if [ -z "$XSDL_PORT" ]; then
    echo "Port nenalezen, konec."
    exit 1
fi
echo "Spouštím KlipperScreen na portu $XSDL_PORT"
DISPLAY=:100 "$HOME/.KlipperScreen-env/bin/python" \
    "$HOME/KlipperScreen/screen.py"
LAUNCH_EOF
chmod +x "$HOME/KlipperScreen/launch_klipperscreen.sh"

# 3) Přepiš systemd unit (User a cesty se vyplní z aktuálního uživatele)
sudo tee /etc/systemd/system/KlipperScreen.service > /dev/null << SERVICE_EOF
[Unit]
Description=KlipperScreen
StartLimitIntervalSec=0
After=moonraker.service
After=dbus.socket systemd-logind.service

[Service]
Type=simple
Restart=always
RestartSec=5
User=$USER
WorkingDirectory=$HOME/KlipperScreen
ExecStart=$HOME/KlipperScreen/launch_klipperscreen.sh

[Install]
WantedBy=multi-user.target
SERVICE_EOF

# 4) Aktivuj
sudo systemctl daemon-reload
sudo systemctl enable KlipperScreen.service
sudo systemctl restart KlipperScreen.service
sudo systemctl status KlipperScreen.service --no-pager
```
### Authorization of Android device

adb devices vrací buď prázdno nebo unauthorized — proto skript pořád čeká. Pojďme to diagnostikovat:

```bash
# 1) zastav službu aby ti nepřekážela při testování
sudo systemctl stop KlipperScreen.service

# 2) ručně otestuj ADB
adb devices
```

Co očekávat a jak reagovat:
a) Prázdný výpis (List of devices attached a nic dál):

Tablet vůbec není vidět přes USB. Zkontroluj kabel (musí být datový, ne jen napájecí), zkus jiný USB port na RPi, případně restartuj ADB:

```bash
adb kill-server
  adb start-server
  adb devices
  lsusb   # vidíš tablet v seznamu USB zařízení?
```

b) unauthorized vedle ID zařízení:

Na tabletu se má objevit dialog "Allow USB debugging from this computer?" — potvrď ho a zaškrtni "Always allow". Pokud se dialog neobjeví, zkus na tabletu v Developer Options:

vypni a zapni USB debugging
klikni na "Revoke USB debugging authorizations" (smaže staré klíče z původního RPi)
znovu připoj kabel → dialog by se měl objevit



c) device se objeví ✓:

Pak je ADB OK a problém byl jen v autorizaci. Spusť ručně skript a uvidíš celý průběh včetně XSDL detekce:

```bash
/home/m/KlipperScreen/launch_klipperscreen.sh
```

Důležité: RPi a tablet mají teď nový pár ADB klíčů (původní byly na starém RPi v ~/.android/). Tablet tě musí znovu autorizovat, jinak jedeš pořád dokola na čekání.

```bash
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached
0123456789ABCDEF        no permissions (missing udev rules? user is in the plugdev group); see [http://developer.android.com/tools/device.html]
```

Klasika — chybí udev pravidla pro USB tablet. Tohle se stalo protože nový systém nemá nastaveny ty samé permissions jako starý. Oprava:

```bash
# 1) Doinstaluj balík s udev pravidly pro Android zařízení
sudo apt install -y android-sdk-platform-tools-common

# 2) Ujisti se že jsi v plugdev (asi už jsi, ale pro jistotu)
sudo usermod -aG plugdev $USER

# 3) Reload udev pravidel a restart ADB
sudo udevadm control --reload-rules
sudo udevadm trigger
adb kill-server
adb start-server

# 4) Otestuj
adb devices
```

Pokud i po tomhle zůstane no permissions, znamená to že vendor ID tvého tabletu není v default pravidlech (/lib/udev/rules.d/51-android.rules). Zjisti vendor ID:

```bash
lsusb
```

Najdi v seznamu tablet (něco jako Bus 001 Device 005: ID 18d1:4ee7 Google Inc.) a pošli sem ten řádek. Pak ti vyrobím vlastní udev pravidlo.
Pozn.: Po usermod -aG plugdev se členství projeví až po novém přihlášení — ale na samotnou službu to nemá vliv (ta běží jako User=m a systemd respektuje supplementary groups z /etc/group při startu). Takže až udev pravidla povolí přístup, můžeš službu rovnou nahodit.

```bash
lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 003: ID 0bb4:0c03 HTC (High Tech Computer Corp.) Android Phone [Fairphone First Edition (FP1)]
Bus 001 Device 004: ID 1a86:7523 QinHeng Electronics CH340 serial converter
Bus 001 Device 005: ID 1e45:8022 Suyin HD Camera
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```

Tablet vidím — je to ten HTC 0bb4:0c03 (vendor ID 0bb4 patří HTC). Vytvoříme pro něj vlastní udev pravidlo:

```bash
# Vytvoř udev pravidlo pro HTC vendor ID 0bb4
sudo tee /etc/udev/rules.d/51-android-htc.rules > /dev/null << 'EOF'
SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", MODE="0666", GROUP="plugdev"
EOF

# Reload pravidel a aplikace na již připojené zařízení
sudo udevadm control --reload-rules
sudo udevadm trigger

# Restart ADB
adb kill-server
adb start-server

# Test
adb devices
```

Měl bys vidět 0123456789ABCDEF device (bez no permissions).
Pak by tablet měl ukázat dialog na autorizaci — potvrď, dej "Always allow", a:

```bash
adb devices       # mělo by být 'device', ne 'unauthorized'
sudo systemctl start KlipperScreen.service
sudo systemctl status KlipperScreen.service --no-pager
```

Pokud bude pořád no permissions, je možné že je třeba fyzicky odpojit a znovu připojit USB kabel aby se nová pravidla aplikovala (udev pravidla se na MODE/GROUP aplikují při enumeraci zařízení, ne retroaktivně, i přes udevadm trigger).

## Notes

- The original printer frame, rods, and leadscrews were reused
- The Raspberry Pi 4 runs Klipper host + Moonraker + Mainsail
- All custom structural parts were designed in **Autodesk Fusion 360**
- STL files are print-ready for 0.2mm layer height, 3 perimeters, 40% infill, I choose PCCF
