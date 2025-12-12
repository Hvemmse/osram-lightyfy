# OSRAM Lightify Gateway – Teknisk Oversigt og Kendte Fakta

Denne dokumentation samler de vigtigste tekniske oplysninger om OSRAM Lightify Gateway (EU-version), herunder hardware-design, firmwarestruktur, debug-adgang og genbrugsmuligheder efter end-of-life.

---

## 1. Hardwareoversigt
Lightify Gateway er en lille embedded Linux-enhed bestående af:

- **CPU / SoC:** Qualcomm/Atheros WiFi SoC med integreret 2.4 GHz WiFi
- **RAM:** 64 MB DDR
- **Flash:** 16–32 MB NAND (kernel + rootfs)
- **Zigbee-radio:** TI CC2530/CC2531 baseret eller OSRAM CC26xx variant
- **WiFi:** 2.4 GHz b/g/n (AP/STA)
- **Strøm:** 5V via intern regulator

Enheden består af **to afskærmede sektioner**:

### Stor metalskærm (venstre)
Indeholder:
- CPU / SoC
- RAM
- NAND flash
- WiFi subsystem
- Bootloader (U-Boot)
- Linux kernel + rootfs
- **UART-pads** (Linux konsol)
- **JTAG pads**

### Lille metalskærm (højre)
Indeholder:
- Zigbee SoC (TI CC2530/CC2531)
- RF-sektion og antennekredsløb
- Zigbee-debug pads (IKKE Linux UART)

---

## 2. Debug og interfaces
Gatewayen har følgende udviklingsinterfaces:

### UART til Linux-konsol
- Findes under den store metalskærm
- Består af 4 pads (GND, TX, RX, 3.3V)
- Baudrate: **115200 8N1**
- Giver adgang til:
  - U-Boot bootloader
  - Kernel bootlog
  - Root shell (afhængig af firmwaretilstand)
  - Mulighed for flash/dump af NAND

### Zigbee-debug interface
- På separat 2×4 pad-gruppe
- Bruges til flashing og debug af Zigbee-radio
- Giver **ikke** adgang til Linux

### 6-pin fabriks-testheader
- Sidder nær strømsektionen
- Ingen Linux uart, primært power/JTAG/boundary scan

---

## 3. Firmware
### OSRAM Lightify firmware består af:
- **U-Boot** bootloader
- Linux kernel (2.x/3.x baseret)
- BusyBox rootfs
- Lightify daemon (lightifyd)
- Zigbee bridge-service
- Cloud agent (nu inaktiv)

### Kendte problemer efter OSRAM Cloud shutdown (2021)
- Cloud-agent hænger under boot
- lightifyd starter ikke → ingen svar på TCP port 4000
- Watchdog genstarter enheden ca. hver 20 sek.
- rootfs-korruption forekommer på NAND

Resultatet er en gateway der:
- Starter WiFi-SSID korrekt
- Accepterer TCP forbindelse på port 4000
- **Sender 0 bytes tilbage** → tjeneste kører ikke
- Rebooter i loop → firmware går aldrig op

---

## 4. Kommunikationen (Lightify LAN-protokol)
- Kører over **TCP port 4000**
- Header: `FE ED`
- Kommandoer for:
  - Version (`0x13`)
  - Device-list (`0x11`)
  - State-change (`0x31`/`0x32`)
- Gateway svarer **kun**, hvis lightifyd og zigbee-services kører.
- Hvis firmware er ødelagt → **ingen svar** (0 bytes)

---

## 5. Kendte firmwaretilstande
- **Normal drift:** port 4000 svarer, zigbee aktiv
- **AP-only / fallback:** kun WiFi-AP, ingen tjenester
- **Bootloop:** watchdog resetter firmware efter ~20 sek
- **Semi-bricked:** kernel booter, men rootfs er korrupt

Din enhed viser symptomer på:
> *Bootloop + service-failure (port 4000 svarer ikke)*

---

## 6. Hvad kan man gøre ved defekt firmware
Gatewayen er fuldt ud **genoplivningsdygtig** via UART:

### Muligheder:
1. **Gendanne OSRAM-firmware** (hvis man har image)  
2. **Installere et selvbygget Linux rootfs**
   - BusyBox + SSH
   - hostapd/dnsmasq (hotspot/driftsmode)
   - MQTT/Zigbee stacks
3. **Brug som Zigbee coordinator**
   - Flash CC2530/31 med deCONZ firmware
4. **OpenWRT-light port** (manuelt build)
5. **NAND reflash + U-Boot recovery**

### Det kræver:
- Aftagning af stor metalskærm
- Lodning eller brug af pogo-pins på UART-pads
- Forbindelse til USB-TTL (3.3V logik)
- Stop af bootloader
- Flash via YMODEM, Kermit eller TFTP

---

## 7. Samlet vurdering
OSRAM Lightify Gateway er en **fuldt hackbar embedded Linux-enhed**, men efter cloud-nedlukningen går mange enheder i en firmwaretilstand hvor de ikke kan initialisere deres tjenester.

Hardware er dog 100% brugbar:
- Linux kan bootes
- Zigbee-radio kan genbruges
- Man kan flashe alternative firmware
- Enheden kan blive hotspot, IoT-node, Zigbee-coordinator m.m.

Med UART-adgang kan alle fejl rettes.

---

## 8. Hvá kan enheden bruges til i dag?
- Zigbee coordinator (deCONZ eller zigpy)
- Lokal Linux hotspot/router
- MQTT-gateway
- Python IoT-controller
- Embedded mini-server

Enheden har gode ressourcer for dens størrelse og kan uden problemer fortsætte livet uden OSRAMs cloud.

---

## Slutbemærkning
Lightify Gateway er et godt eksempel på hardware, som ellers ville blive e-waste, men som stadig har fuld funktionalitet hvis man får adgang via UART og flasher nyt firmware.
Det gør den ideel for entusiaster, udviklere og genbrug af IoT-hardware.

