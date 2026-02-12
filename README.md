# Handwritten Arabic Bill OCR Backend

Professional, modular Python backend skeleton for a university junior project focused on OCR for handwritten Arabic bills.

## Project Goal

Build an end-to-end OCR backend pipeline for handwritten Arabic bills:

1. Image Alignment
2. Text Detection
3. Handwriting Recognition
4. LLM Correction

This repository currently contains the architecture skeleton (folders and module stubs with docstrings) and is ready for incremental implementation.

## Architecture Overview

The codebase is organized into three major layers:

- `src/data/pipelines`: assembly-line execution of OCR stages (preprocess -> process -> postprocess).
- `src/strategies`: interchangeable algorithm implementations (e.g., YOLO vs contour detection).
- `src/Engine` + `src/Solvers`: orchestration and decision logic for end-to-end request handling.

## OCR Pipeline

- **Pre-processing**
  - Image cleaning and normalization (OpenCV-based stage design).
  - Image alignment and perspective correction.
- **Processing**
  - Text region detection (YOLO/traditional strategy swap).
  - Handwriting recognition (TrOCR and future alternatives).
- **Post-processing**
  - LLM-based text correction and refinement (Ollama wrapper layer).

## Directory Guide (With Examples)

- `data/`: local dataset storage.
  - Example: put original phone photos in `data/raw/` and generated clean training crops in `data/synthetic/`.
- `models/`: local model artifacts.
  - Example: `models/yolo_detector.pt`, `models/trocr_model.safetensors`.
- `notebooks/`: experiment notebooks.
  - Example: `notebooks/discovery.ipynb` for trying preprocessing parameters.
- `scripts/`: automation entrypoints.
  - Example: `generate_synthetic_data.py` to create augmented bill images, `evaluate_model.py` to compute CER/WER.
- `tests/`: pytest unit + integration tests.
  - Example: unit test for Arabic cleaner output; integration test for full OCR flow.
- `src/api/`: FastAPI transport layer.
  - Example: `routes.py` receives an image request and forwards it to controller logic.
- `src/core/`: shared config and external service clients.
  - Example: `config.py` loads environment variables; `services/ollama_client.py` wraps Ollama calls.
- `src/data/dataclasses/`: OCR domain models.
  - Example: a `Bill` object containing merchant info and `LineItem` records.
- `src/data/repository/`: persistence contracts.
  - Example: save intermediate text boxes and final normalized OCR output.
- `src/data/pipelines/`: assembly-line execution of OCR stages.
  - Example: `preprocessing/image_alignment_step.py` -> `processing/text_detection_step.py` -> `postprocessing/llm_correction_step.py`.
- `src/strategies/`: pluggable algorithms behind shared interfaces.
  - Example: swap `yolo_detection_strategy.py` with `contour_detection_strategy.py` without changing pipeline code.
- `src/Engine/controllers/`: request orchestration layer.
  - Example: `ocr_controller.py` coordinates a full run from input image to final response.
- `src/Engine/Managers/`: lifecycle and decision managers.
  - Example: `model_manager.py` loads heavy models once; manager logic chooses the best solver based on criteria (quality, speed, hardware).
- `src/Solvers/`: problem-specific execution logic used by pipelines.
  - Example: manager selects `bill_ocr_solver.py`; selected solver is then used inside pipeline steps for the current request.
- `src/utils/`: reusable helper functions.
  - Example: Arabic normalization helpers and image utility functions shared across modules.

## Responsibilities by Layer

- `api`: transport and validation layer (HTTP contracts).
- `core`: shared runtime config + external clients.
- `data`: domain models, repositories, and pipeline stage composition.
- `strategies`: pluggable algorithm families behind stable interfaces.
- `Engine`: lifecycle and orchestration (model and pipeline managers, controllers).
- `Solvers`: solver implementations used by pipelines after `Managers` choose the best solver for the request context.
- `utils`: reusable utility helpers.
- `scripts`: operational scripts for data, training, evaluation, and local runs.
- `tests`: behavior checks for both unit and integration coverage.





