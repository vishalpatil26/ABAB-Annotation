# 🎙️ AI Speech Annotation Pipeline

Automated ABAB-style speech annotation using **WhisperX** (transcription) + **pyannote.audio** (speaker diarization).

---

## 📁 Project Structure

```
speech_annotation/
├── pipeline.py        ← Core AI pipeline (VAD + diarization + transcription)
├── api.py             ← FastAPI REST backend
├── app.py             ← Gradio web UI
├── requirements.txt   ← Python dependencies
└── README.md
```

---

## ⚙️ Setup Instructions

### 1. Prerequisites

- Python 3.9 – 3.11
- CUDA GPU strongly recommended (NVIDIA, with CUDA 11.8 or 12.x)
- ffmpeg installed on your system

```bash
# Ubuntu / Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg

# Windows (with winget)
winget install ffmpeg
```

---

### 2. Create a Virtual Environment

```bash
python -m venv venv

# Linux / macOS
source venv/bin/activate

# Windows
venv\Scripts\activate
```

---

### 3. Install PyTorch (GPU)

> Skip if you only have CPU (slower but works).

```bash
# CUDA 11.8
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu118

# CUDA 12.1
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu121
```

---

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

---

### 5. Get a Hugging Face Token (REQUIRED for diarization)

pyannote models are gated — you must:

1. Create a free account at https://huggingface.co
2. Go to https://hf.co/pyannote/speaker-diarization-3.1 and **accept the license**
3. Go to https://hf.co/pyannote/segmentation-3.0 and **accept the license**
4. Generate a token at https://huggingface.co/settings/tokens

Then set it as an environment variable:

```bash
# Linux / macOS
export HF_TOKEN="hf_your_token_here"

# Windows
set HF_TOKEN=hf_your_token_here
```

---

## 🚀 Running the System

### Option A — Gradio Web UI (Recommended for beginners)

```bash
cd speech_annotation
python app.py
```

Open your browser at: **http://localhost:7860**

- Upload a `.wav` or `.mp3` file
- Click **▶ Run Annotation**
- Edit the transcript in the text box
- Export as JSON or CSV

---

### Option B — FastAPI REST Backend

```bash
cd speech_annotation
python api.py
```

API available at: **http://localhost:8000**
Swagger docs at: **http://localhost:8000/docs**

#### Example API calls:

```bash
# Annotate and get JSON response
curl -X POST http://localhost:8000/annotate \
  -F "file=@your_audio.wav"

# Annotate and download JSON file
curl -X POST "http://localhost:8000/annotate/export?format=json" \
  -F "file=@your_audio.wav" \
  -o annotation.json

# Annotate and download CSV file
curl -X POST "http://localhost:8000/annotate/export?format=csv" \
  -F "file=@your_audio.wav" \
  -o annotation.csv
```

---

## 📤 Output Format

### ABAB Text
```
A 00:01 Hello how are you
B 00:04 I'm fine thank you
A 00:07 Are you coming tomorrow
```

### JSON
```json
[
  { "speaker": "A", "start": 1.2, "end": 3.8, "start_fmt": "00:01", "end_fmt": "00:03", "text": "Hello how are you" },
  { "speaker": "B", "start": 4.1, "end": 6.5, "start_fmt": "00:04", "end_fmt": "00:06", "text": "I'm fine thank you" }
]
```

### CSV
```
speaker,start,end,text
A,00:01,00:03,Hello how are you
B,00:04,00:06,I'm fine thank you
```

---

## ⚡ GPU Optimization (for 100h/day throughput)

| Setting | Value |
|---|---|
| Whisper model | `large-v2` (most accurate) or `medium` (faster) |
| Compute type | `float16` on GPU |
| Batch size | 16 (auto, can increase to 32 on A100) |

```bash
# Use a larger model for better accuracy
export WHISPER_MODEL=large-v2
```

**Throughput estimates (A100 GPU):**
- `base` model → ~50× real-time → ~1200h/day
- `large-v2` model → ~8× real-time → ~192h/day

For 100h/day you only need a single mid-range GPU (e.g., RTX 3080).

---

## 🔧 Environment Variables

| Variable | Default | Description |
|---|---|---|
| `HF_TOKEN` | *(required)* | Hugging Face token for pyannote |
| `WHISPER_MODEL` | `base` | Whisper model size (`base`, `small`, `medium`, `large-v2`) |

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| `EnvironmentError: HF_TOKEN not set` | Set your HF token (see step 5) |
| `CUDA out of memory` | Use a smaller Whisper model or reduce batch_size |
| `No speech detected` | Check audio quality; ensure it's not silent |
| `ffmpeg not found` | Install ffmpeg (see step 1) |
| Slow on CPU | Use GPU or reduce model size to `base` |# ABAB-Annotation
