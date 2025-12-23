# DeepSeek OCR Finetuning

This folder contains the DeepSeek-OCR finetuning workflow. Use the commands below to prepare data, train with LoRA, and push the adapter to Hugging Face.

## 1. Setup
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## 2. Prepare data
The processor downloads and formats Spider into `processed_data/deepseek_finetune_data.jsonl` within this repo (no external paths).
```bash
python data_processor.py
```

## 3. Train
This runs LoRA finetuning and writes the adapter and processor into `deepseek_ocr_finetuned/`.
```bash
python finetune_deepseek_ocr.py
```

## 4. Push to Hugging Face
Authenticate and upload the training artifacts from `deepseek_ocr_finetuned/`.
```bash
export HF_TOKEN=YOUR_HF_TOKEN
export HF_REPO=YOUR_HF_REPO # e.g. JohnnyZeppelin/sql-ocr

huggingface-cli login --token "$HF_TOKEN"
python - <<'PY'
from huggingface_hub import HfApi
from pathlib import Path
import os

api = HfApi()
repo_id = os.environ["HF_REPO"]
local_dir = Path("deepseek_ocr_finetuned")
api.create_repo(repo_id, private=False, exist_ok=True)
api.upload_folder(folder_path=local_dir, repo_id=repo_id)
print(f"Uploaded {local_dir} to {repo_id}")
PY
```

## Snow → Train → Push in one go
If you want to run the entire flow with your provided Hugging Face repo/token in a single command, set your credentials and chain the steps:
```bash
export HF_TOKEN=YOUR_HF_TOKEN
export HF_REPO=JohnnyZeppelin/sql-ocr

huggingface-cli login --token "$HF_TOKEN" && \
python data_processor.py && \
python finetune_deepseek_ocr.py && \
python - <<'PY'
from huggingface_hub import HfApi
from pathlib import Path
import os

api = HfApi()
repo_id = os.environ["HF_REPO"]
local_dir = Path("deepseek_ocr_finetuned")
api.create_repo(repo_id, private=False, exist_ok=True)
api.upload_folder(folder_path=local_dir, repo_id=repo_id)
print(f"Uploaded {local_dir} to {repo_id}")
PY
```

The commands above follow the requested **Snow → train → push** flow: data is prepared (Snow), finetuning runs (train), and the resulting adapter is uploaded (push). Replace the placeholders with your Hugging Face token when running locally.
