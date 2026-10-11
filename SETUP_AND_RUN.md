# 🔬 MalariaScope AI — Setup & Execution Guide

**Author:** Dhanshree Lonare  
**Copyright:** © 2026 Dhanshree Lonare. All rights reserved.  
**Project Repository:** [malaria-detection-using-deep-learning-](https://github.com/dhanshreelonare/malaria-detection-using-deep-learning-)

---

## 📋 Table of Contents
1. [Overview](#-overview)
2. [System Requirements](#-system-requirements)
3. [Environment Setup](#-environment-setup)
4. [Dataset Download & Preparation](#-dataset-download--preparation)
5. [Training the Deep Learning Models](#-training-the-deep-learning-models)
6. [Launching the Web Application Dashboard](#-launching-the-web-application-dashboard)
7. [API Endpoints Reference](#-api-endpoints-reference)
8. [Troubleshooting & FAQs](#-troubleshooting--faqs)

---

## 🌟 Overview

MalariaScope AI is an end-to-end clinical deep learning system designed for the rapid, automated detection of *Plasmodium* malaria parasites in Giemsa-stained thin blood smear microscopic images. The project encompasses:
- Three deep neural network architectures:
  1. **Custom Deep CNN** (Optimized 4-stage convolutional neural network)
  2. **MobileNetV2** (Lightweight inverted residual architecture for edge devices)
  3. **EfficientNetB0** (Compound-scaled transfer learning architecture)
- Comprehensive evaluation metrics, confusion matrices, ROC curves, and training history plots.
- Modern Flask web application featuring glassmorphism dark-mode UI, real-time image drag-and-drop diagnostic inference, and confidence score visualization.

---

## 💻 System Requirements

### Hardware Specifications
- **Processor:** Intel Core i5 / AMD Ryzen 5 or higher (Apple M1/M2/M3 or modern x86_64)
- **RAM:** Minimum 8 GB (16 GB recommended for model training)
- **GPU (Optional):** NVIDIA GPU with CUDA 11.8+ / 12.x for accelerated training (CPU training supported)
- **Storage:** Minimum 5 GB free disk space (for dataset, weights, and logs)

### Software Prerequisites
- **Operating System:** Windows 10/11, Ubuntu 20.04+, or macOS
- **Python:** Python 3.10, 3.11, 3.12, or 3.13
- **Git:** Git 2.30+ installed

---

## ⚙️ Environment Setup

### 1. Clone the Repository
```bash
git clone https://github.com/dhanshreelonare/malaria-detection-using-deep-learning-.git
cd malaria-detection-using-deep-learning-
```

### 2. Create a Python Virtual Environment
It is strongly recommended to use a virtual environment to isolate project dependencies:

**On Windows (PowerShell):**
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

**On Windows (Command Prompt):**
```cmd
python -m venv venv
venv\Scripts\activate.bat
```

**On Linux / macOS:**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Required Dependencies
Install the required packages from `requirements.txt`:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Verify the installation of critical libraries:
```bash
python -c "import tensorflow as tf; print('TensorFlow version:', tf.__version__)"
python -c "import flask; print('Flask version:', flask.__version__)"
```

---

## 📦 Dataset Download & Preparation

The dataset used is the NIH / Lister Hill National Center for Biomedical Communications Malaria Cell Image Dataset, hosted on Kaggle:
- **Total Images:** 27,558 single-cell images
- **Classes:** 
  - `Parasitized`: 13,779 images
  - `Uninfected`: 13,779 images

### Method 1: Automated Download via KaggleHub
Execute the provided downloader script:
```bash
python download_dataset.py
```
This downloads the dataset to your local cache (`~/.cache/kagglehub/...`) and prints the storage path.

### Method 2: Manual Download
1. Download from Kaggle: [Cell Images for Detecting Malaria](https://www.kaggle.com/datasets/iarunava/cell-images-for-detecting-malaria)
2. Extract the archive into a directory named `cell_images/` inside the project folder:
   ```
   malaria_dhanu/
   ├── cell_images/
   │   ├── Parasitized/
   │   │   ├── ...png
   │   └── Uninfected/
   │       └── ...png
   ```

---

## 🚀 Training the Deep Learning Models

To train all three neural network architectures, generate confusion matrices, ROC curves, and export serialized model files (`.h5`):

```bash
python malaria_detection.py
```

### What `malaria_detection.py` Executes:
1. **Data Ingestion & Verification:** Cleans unreadable images and balances splits.
2. **Train / Validation / Test Splitting:** Standard 70% / 15% / 15% stratified split.
3. **Data Augmentation:** Real-time geometric transformations (random rotation, shifts, flips, zoom) to prevent overfitting.
4. **Model Architecture Compilation:**
   - **Custom CNN:** 4 Convolutional blocks + BatchNormalization + Dropout (0.3 - 0.5) + Dense layers.
   - **MobileNetV2:** ImageNet pre-trained feature extractor + GlobalAveragePooling2D + Custom classifier head.
   - **EfficientNetB0:** High-efficiency compound scaling backbone + Fine-tuning.
5. **Training Callbacks:** `EarlyStopping` (patience=5, restore best weights) and `ReduceLROnPlateau`.
6. **Artifact Generation:**
   - `CustomCNN_best.h5`
   - `MobileNetV2_best.h5`
   - `EfficientNetB0_best.h5`
   - `CustomCNN_confusion_matrix.png` & `CustomCNN_roc_curve.png`
   - `MobileNetV2_confusion_matrix.png` & `MobileNetV2_training_history.png`

---

## 🌐 Launching the Web Application Dashboard

The interactive clinical inference portal allows medical laboratory scientists to upload blood cell micrographs and receive instant diagnoses.

### 1. Start the Flask Server
```bash
python app.py
```

Console output will display:
```
 * Serving Flask app 'app'
 * Debug mode: off
 * Loading deep learning model...
 * Successfully loaded model: CustomCNN_best.h5
 * Running on http://127.0.0.1:5000
```

### 2. Access the Application
Open your web browser and navigate to:
```
http://127.0.0.1:5000
```
or
```
http://localhost:5000
```

### 3. Performing a Diagnosis
1. Click the upload zone or drag-and-drop a `.png`, `.jpg`, or `.jpeg` blood cell image.
2. The UI displays an instant thumbnail preview and cell metadata.
3. Click **"Run Diagnostic Analysis"**.
4. The system computes binary classification probabilities and outputs:
   - **Diagnostic Outcome:** `Parasitized (Positive)` or `Uninfected (Negative)`.
   - **Confidence Score:** Percentage certainty (e.g., 99.84%).
   - **Severity / Clinical Advice:** Automated next steps for verification.
   - **Model Details:** Active neural architecture utilized.

---

## 📡 API Endpoints Reference

| Method | Endpoint | Description | Payload / Response |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | Web portal UI | HTML page |
| `POST` | `/predict` | Single-image inference | Multipart `image` file $\rightarrow$ JSON `{status, label, confidence, risk_level, model_used}` |
| `GET` | `/models` | Available & active models | JSON `{active_model, available_models}` |
| `GET` | `/health` | Service health status | JSON `{status: "healthy", timestamp: "..."}` |

---

## 🛠️ Troubleshooting & FAQs

### Q1: `No module named pip` or environment issues
**Resolution:** Ensure you create the virtual environment using Python's standard `venv` module:
```bash
python -m venv venv
```

### Q2: Port 5000 is already in use
**Resolution:** Run the app on an alternative port by modifying `app.py` or running:
```bash
python -c "import app; app.app.run(port=5001, debug=True)"
```

### Q3: Model weights file not found
**Resolution:** Ensure that `CustomCNN_best.h5`, `MobileNetV2_best.h5`, or `EfficientNetB0_best.h5` is in the root directory. `app.py` automatically scans and falls back to any `.h5` file present in the directory.

### Q4: UTF-8 encoding errors on Windows Command Prompt
**Resolution:** `app.py` and `malaria_detection.py` include automatic standard stream re-encoding wrappers (`sys.stdout.reconfigure(encoding='utf-8')`). You can also run:
```cmd
chcp 65001
```

---

**© 2026 Dhanshree Lonare. All rights reserved.**
