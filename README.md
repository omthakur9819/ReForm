# ♻️ ReForm — E-Waste Plastic Identification and Reuse System

ReForm is a Streamlit web app that helps identify plastic types in e-waste items using a two-stage MobileNetV2 classifier and EasyOCR-based resin code scanning. It provides material identification, hazard warnings, handling rules, and reuse/recycling guidance based on India's **E-Waste (Management) Rules 2022**.

---

## 🖥️ Demo

| Step 1 — Item photo | Step 2 — Resin code scan |
|---|---|
| Upload or capture a photo of the e-waste item | Scan the moulded ISO 11469 resin code (`>ABS<`, `>PP<`, etc.) |
| Stage 0 classifies the **material** (plastic / metal / PCB / ...) | EasyOCR + fuzzy matching identifies the **polymer type** |
| Stage 1 classifies the **item type** if plastic | Result: hazards, handling rules, reuse options |

---

## 🏗️ Architecture

```
Step 1 photo
    │
    ▼
Stage 0: MobileNetV2 material sorter (6 classes)
    │  plastic → Stage 1
    │  metal / PCB / glass / battery / unknown → nonplastic_routes.json
    ▼
Stage 1: MobileNetV2 item classifier (7 classes, plastic only)
    │
    ▼
Step 2 (optional): close-up resin-code photo
    │  EasyOCR + OpenCV preprocessing + fuzzy match → polymer confirmed
    ▼
Result: material + item + plastic type + BFR hazards + handling + reuse + regulation
```

**Explainability:** Grad-CAM heatmaps are overlaid on both Stage 0 and Stage 1 predictions so users can see what region of the image the model focused on.

---

## 🔍 What It Can Identify

### Stage 0 — Materials (6 classes)
`plastic` · `metal` · `PCB` · `glass_screen` · `battery` · `unknown`

### Stage 1 — Plastic Item Types (7 classes)
`keyboard` · `mouse` · `printer_casing` · `tv_panel` · `remote` · `other_plastic` · `unknown`

### Step 2 — Resin Codes (ISO 11469)
`>ABS<` · `>PC+ABS<` · `>PC<` · `>PS<` · `>HIPS<` · `>PP<` · `>PVC<` · `>PE<` · `>HDPE<` · `>LDPE<` · `>PET<` · `>ABS+PC<`

---

## 🚀 Setup

### Prerequisites
- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or Anaconda
- Python 3.11
- *(Optional)* NVIDIA GPU with CUDA 11.8 + cuDNN 8.6 for faster inference

### 1. Create the conda environment

```bash
conda create -n reform python=3.11 -y
conda activate reform
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Download pre-trained models

The trained `.keras` model files are too large for GitHub. Download them and extract into the `models/` folder:

> 📦 **[Download reform_models.zip](#)** ← *(replace with your Drive/Release link)*

```
models/
├── stage0_model.keras
├── stage0_classes.json
├── stage1_model.keras
└── stage1_classes.json
```

### 4. Run the app

```bash
conda activate reform
streamlit run app.py
```

Then open http://localhost:8501 in your browser.

---

## 🧠 Training Your Own Models

Open `train.ipynb` in Jupyter and run cells in order:

```bash
conda activate reform
jupyter notebook train.ipynb
```

**Cell order:** Cell 1 → Cell 3 (or 4) → Cell 7 → Cell 9 → Cell 10 → Cell 14

Training takes ~45–60 min on an RTX 4050. On CPU (e.g. Colab free tier) expect several hours.

Dataset is sourced from Roboflow's "E-Waste Detector v1i" and garbage-classification datasets. Use `download_datasets.py` and `organise_dataset.py` to prepare `data/stage0/` and `data/stage1/`.

---

## 📁 Project Structure

```
ReForm/
├── app.py                  # Streamlit UI (main entry point)
├── gradcam.py              # Grad-CAM heatmap generation
├── train.ipynb             # Model training notebook
├── knowledge_base.json     # Hazards, reuse, regulation info per item
├── nonplastic_routes.json  # Recovery routing for non-plastic materials
├── requirements.txt        # Pinned dependencies
├── download_datasets.py    # Kaggle dataset downloader
├── organise_dataset.py     # Converts source classes → project classes
├── check_env.py            # Verify GPU + package versions
├── data/
│   ├── stage0/             # Train/val/test images for material sorter
│   └── stage1/             # Train/val/test images for item classifier
└── models/                 # Trained .keras models + classes.json maps
```

---

## 📋 Regulation Note

Results reference **India's E-Waste (Management) Rules 2022** (EPR framework). All guidance is indicative — verify against current local regulations before acting.

---

## ⚠️ Known Limitations

- A photo alone **cannot** identify polymer type — ABS, PC, and HIPS look identical visually. Step 2 (resin code OCR) is needed for a confirmed polymer identification.
- Confidence threshold is 70%. Photos should be well-lit with the item filling most of the frame.
- BFR (Brominated Flame Retardant) risk levels are based on typical usage for each item class, not measured values.

---

## 🛠️ Tech Stack

| Component | Library |
|---|---|
| UI | Streamlit |
| Vision models | TensorFlow 2.15 / Keras (MobileNetV2) |
| Image processing | OpenCV 4.11 |
| OCR | EasyOCR 1.7.2 |
| Fuzzy matching | thefuzz |
| Explainability | Grad-CAM (custom `gradcam.py`) |
