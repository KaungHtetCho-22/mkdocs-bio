# AI Models

The AI module in the Biodiversity Project is composed of two main parts:

1. **Sound Classification System**  
   - Identifies bird and insect species, as well as noise categories, from recorded audio.
2. **Score Prediction Model**  
   - Generates a biodiversity score based on the outputs of the sound classification system.

---

## Sound Model Training

This section describes the process and details for training the sound classification model.

---

### 1. Training Dataset

The dataset consists of **bird species**, **insect species**, and **noise classes**.

#### 1.1 Bird Species

| Common Name | Biological Name | Number of Samples |
|-------------|-----------------|-------------------|
|  Asian Koel | *Eudynamys scolopaceus* | 1200 |
| *(Add more rows here)* | | |

#### 1.2 Insect Species

| Common Name | Biological Name | Number of Samples |
|-------------|-----------------|-------------------|
|  Cicada | *Cicadidae* | 800 |
| *(Add more rows here)* | | |

#### 1.3 Noise Classes

| Class Name | Description | Number of Samples |
|------------|-------------|-------------------|
| *(Example)* Rain | Background rain noise | 500 |
| *(Add more rows here)* | | |

---

### 2. Model Accuracy

| Metric   | Value |
|----------|-------|
| Accuracy | XX%   |
| Precision| XX%   |
| Recall   | XX%   |
| F1-Score | XX%   |

*(Replace XX% with actual results after training.)*

---

### 3. Model Architecture

The sound classification system is based on:

- **Input:** Mel-spectrograms extracted from audio recordings.
- **Feature Extraction:** Convolutional Neural Networks (CNNs) for spatial feature learning.
- **Classification Layer:** Fully-connected layers with softmax output for multi-class classification.
- **Training Details:**
  - Optimizer: Adam
  - Loss: Categorical Cross-Entropy
  - Learning Rate: *(e.g., 0.001)*
  - Epochs: *(e.g., 50)*
  - Batch Size: *(e.g., 32)*

---

**Next:** [Score Prediction Model](score.md)
