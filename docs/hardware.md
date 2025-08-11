# Hardware Components Documentation

## 1. IoT Device List

The following devices are used in the system:

1. **Raspberry Pi**  
2. **AudioMoth**  
3. **4G Router**  
4. **Internet SIM Card**  
5. **Solar Panel**

<!-- 
![Raspberry Pi](pi.jpg)  
![AudioMoth](audiomoth.jpg)  
![Internet SIM Card](sim-card.jpg)  
![Solar Panel](solar-panel.jpg) -->

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

<!-- ![Raspberry Pi](images/pi.png)  
![SD Card](images/sd-card.png)   -->

<p align="center">
  <img src="images/pi.png" alt="Raspberry Pi" width="45%"/>
  <img src="images/sd-card.png" alt="SD Card" width="45%"/>
</p>





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

AudioMoth is a low-cost, full-spectrum acoustic logger, based on the Gecko processor range from Silicon Labs. Just like its
namesake the moth, AudioMoth can listen at audible frequencies, well into ultrasonic frequencies. It is capable of recording
uncompressed audio to microSD card at rates from 8,000 to 384,000 samples per second. The AudioMoth is used for audio data collection in **two modes**:


### 3.1. Mobile Type Usage
- Portable configuration for temporary deployments.  
- Ideal for short-term surveys.  
- See attached manual for setting up the mobile type Audiomoth.

[Download Audiomoth setup manual PDF](files/Audiomoth mobile type recording Manual.pdf)

### 3.2. Station Type Usage
- Fixed position setup for continuous monitoring.  
- Powered by solar and external battery.  
- See attached manual for setting up the IOT station type.

[Download Whole IOT device setup PDF version](files/Setting Up IoT Station Devices.pdf)

---

## 4. Router Status

- **Type:** 4G Router  
- **Function:** Provides internet connectivity via SIM card.  

**Troubleshooting:**
- Check LED status indicators.  
- Ensure SIM card is active.  
- Restart router if connection drops.

![Router](images/img.jpg)  

---

## 5. Solar Panel Status

- **Purpose:** Supplies power to IoT devices in remote areas.  

**Indicators:**
- Green light: Charging  
- Red light: Low battery  
- Off: No power supply


---

## 6. Battery Status

- **Purpose:** Stores energy for nighttime or cloudy-day operation.  

**LED/Blink Indicators:**
- 1 blink: Low power  
- 2 blinks: Medium  
- 3 blinks: Fully charged  


---

## 7. Overall Working Flow

1. **Power Supply:** Solar panel → Battery → Raspberry Pi + Router.  
2. **Data Capture:** AudioMoth / Pi records data.  
3. **Data Transmission:** Router sends data over 4G network.  
4. **Remote Access:** OpenVPN connection to retrieve/manage data.  
5. **Monitoring:** Logs checked via `journalctl` or SSH commands.

![System Overview](images/device-overview.png)  
