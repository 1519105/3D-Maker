# Kompletní návod: Flash Klipper firmware na Lerdge Z
> STM32F407VET6 | Ověřený funkční postup | v0.13.0

---

## Přehled klíčových parametrů

| Parametr | Hodnota |
|---|---|
| MCU | STM32F407VET6 |
| **Krystal** | **25 MHz** (kritické — ne 8 MHz!) |
| Bootloader offset | 64 KiB (0x08010000) |
| Komunikace | Serial USART1 PA10/PA9 |
| Baud rate | 250 000 |
| RAM | 0x1FFF0 (128KB - 16 bytes) |
| Šifrování SD | Lerdge byte transformace |
| SD složka | `Lerdge_Z_system/Firmware/Lerdge_Z_firmware_force.bin` |

---

## Co potřebuješ

- Raspberry Pi s nainstalovaným Klipperem (Mainsail/Fluidd)
- MicroSD karta ≤ 8 GB
- USB kabel (datový)
- Displej Lerdge nebo originální Lerdge firmware pro obnovu

---

## Krok 1 — Záloha

Před začátkem si ulož originální Lerdge Z firmware (`Lerdge_Z_firmware_force.bin`) na bezpečné místo. Pokud něco selže, vložíš ho na SD kartu do složky `Lerdge_Z_system/Firmware/` a restartneš desku → deska se obnoví.

---

## Krok 2 — Úprava Kconfig

Standardní Klipper má chybnou hodnotu RAM_SIZE pro STM32F4x5. Oprav ji:

```bash
# Záloha původního souboru
cp ~/klipper/src/stm32/Kconfig \
   ~/klipper/src/stm32/Kconfig_default

# Oprava RAM_SIZE: 0x20000 → 0x1FFF0
sed -i 's/default 0x20000 if MACH_STM32F4x5 || MACH_STM32F446/default 0x1FFF0 if MACH_STM32F4x5 || MACH_STM32F446/' \
  ~/klipper/src/stm32/Kconfig

# Ověř obě správné hodnoty
grep "1FFF0\|HAVE_STEPPER_OPTIMIZED_BOTH_EDGE" ~/klipper/src/stm32/Kconfig
```

Očekávaný výstup:
```
    select HAVE_STEPPER_OPTIMIZED_BOTH_EDGE if !MACH_STM32H7
    default 0x1FFF0 if MACH_STM32F4x5 || MACH_STM32F446
```

---

## Krok 3 — Kompilace firmware

```bash
cd ~/klipper
make clean
make menuconfig
```

### Nastavení menuconfig — přesně takto:

```
[*] Enable extra low-level configuration options
    Micro-controller Architecture → STMicroelectronics STM32
    Processor model               → STM32F407
    Bootloader offset             → 64KiB bootloader
    Clock Reference               → 25 MHz crystal    ← KRITICKÉ!
    Communication interface       → Serial (on USART1 PA10/PA9)
(250000) Baud rate for serial port
[*] Optimize stepper code for 'step on both edges'
```

Uložit: `Q` → `Y`

```bash
# Kompilace
make
```

### Ověření správnosti:

```bash
python3 -c "
d = open('out/klipper.bin', 'rb').read()
sp    = int.from_bytes(d[0:4], 'little')
reset = int.from_bytes(d[4:8], 'little') & ~1
print(f'SP:     {hex(sp)}')
print(f'Reset:  {hex(reset)}')
print(f'Size:   {len(d)} bytes')
if sp == 0x2001fff0 and 0x08010000 <= reset < 0x08020000:
    print('✓ OK - firmware je správný!')
else:
    print('✗ CHYBA - zkontroluj nastavení menuconfig')
"
```

Očekávaný výstup:
```
SP:     0x2001fff0
Reset:  0x8010xxx
Size:   ~24000 bytes
✓ OK - firmware je správný!
```

---

## Krok 4 — Šifrovací skript

Lerdge bootloader přijímá pouze šifrované soubory. Vytvoř skript jednou, používej opakovaně:

```bash
cat > ~/lerdge_encrypt.py << 'SCRIPT'
#!/usr/bin/env python3
import sys

def encrypt(data):
    result = bytearray(data)
    for i in range(len(result)):
        b = result[i]
        b = (~b) & 0xFF
        b = ((b >> 5) | (b << 3)) & 0xFF
        b = (b + 85) & 0xFF
        b = (~b) & 0xFF
        b = ((b << 3) | (b >> 5)) & 0xFF
        result[i] = b
    return bytes(result)

with open(sys.argv[1], 'rb') as f:
    data = f.read()
encrypted = encrypt(data)
with open(sys.argv[2], 'wb') as f:
    f.write(encrypted)
print(f"Hotovo: {len(data)} bytes -> {sys.argv[2]}")
SCRIPT

chmod +x ~/lerdge_encrypt.py
```

### Zašifrování firmware:

```bash
python3 ~/lerdge_encrypt.py \
  ~/klipper/out/klipper.bin \
  ~/klipper_for_lerdge.bin

echo "Velikost: $(wc -c < ~/klipper_for_lerdge.bin) bytes"
```

---

## Krok 5 — Příprava SD karty

1. Formátuj SD kartu: **FAT32, alokační jednotka 4096 bytes**, velikost ≤ 8 GB
2. Vytvoř přesnou strukturu složek:

```
SD:\
  └── Lerdge_Z_system\
        └── Firmware\
              └── Lerdge_Z_firmware_force.bin
```

3. Stáhni soubor z RPi na PC:

```bash
# Linux/Mac:
scp m@<IP_ADRESA>:/home/m/klipper_for_lerdge.bin ./Lerdge_Z_firmware_force.bin

# Windows: použij WinSCP
# Cesta na RPi: /home/m/klipper_for_lerdge.bin
```

4. Zkopíruj soubor na SD kartu — přesný název: `Lerdge_Z_firmware_force.bin`

---

## Krok 6 — Flash firmware

### Varianta A: Přes Lerdge displej UI (pokud displej svítí)

1. Vlož SD kartu do tiskárny
2. Na displeji: **Ozubené kolo → ikona aktualizace → vyber soubor → potvrď**
3. Po dokončení displej zhasne — Klipper běží ✓

### Varianta B: Force update — automatický (pokud displej je tmavý)

1. SD karta s přesnou strukturou z Kroku 5
2. Vlož SD do tiskárny
3. Vypni a zapni tiskárnu — bootloader automaticky detekuje soubor a flashuje
4. **Po úspěchu smaž složku `Lerdge_Z_system` z SD karty** — jinak se bude přepisovat při každém startu

---

## Krok 7 — Oprava printer.cfg

Po flashi nastav správný restart_method v `~/printer_data/config/printer.cfg`:

```bash
# Zkontroluj sekci [mcu]
grep -n -A5 "^\[mcu\]" ~/printer_data/config/printer.cfg
```

Sekce musí vypadat takto:
```ini
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
restart_method: command
```

Pokud `restart_method` chybí, přidej ho:
```bash
sed -i '/^serial:\/dev\/serial\/by-id\/usb-1a86_USB_Serial-if00-port0/a restart_method: command' \
  ~/printer_data/config/printer.cfg
```

Pak restartuj Klipper:
```bash
sudo systemctl restart klipper
```

---

## Krok 8 — Ověření

```bash
# Zkontroluj verzi MCU
journalctl -u klipper -n 20 | grep -i "version\|mcu\|connect"
```

Nebo v Mainsail/Fluidd → Machine → zobrazí se verze MCU.

Správný výsledek:
```
MCU version: v0.13.0-xxx (bez deprecated hlášky)
Freq: 168 MHz
```

---

## Aktualizace firmware v budoucnu

Při každé aktualizaci Klipperu stačí opakovat kroky 3–6:

```bash
cd ~/klipper
git pull
make clean && make
python3 ~/lerdge_encrypt.py ~/klipper/out/klipper.bin ~/klipper_for_lerdge.bin
# → stáhnout a nahrát přes displej nebo Force update
```

Kconfig úprava z Kroku 2 přetrvá — není potřeba opakovat.  
Pokud by `git pull` přepsal `Kconfig`, znovu spusť příkaz ze Kroku 2.

---

## Obnova Lerdge firmware (záchrana)

Pokud deska přestane reagovat:

1. Na SD kartě: `Lerdge_Z_system/Firmware/Lerdge_Z_firmware_force.bin` — originální Lerdge soubor
2. Vlož SD do desky, restartuj
3. Displej se rozsvítí → deska obnovena
4. Zopakuj postup od Kroku 6

---

## Řešení problémů

| Problém | Příčina | Řešení |
|---|---|---|
| `Serial connection closed` | Špatný krystal (8 vs 25 MHz) | Zkompiluj s 25 MHz |
| `Failed automated reset` | Chybí `restart_method` | Přidej do printer.cfg |
| SD soubor zůstal nenulován | Špatná cesta/název souboru | Zkontroluj přesnou strukturu |
| SP = 0x20020000 (ne 0x2001fff0) | Chybí úprava RAM_SIZE | Zopakuj Krok 2 |
| `make: error: CONFIG_STM32_DFU_ROM_ADDRESS` | Lerdge Kconfig je nekompatibilní | Použij výchozí Kconfig + úprava z Kroku 2 |
| `-dirty` ve verzi | Upraven Kconfig soubor | Normální, nevadí |

---

*Odladěno na: Lerdge Z V1.0.0, STM32F407VET6, Klipper v0.13.0, RPi 3B, Debian Bullseye*
