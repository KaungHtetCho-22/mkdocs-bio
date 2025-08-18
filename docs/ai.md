# Biodiversity Score Prediction
## Outlines
<!-- - [Overview](#ov)
- [Data](#data)
- [Models](#models)
- [Trainings](#trainings) -->

<a id="ov"></a>
## Overview of Biodiversity Score Level Prediction
This project predicts regional biodiversity scores through a two-stage workflow <a href="#bioscoreoverview">Figure 1</a>: **1. bird and insect sound classification** and the **2. biodiversity score level prediction**.  

1. Bird and Insect Sound Classification
Audio recordings are collected by deployed AudioMoth devices. As illustrated in <a href="#bioscoreoverview">Figure 1</a>, the recordings are preprocessed and fed into a deep learning classifier based on a modified implementation of the [BirdCLEF 2023 4th Place Solution](https://www.kaggle.com/competitions/birdclef-2023/writeups/atfujita-4th-place-solution-knowledge-distillation). The model identifies bird and insect species and also detects non-biological sounds such as human speech, other human-generated noises, and vehicle sounds. It was pre-trained on species recordings from [Xeno-canto](https://xeno-canto.org/) and noise/no-call recordings from our own data collection.
2. Biodiversity Score Level Prediction
For each region, the frequency of occurrence of every detected species and noise class is aggregated from the classified recordings. These frequencies serve as input features to a traditional machine learning model (XGBoost), which predicts the region’s biodiversity score level: high, medium, or low.  

<!-- <figure id="bioscoreoverview" style="text-align: center;">
  <img 
    src="images/biodiversity_overview.png" 
    alt="Overview of Biodiversity Score Level Prediction" 
    width="400">
  <figcaption>Figure 1: Biodiversity Score Level Prediction Overview.</figcaption>
</figure> -->


![Biodiversity score level prediction overview](images/biodiversity_overview.png){ style="width: 400px; display: block; margin: 0 auto;" }

<p align="center"><em>Figure 1: Biodiversity score level prediction overview.</em></p>

<!-- 
<div style="text-align: center;">
<img src="images/biodiversity_overview.png" alt="Biodiversity Overview" width="400">
</div>
--- -->


<a id="data"></a>
## Data
This section provides an overview of the data used in this project. We summarize the sources, label quality, use in training, geographic filtering, and the class list.

### Sources
Public: Expert-labeled wildlife audio from [Xeno-canto](https://xeno-canto.org/), covering birds and insects.
Self-collected: Field recordings captured with AudioMoth devices in tea plantations around Chiang Mai, Thailand.
### Audio Data Labels
Xeno-canto recordings include expert-provided species labels.
The self-collected recordings lack ground-truth annotations.
### Data Usage
The sound-classification model is trained primarily on the labeled Xeno-canto data.
Additional noise examples from our self-collected recordings (e.g., human speech, noises from human activity, vehicles, and other environmental noises) are included to improve robustness.
### Geographic filtering
To reduce label noise and improve relevance, we removed species not known to occur in Thailand, with a particular focus on the Chiang Mai region.
### Class list
The final set of bird and insect species used in training and inference is documented in 
**[species.txt](files/species.txt)** and **[species_count.txt](files/species_count.txt)**.  
The number of classes included in the analysis is summarized in [Table 1](#table-1).


<a id="table-1"></a>
<!-- Table 1: Summary of the Number of Classes in Training Data
| Type of Sound | Class Count |
|-----------------|-------------------|
| Bird Species | 66 |
| Insect Species | 14 |
| Noise/No-Call | 10 |
| **Total** | **90** | -->

<a id="table-1"></a>

### Table 1 — Summary of the Number of Classes in Training Data

| **Type of Sound** | **Class Count** |
|-------------------|-----------------:|
| Bird Species      | 66 |
| Insect Species    | 14 |
| Noise / No-Call   | 10 |
| **Total**         | **90** |

---

<!-- 
### 2. Model Accuracy

| Metric   | Value |
|----------|-------|
| Accuracy | XX%   |
| Precision| XX%   |
| Recall   | XX%   |
| F1-Score | XX%   |

*(Replace XX% with actual results after training.)*

---
 -->
<a id="models"></a>
## Models

### Sound Classification Model

### Biodiversity Score Level Prediction Model
- **Input:** Mel-spectrograms extracted from audio recordings.
- **Feature Extraction:** Convolutional Neural Networks (CNNs) for spatial feature learning.
- **Classification Layer:** Fully-connected layers with softmax output for multi-class classification.
- **Training Details:**
  - Optimizer: Adam
  - Loss: Categorical Cross-Entropy
  - Learning Rate: *(e.g., 0.001)*
  - Epochs: *(e.g., 50)*
  - Batch Size: *(e.g., 32)*

