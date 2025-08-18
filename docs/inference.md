# **Bird Sound Monitoring & Scoring Pipeline**


## Overview

This system automates the end-to-end process of **monitoring bird sounds** using IoT devices, classifying them with a soundscape model, predicting biodiversity scores, and delivering the results as JSON payloads to an API.

Repository: **[inference-workflow‑iNet](https://github.com/KaungHtetCho-22/inference-workflow-iNet)**

![System Overview Diagram](images/overview-diagram.png){ style="width: 400px; display: block; margin: 0 auto;" }

<p align="center"><em>Figure 1: Bird Sound Monitoring &amp; Scoring Pipeline Overview</em></p>

---

### Data source – Raspberry Pi
- **Device:** Raspberry Pi with Audiomoth sensors for continuous field audio collection.  
- **Protocol:** FTPS (Secure FTP) for encrypted data transfer.  

- **iNet Server 4 Machine Information:**
  
  | Parameter  | Value             |
  |------------|-------------------|
  | Username   | monsoon           |
  | IP Address | 192.168.70.5      |
  | Password   | p8z3%1P#04        |

- **Destination:** Audio files are securely uploaded to the **iNet private cloud**.

---

### Audio collection
- Captures **10-minute audio clips** in `.WAV` format.
- Sample audio files can be downloaded from **[this Google Drive link](https://drive.google.com/drive/folders/1y59QnqjmbVWW-pZhIONdZJuEHzz-iiYz?usp=sharing)**
- See attached how to fetch audio from filezilla setup manual **[Filezilla setup manual](files/iNET_audio_files_feteching_manual.pdf)**

---

### Bird classification model
- Processes audio clips using a **deep learning soundscape model**.  
- Identifies bird species with **confidence scores**.  
- Stores classification results in a **SQLite database** for further analysis.

---

### Score prediction model
- Retrieves classification results from SQLite.  
- Generates a **biodiversity score** for the monitored area based on detected species:  
   - **Score A** – High biodiversity  
   - **Score B** – Medium biodiversity  
   - **Score C** – Low biodiversity  
- Outputs results to an API in structured JSON format.


---

| Component                                 | Description                                                   |
|-------------------------------------------|---------------------------------------------------------------|
| Raspberry Pi + Audiomoth + 4G Router      | Collects audio data from the field                            |
| FTPS                                      | Secure file transfer protocol for uploading audio files       |
| iNet Server                               | Runs inference processes for species detection                |
| Bird Classification Model                 | Analyzes audio clips and identifies bird species + confidence |
| SQLite                                    | Stores classification results and associated metadata         |
| Score Prediction Model                    | Generates biodiversity scores from classification results     |
| API                                       | Receives JSON payloads containing final scoring results       |


---

## Pipeline file structure

```
inference-pipeline/
├── app-data/                    # Database files
├── audio-data/                  # Input audio recordings
├── docker/                      # Docker configuration files
├── json-output/                 # Prediction results and reports
├── logs/                        # Application logs
├── monsoon_biodiversity_common/ # Core library modules
│   ├── config.py                # Model and system configuration
│   ├── dataset.py               # Data loading and preprocessing
│   ├── db_model.py              # Database models and schema
│   ├── model.py                 # Neural network architecture
├── scripts/                     # Utility and deployment scripts
├── src/                         # Main application source code
│   ├── audio_monitoring.py      # Real-time audio monitoring
│   ├── process_detections.py    # Detection processing and reporting
│   └── species_mapping.py       # Species classification mapping (Thai - Eng)
├── weights/                     # Pre-trained model weights
└── requirements.txt             # Main project libraries
```

---

## Components



### Main functions

- **File Monitoring**: Watchdog-based file system monitoring for automatic processing
- **Audio Processing**: Real-time audio file monitoring and processing
- **Deep Learning Models**: Attention-based neural network for species classification and xgboost model for prediction the score of the area.
- **Database**: SQLite database for storing detections results
- **Docker Support**: Containerized deployment for development and production

Note: This deployment is already set up on iNet and running in production. Use this guide to deploy elsewhere.

---

#### File Monitoring

The monitoring service is implemented in **`src/audio_monitoring.py`**.  
Its function is to detect new audio files, check stability, and run inference.  

- **Watched path:** `/app/audio-data/` (including subfolders with `RPiID-*`)  
- **Supported formats:** `.wav`, `.ogg`, `.mp3`  
- **Queueing:** Uses `watchdog` with a thread-safe `queue` for sequential processing  

-**File stability check**  
The system performs a file stability check before processing any new audio file. It waits until the file size is both non-zero and remains unchanged for at least three seconds. If the file does not stabilize within a maximum wait time of 60 seconds, it is considered unstable or empty and is skipped.

**Inference process**  
The inference process uses the model stored at `/app/weights/soundscape-model.pt`. Each audio file is resampled to **48 kHz** and then divided into **5-second segments**. For every segment, the system predicts the class with the highest probability and records its confidence score. Each prediction is assigned a unique segment ID in the format `<file_stem>_<second>`.

**Logging and retention**  
All activity is logged in `/app/logs/audio_inference.log`. After processing, the audio files are retained by default, although a safe delete option exists but is disabled.

---

#### Audio processing & detection model  

This stage takes the daily detections from the database, turns them into structured features, predicts biodiversity scores, and delivers results to the API.

- For aggregation and scoring, detections are read from the SQLite database each day and grouped by device and hour. An XGBoost model (`/app/weights/xgboost-model.pkl`) processes these counts, fills in any missing species with zeros, and predicts a score. The device’s daily score is then determined by the majority of its hourly predictions. 

- For species information, each detected species is categorized as either a **bird** or an **insect**, with both English and Thai names provided from `species_mapping.py`.

- The final results are saved as JSON files in `json-output/predictions_<YYYY-MM-DD>.json`. Each file contains the **device ID, date, location, daily score, and detailed per-species data**.

- These results are also sent to the API using **OAuth authentication**. If delivery fails, the system retries up to **10 times** with exponential backoff. All activities related to result delivery are logged in `/app/logs/daily_report.log`.



- Example Payload per Device
```json
{
  "<DEVICE_ID>": [
    {
      "date": "YYYYMMDD",
      "coordinate": [18.8018, 98.9948],
      "score": "A",
      "species": [
        {
          "name_en": "Blue-throated Barbet",
          "name_th": "นกโพระดกคางฟ้า",
          "type": "bird",
          "data": ["0", "1", "45", "23", "56", "12", "2", "0", "5", "9", "19", "21", "22", "34", "61", "23", "12", "6", "87", "112", "22", "46", "23", "11"]
        }
      ]
    }
  ]
}
```
---


#### Deep learning models

This AI models can be read from [AI Models](ai.md).

#### Database schema

The database schema is defined using SQLAlchemy ORM. The schema consists of three main tables, each with relationships and constraints as described below:

**1. RpiDevices**
- **Purpose:** Stores information about each Raspberry Pi device.
- **Key Columns:**
  - `id` (Primary Key)
  - `pi_id` (Unique string identifier for the device)
  - `pi_type` (Integer: 0 = Mobile, 1 = Station; default is 1)
- **Relationships:** One-to-many with `AudioFiles` (a device can have multiple audio files).

**2. AudioFiles**
- **Purpose:** Records metadata for each audio file collected.
- **Key Columns:**
  - `id` (Primary Key)
  - `device_id` (Foreign Key to `RpiDevices.id`)
  - `recording_date` (Date of recording)
  - `file_key` (Unique file identifier)
- **Constraints:** Unique constraint on (`device_id`, `recording_date`, `file_key`) to prevent duplicate entries per device and date.
- **Relationships:** One-to-many with `SpeciesDetections` (an audio file can have multiple detections).

**3. SpeciesDetections**
- **Purpose:** Stores the results of species detection for each audio file segment.
- **Key Columns:**
  - `id` (Primary Key)
  - `audio_file_id` (Foreign Key to `AudioFiles.id`)
  - `species_class` (Detected species name)
  - `confidence_score` (Detection confidence, float)
  - `created_at` (Timestamp of detection)
  - `time_segment_id` (String identifier for the segment within the audio file)
- **Relationships:** Many-to-one with `AudioFiles`.

**Entity Relationship Overview:**
- Each `RpiDevices` entry can have multiple `AudioFiles`.
- Each `AudioFiles` entry can have multiple `SpeciesDetections`.

![Database conceptual design](images/db_design.png)

---

#### Docker support

1. **Build production image**
   ```
   sh scripts/build_prod_image.sh
   ```

2. **Run with docker-compose**
   ```
   docker compose -f docker/docker-compose-prod.yaml up -d
   ```

3. **Access container**
   ```
   docker exec -it prod-bio-service bash
   ```

4. **Check logs**
   ```
   docker logs -f prod-bio-service 
   ```

---

## Example usage


### Installation

1. **Clone the repository (with submodules)**
   ```bash
   git clone <repository-url>
   cd inference-pipeline
   git submodule update --init --recursive
   ```

2. **Download model weights**
   Download from Google Drive and place in the `weights/` directory:
   - `weights/soundscape-model.pt` — sound classification model
   - `weights/xgboost-model.pkl` — score prediction model
   - Google Drive: [Weights link](https://drive.google.com/drive/folders/1y59QnqjmbVWW-pZhIONdZJuEHzz-iiYz?usp=sharing)
   - Ensure the model architecture matches `monsoon_biodiversity_common/config.py`


3. **Verify logs (JSON sending and daily detections)**
   - Follow container logs:
     ```bash
     docker logs -f prod-bio-service
     ```
   - Inspect detailed log files inside the container:
     ```bash
     docker exec -it prod-bio-service bash
     tail -n 200 -f /app/logs/audio_inference.log   # file monitoring & segment inference
     tail -n 200 -f /app/logs/daily_report.log      # daily aggregation & API sending
     ```
   - Expected entries:
     - Saved JSON: lines with "[SAVE] JSON saved to"
     - Successful API send: lines with "[API] Prediction sent"
     - Daily aggregation: lines with "[DATE] Generating report for:" and device/file counts
     - Real-time processing: lines with "[NEW FILE]", "[INFER] Processing", and "[DB] Added <k> detections"

---



