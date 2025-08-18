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

---

### Main functions

- **File Monitoring**: Watchdog-based file system monitoring for automatic processing
- **Audio Processing**: Real-time audio file monitoring and processing
- **Deep Learning Models**: Attention-based neural network for species classification and xgboost model for prediction the score of the area.
- **Database**: SQLite database for storing detections results
- **Docker Support**: Containerized deployment for development and production

Note: This deployment is already set up on iNet and running in production. Use this guide to deploy elsewhere.

---

#### File Monitoring

**Location:**  
`src/audio_monitoring.py`

**Purpose:**  
Continuously watches incoming audio, validates file stability, performs 5‑second window classification, and writes per‑segment detections to the database with resilient logging.

**Watched paths and filters:**
- **Root directory:** `/app/audio-data/`
- **Subfolders monitored:** Only folders whose names contain `RPiID` (e.g., `RPiID-001`), monitored recursively
- **File types:** `.wav`, `.ogg`, `.mp3`

**Monitoring and queueing:**
- Uses `watchdog` (`Observer` + `FileSystemEventHandler`) to enqueue new files into a thread-safe `queue.Queue`
- A dedicated background thread consumes the queue and processes files sequentially

**File stability gate:**
- Before inference, the service waits for the file size to remain unchanged and non‑zero for 3 consecutive seconds
- Maximum wait window: 60 seconds; unstable or empty files are skipped with an error log

**Inference details:**
- Loads `AttModel` with weights at `/app/weights/soundscape-model.pt`
- Audio loaded with `librosa` at 32 kHz; split into 5‑second segments: seconds = 5, 10, 15, ... up to clip length
- For each segment, computes sigmoid probabilities for `CFG.target_columns` and selects:
  - `Class`: argmax label
  - `Score`: max probability
- Segment identifier `row_id` format: `<file_stem>_<second>`

<!-- **Database write (SQLite):**
- DB URL: `sqlite:////app/app-data/soundscape-model.db`
- Entities used: `RpiDevices`, `AudioFiles`, `SpeciesDetections`
- Path parsing: expects folder layout `.../RPiID-XXX/YYYY-MM-DD/<audio_file>`
- Unique `AudioFiles.file_key`: `<pi_id>_<recording_date>_<audio_filename>`
- For each segment, inserts a `SpeciesDetections` row with:
  - `time_segment_id` = `row_id`
  - `species_class` = predicted `Class`
  - `confidence_score` = `Score`
  - `created_at` = current UTC timestamp -->

**Logging:**
- File: `/app/logs/audio_inference.log`

**File retention:**
- Files are currently kept after processing; a safe delete helper exists but is disabled by default


#### Audio processing & detection model  


**Daily aggregation and scoring:**
- Loads an XGBoost model from `/app/weights/xgboost-model.pkl`
- Reads detections from SQLite (`sqlite:///app-data/soundscape-model.db`) for a target date (default: yesterday)
- Builds a feature table per device and hour by counting detections per species
- Ensures a fixed feature set (`SELECTED_FEATURES`), filling missing species with zeros
- Predicts per‑row scores and assigns the device score by majority vote across its hourly rows

**Hourly bucketing logic:**
- `AudioFiles.file_key` encodes a start time: `..._<YYYY-MM-DD>_<HH>-<MM>-<SS>`
- For each `SpeciesDetections.time_segment_id` (`..._<relative_second>`), computes `absolute_second = start_time_in_seconds + relative_second`
- Buckets into `hour = clamp(floor(absolute_second / 3600), 0..23)`; fallback `hour=0` if parsing fails

**Species categorization and localization:**
- Species are tagged as `bird` or `insect` via sets in `process_detections.py`
- English/Thai display names are pulled from `SPECIES_INFO` in `species_mapping.py`

**JSON output (saved and sent):**
- Saves to `json-output/predictions_<YYYY-MM-DD>.json`
- Payload per device:
```json
{
  "<DEVICE_ID>": [
    {
      "date": "YYYYMMDD",
      "coordinate": [18.8018, 98.9948],
      "score": 5,
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

**API delivery and retries:**
- Obtains OAuth token using env vars: `TOKEN_URL`, `CLIENT_ID`, `CLIENT_SECRET`, `API_USERNAME`, `API_PASSWORD`
- Sends JSON to `API_URL` with `Authorization: Bearer <token>`
- Up to 10 attempts with exponential backoff; skips sending if token cannot be obtained
- Logs to `/app/logs/daily_report.log`

**CLI usage:**
- `python process_detections.py --date YYYY-MM-DD` — run for a specific date
- `python process_detections.py --now` — run immediately for today
- `python process_detections.py --schedule` — run daily at 23:59

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



