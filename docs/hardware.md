# **Hardware Components & Setup Documentation**

---

## Device list

This system consists of the following IoT hardware components:

1. **Raspberry Pi 3 B+** (64 GB microSD storage)  
2. **AudioMoth** (Acoustic logger)  
3. **4G Router** (with SIM card)  
4. **Internet SIM Card**  
5. **Solar Panel** (with battery storage)  

---

## Raspberry Pi setup

### Hardware
- **Model:** Raspberry Pi 3 B+  
- **Storage:** 64 GB microSD card  

---

### OS installation
1. Insert the SD card into your computer.  
2. Use **Balena Etcher** (or similar) to flash the provided `.img` backup OS image.  
3. Insert the flashed SD card into the Raspberry Pi.  
4. Power on the device.  

<!-- | Raspberry Pi | SD Card |
|--------------|---------|
| ![Raspberry Pi](images/pi.png){ style="max-width: 320px;" } | ![SD Card](images/sd-card.png){ style="max-width: 320px;" } | -->

**[Download Raspberry Pi OS Image](https://drive.google.com/drive/folders/19RC69tCjV7lfupJODWT0BL_QIx_DtFqr)**

---

### Login credentials
Default credentials (can be customized):  
- **Username:** `pi`  
- **Password:** `raspberry`  

---

## Example VPN configuration

Each Raspberry Pi has its own OpenVPN account.

### File structure
Navigate to the OpenVPN directory:

```bash
cd /etc/openvpn/
ls
```

Expected files:
```
credentials.txt
openvpn_MONSOON_TEA05.conf
```

- **`openvpn_MONSOON_TEA05.conf`** → Converted `.ovpn` client config file  
- **`credentials.txt`** → VPN username & password (two lines only)

---

### Credentials setup
Check credentials:
```bash
cat /etc/openvpn/credentials.txt
```
Format:
```
vpn_username
vpn_password
```
Secure file permissions:
```bash
sudo chmod 600 /etc/openvpn/credentials.txt
```

---

### Example VPN service setup
Enable & start service:
```bash
sudo systemctl enable openvpn@openvpn_MONSOON_TEA05
sudo systemctl start openvpn@openvpn_MONSOON_TEA05
```

Manual connect:
```bash
sudo openvpn --config openvpn_MONSOON_TEA05.ovpn
```

Verify connection:
```bash
ifconfig
```
VPN tunnel should point to `10.81.234.5`.

---

## Configuration File Explanation (`config.json`)

This file defines the settings for the **bird sound monitoring device**. It covers FTP transfer, sensor recording behavior, system paths, and device identity.  


Example **`config.json`**:
```json
{
    "ftp": {
        "uname": "monsoon",
        "pword": "p8z3%1P#04",
        "host": "192.168.70.5/production-workflow-ec2",
        "use_ftps": 1
    },
    "offline_mode": 0,
    "sensor": {
        "sensor_index": 2,
        "sensor_type": "USBSoundcardMic",
        "record_length": 600,
        "compress_data": false,
        "capture_delay": 0
    },
    "sys": {
        "working_dir": "/home/pi/tmp_dir",
        "upload_dir": "/home/pi/continuous_monitoring_data",
        "reboot_time": "02:00"
    },
    "device_id": "00000000f1c084c2"
}
```



---

### FTP Settings (`ftp`)
Controls how audio files are transferred from the device to the server.

| Key       | Value Example                         | Description                                                                 |
|-----------|---------------------------------------|-----------------------------------------------------------------------------|
| `uname`   | `"monsoon"`                           | FTP username for authentication.                                            |
| `pword`   | `"p8z3%1P#04"`                        | FTP password for authentication.                                            |
| `host`    | `"192.168.70.5/production-workflow-ec2"` | FTP server address and target directory for uploads.                        |
| `use_ftps`| `1`                                   | Enables **FTPS** (1 = secure FTPS, 0 = plain FTP).                          |

---

### Offline Mode (`offline_mode`)
Determines whether the device operates **without uploading data**.

- `0` → Online mode (default) – data is uploaded to server.  
- `1` → Offline mode – data is only stored locally.  

---

### Sensor Settings (`sensor`)
Defines how the sensor captures audio.

| Key             | Value Example   | Description                                                                 |
|-----------------|-----------------|-----------------------------------------------------------------------------|
| `sensor_index`  | `2`             | Index/ID of the sensor used (useful if multiple microphones are connected). |
| `sensor_type`   | `"USBSoundcardMic"` | Type of sensor (here: USB sound card microphone).                           |
| `record_length` | `600`           | Recording length in seconds (600 = 10 minutes).                             |
| `compress_data` | `false`         | Whether to compress recordings before saving/uploading.                     |
| `capture_delay` | `0`             | Delay before starting capture (in seconds).                                 |

---

### System Settings (`sys`)
Defines working directories and reboot schedule for the device.

| Key           | Value Example                         | Description                                                                 |
|---------------|---------------------------------------|-----------------------------------------------------------------------------|
| `working_dir` | `"/home/pi/tmp_dir"`                  | Temporary folder for intermediate files.                                    |
| `upload_dir`  | `"/home/pi/continuous_monitoring_data"` | Directory where recordings are stored before upload.                        |
| `reboot_time` | `"02:00"`                             | Daily scheduled reboot time (HH:MM, 24-hour format).                        |

---

### Device ID (`device_id`)
- Example: `"00000000f1c084c2"`  
- Unique identifier for the Raspberry Pi device.  
- Used to distinguish recordings when multiple devices upload data to the same server.  


---

## Automatic recording service


This shellscript is setup to run the recording script at startup and reboot. So, the device will start recording as soon as it is powered on. And it will also reboot at the scheduled time. The shellscript is located at `/home/pi/custom-pi-setup/recorder_startup_script.sh`.

**systemd service** (`/etc/systemd/system/shellscript.service`):
```ini
[Unit]
Description=My Shell Script

[Service]
ExecStart=/home/pi/custom-pi-setup/recorder_startup_script.sh

[Install]
WantedBy=multi-user.target
```

Check live service logs:
```bash
journalctl -u shellscript.service -f
```

---

## Important commands
| Command | Purpose |
|---------|---------|
| `arecord -l` | List available recording devices |
| `journalctl -u shellscript.service -f` | Live monitoring of recording service |
| `sudo systemctl restart shellscript.service` | Restart recording service |

---

## AudioMoth setup

### Overview
AudioMoth is a low-cost, full-spectrum acoustic logger, based on the Gecko processor range from Silicon Labs.  
It can record **audible and ultrasonic frequencies** at rates from **8,000 to 384,000 samples/sec**.  
It is used in two modes: **mobile** and **station**.

---

### Modes
#### **Mobile Type**
- Portable configuration for temporary deployments  
- Ideal for short-term surveys  
**[Download Mobile AudioMoth Manual (PDF)](files/audiomoth-mobile-recording-manual.pdf)**  

#### **Station Type**
- Fixed position setup for continuous monitoring  
- Powered by solar & external battery  
**[Download IoT Station Setup Manual (PDF)](files/iot-station-setup-devices.pdf)**  

---

## Router setup & troubleshooting

- **Type:** 4G Router with SIM  
- **Purpose:** Internet connection for remote locations  

**Troubleshooting Checklist:**
1. Check LED status indicators  
2. Ensure SIM card is active  
3. Restart router if connection drops  

---

## Solar panel & battery


### Battery
- Stores energy for night/cloudy use  
- **Blink Indicators:**  
  - 1 blink → Low  
  - 2 blinks → Medium  
  - 3 blinks → Full  

---

## System workflow

1. **Power Supply** → Solar Panel → Battery → Raspberry Pi & Router  
2. **Data Capture** → AudioMoth or Raspberry Pi records audio  
3. **Data Transmission** → Router sends via 4G  
4. **Remote Access** → VPN connection for management  
5. **Monitoring** → Logs checked via `journalctl` or SSH  

![System Overview](images/device-overview.png){ style="width: 200px; display: block; margin: 0 auto;" }