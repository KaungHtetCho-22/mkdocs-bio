# Hardware Components Documentation

## 1. IoT Device List

The following devices are used in the system:

1. **Raspberry Pi**  
2. **AudioMoth**  
3. **4G Router**  
4. **Internet SIM Card**  
5. **Solar Panel**

![System Overview](overview-img.jpg)  
![Raspberry Pi](pi.jpg)  
![AudioMoth](audiomoth.jpg)  
![Internet SIM Card](sim-card.jpg)  
![Solar Panel](solar-panel.jpg)

---

## 2. Raspberry Pi — OS & Installation

- **Model:** Raspberry Pi 3 B+  
- **Storage:** 64 GB microSD card  
- **Setup:** Flash the backup OS image to the new SD card.  

**Installation Process:**
1. Insert the SD card into your computer.  
2. Use an imaging tool (e.g., Balena Etcher) to flash the backup `.img` file.  
3. Insert the flashed SD card into the Raspberry Pi.  
4. Power on the device.

![Raspberry Pi](pi.jpg)  
![SD Card](sd-card.jpg)  

[![Download pi-image here](https://img.shields.io/badge/Google%20Drive-Open%20Folder-blue?logo=google-drive)](https://drive.google.com/drive/folders/19RC69tCjV7lfupJODWT0BL_QIx_DtFqr)

---

### 2.1. Important Scripts & Files

The Raspberry Pi OS contains key scripts and configurations:

| File / Service            | Description |
|---------------------------|-------------|
| `recorder-script.sh`      | Main script to handle audio recording. |
| `config.json`             | Configuration file for device parameters. |
| OpenVPN installation (UI) | Provides secure remote access. |
| `journalctl -u shellscript.service -f` | Command to check live service logs. |
| `arecord -l`               | Command to list available recording devices. |

---

## 3. AudioMoth Device

The AudioMoth is used for audio data collection in **two modes**:

- [Download Audiomoth setup manual PDF](files/Audiomoth mobile type recording Manual.pdf)

- [Download Whole IOT device setup PDF version](files/Setting Up IoT Station Devices.pdf)


### 3.1. Mobile Type Usage
- Portable configuration for temporary deployments.  
- Ideal for short-term surveys.  

*(Insert images here)*

### 3.2. Station Type Usage
- Fixed position setup for continuous monitoring.  
- Powered by solar or external battery.  

*(Insert images here)*

---

## 4. Router Status

- **Type:** 4G Router  
- **Function:** Provides internet connectivity via SIM card.  

**Troubleshooting:**
- Check LED status indicators.  
- Ensure SIM card is active.  
- Restart router if connection drops.

![Router](img.jpg)  

---

## 5. Solar Panel Status

- **Purpose:** Supplies power to IoT devices in remote areas.  

**Indicators:**
- Green light: Charging  
- Red light: Low battery  
- Off: No power supply

*(Insert detailed specifications here)*

---

## 6. Battery Status

- **Purpose:** Stores energy for nighttime or cloudy-day operation.  

**LED/Blink Indicators:**
- 1 blink: Low power  
- 2 blinks: Medium  
- 3 blinks: Fully charged  

*(Insert battery photos here)*

---

## 7. Overall Working Flow

1. **Power Supply:** Solar panel → Battery → Raspberry Pi + Router.  
2. **Data Capture:** AudioMoth / Pi records data.  
3. **Data Transmission:** Router sends data over 4G network.  
4. **Remote Access:** OpenVPN connection to retrieve/manage data.  
5. **Monitoring:** Logs checked via `journalctl` or SSH commands.

*(Insert workflow diagram here)*