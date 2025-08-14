# **Bird Sound Monitoring & Scoring Pipeline**


---

## Overview

This system automates the end-to-end process of **monitoring bird sounds** using IoT devices, classifying them with a soundscape model, predicting biodiversity scores, and delivering the results as JSON payloads to an API.

<!-- <p align="center">
  <img src="images/overview-diagram.png" alt="System Overview Diagram" width="80%" />
</p>

<p align="center"><em>Figure 1: Bird Sound Monitoring &amp; Scoring Pipeline Overview</em></p> -->

![System Overview Diagram](images/overview-diagram.png){ style="width: 400px; display: block; margin: 0 auto;" }

<p align="center"><em>Figure 1: Bird Sound Monitoring &amp; Scoring Pipeline Overview</em></p>

---

### Data source – Raspberry Pi
- **Device:** Raspberry Pi with Audiomoth sensors for continuous field audio collection.  
- **Protocol:** FTPS (Secure FTP) for encrypted data transfer.  
- **Destination:** Audio files are securely uploaded to the **iNet private cloud**.

---

### Audio collection
- Captures **10-minute audio clips** in `.WAV` format.

---

### Bird classification model
- Processes audio clips using a **deep learning soundscape model**.  
- Identifies bird species with **confidence scores**.  
- Stores classification results in a **MySQL database** for further analysis.

---

### Score prediction model
- Retrieves classification results from MySQL.  
- Generates a **biodiversity score** for the monitored area based on detected species:  
   - **Score A** – High biodiversity  
   - **Score B** – Medium biodiversity  
   - **Score C** – Low biodiversity  
- Outputs results to an API in structured JSON format.


---

<table border="1" cellpadding="6" cellspacing="0" style="border-collapse: collapse; text-align: left; width: 100%;">
  <thead style="background-color: #f2f2f2;">
    <tr>
      <th style="border: 1px solid #ccc;">Component</th>
      <th style="border: 1px solid #ccc;">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ccc;">Raspberry Pi + Audiomoth + 4G Router</td>
      <td style="border: 1px solid #ccc;">Collects audio data from the field.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">FTPS</td>
      <td style="border: 1px solid #ccc;">Secure file transfer protocol for uploading audio files.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">iNet Server</td>
      <td style="border: 1px solid #ccc;">Runs inference processes for species detection.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">Bird Classification Model</td>
      <td style="border: 1px solid #ccc;">Analyzes audio clips and identifies bird species with confidence scores.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">MySQL</td>
      <td style="border: 1px solid #ccc;">Stores classification results and associated metadata.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">Score Prediction Model</td>
      <td style="border: 1px solid #ccc;">Generates biodiversity scores based on classification results.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;">API</td>
      <td style="border: 1px solid #ccc;">Receives JSON payloads containing final scoring results.</td>
    </tr>
  </tbody>
</table>


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

NOTES: **This deployment is already setup on the iNET and now it is working in actions.**
This documentation guides for making the deployment anywhere else.

---

#### File monitoring 

#### Audio processing   

#### Deep learning models

This AI models can be read from [AI Models](ai.md).

#### Database schema

This is how the conceptual diagram works inside the inference data accepting


<table border="1" cellpadding="6" cellspacing="0" style="border-collapse: collapse; text-align: left; width: 100%;">
  <thead style="background-color: #f2f2f2;">
    <tr>
      <th style="border: 1px solid #ccc;">Table Name</th>
      <th style="border: 1px solid #ccc;">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ccc;"><strong>RpiDevices</strong></td>
      <td style="border: 1px solid #ccc;">Stores device information and associated metadata.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;"><strong>AudioFiles</strong></td>
      <td style="border: 1px solid #ccc;">Contains audio file records along with relevant metadata.</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc;"><strong>SpeciesDetections</strong></td>
      <td style="border: 1px solid #ccc;">Holds species detection results with confidence scores and related attributes.</td>
    </tr>
  </tbody>
</table>



![Database conceptual design](images/db_design.png)

#### Docker support



## Quick start




### Model configuration

Edit `monsoon_biodiversity_common/config.py` to customize:

- **Audio parameters**: Sample rate, mel bands, FFT settings
- **Model architecture**: Backbone model, number of classes
- **Training settings**: Learning rate, batch size, epochs

---

## Example usage

### Prerequisites

- Python 3.8+
- Audio processing libraries (librosa, torchaudio)
- Deep learning framework (PyTorch)

### Installation

1. **Clone the repository**
This one is already clone at iNET (server4 machine)

   ```bash
   git clone <repository-url>
   cd inference-pipeline
   git submodule update --init --recursive 
   ```

2. **Download model weights**
The weights are already uploaded to the iNET and attached to the drive as well

   - Place `soundscape-model.pt` file in the `weights/` directory
   - sound-scape.pt = sound classification model 
   - xgboost-model.pkl = score prediction model 
   - Ensure the model architecture matches the configuration in `monsoon_biodiversity_common/config.py`

3. **Setup database**
   ```bash
   # The database will be automatically initialized on first run
   python src/debug_audio_monitoring.py
   ```

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


### Real-time Audio Monitoring

Start the audio monitoring service:

```bash
python src/audio_monitoring.py
```

This service:
- Monitors audio directories for new files
- Processes audio files through the species detection model
- Stores results in the database
- Generates real-time logs

<!-- ### Batch Processing

Process existing audio files:

```bash
python src/inference_station.py
```

### Query Results

Query detection results from the database:

```bash
python src/query.py
```

### Daily Reports

Generate daily detection summaries:

```bash
python src/debug_process_detections.py --schedule
``` -->

<!-- 

## Output Formats

### Detection Results

Species detections are stored with:
- Audio file reference
- Species classification
- Confidence score
- Temporal segment information
- Device and timestamp metadata

### Log Files

- `audio_inference.log`: Real-time processing logs
- `batch_audio_inference.log`: Batch processing logs
- `daily_report.log`: Daily summary reports

## Troubleshooting

### Common Issues

1. **Model weights not found**
   - Ensure `soundscape-model.pt` is in the `weights/` directory
   - Check file permissions and paths

2. **Audio directory not accessible**
   - Verify audio data directory exists and is readable
   - Check Docker volume mounts if using containers

3. **Database connection errors**
   - Ensure SQLite database directory is writable
   - Check database file permissions -->


