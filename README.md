# Handwritten Arabic Bill OCR Backend

Production-oriented backend skeleton for a university junior project that extracts text from handwritten Arabic bills.

Pipeline:

1. Image Alignment
2. Text Detection
3. Handwriting Recognition
4. LLM Correction

This repository is currently a skeleton only. Python files are placeholders with docstrings and no implementation logic yet.

---

## 1) Project Goals

- Build a modular OCR backend that is easy for a junior team to extend.
- Keep research work (notebooks) separated from production code (`app/` and `api/`).
- Make model backends swappable (OpenCV, YOLO OBB, TrOCR, Ollama can be replaced later).
- Keep data, model artifacts, and database responsibilities clearly separated.

---

## 2) High-Level Architecture

Flow of a request:

1. Client sends an image to API.
2. API calls the OCR pipeline in `app/pipeline.py`.
3. Pipeline runs stages in order:
   - align -> detect -> recognize -> correct
4. Pipeline returns structured OCR output.
5. Optional persistence:
   - row-based metadata/results in database (`repository/data_access`)
   - files/artifacts in storage (`repository/storage`)

---

## 3) Folder Structure (Simple + Practical)

```text
Haji_OCR/
├─ data/
│  ├─ raw/
│  ├─ clean/
│  └─ synthetic/
├─ notebooks/
│  ├─ 01_exploration/
│  ├─ 02_preprocessing/
│  ├─ 03_detection/
│  ├─ 04_recognition/
│  └─ 05_correction/
├─ app/
│  ├─ pipeline.py
│  ├─ core/
│  ├─ stages/
│  ├─ helpers/
│  ├─ schemas/
│  └─ services/
├─ api/
│  ├─ main.py
│  ├─ core/
│  ├─ routes/
│  └─ schemas/
├─ repository/
│  ├─ db/
│  ├─ data_access/
│  ├─ storage/
│  └─ migrations/
├─ models/
│  ├─ configs/
│  ├─ detection/
│  ├─ recognition/
│  ├─ correction/
│  └─ model_registry.yaml
├─ scripts/
├─ tests/
├─ docs/
├─ requirements.txt
├─ .gitignore
├─ Dockerfile
└─ .env.example
```

---

## 4) Detailed Explanation by Section

## `data/`

Dataset lifecycle folders:

- `raw/`: original untouched data.
- `clean/`: cleaned and standardized data ready for training/inference tests.
- `synthetic/`: generated data (augmentations or synthetic bills) used to improve robustness.

Use `synthetic/` for scenarios that are rare in real data (noise, blur, extreme rotation, unusual layouts).

## `notebooks/`

Experiment-only workspace. Numbering gives a clear workflow timeline:

- `01_exploration`: inspect dataset, understand quality and labels.
- `02_preprocessing`: test alignment and preprocessing ideas.
- `03_detection`: benchmark detection variants.
- `04_recognition`: compare recognition models/settings.
- `05_correction`: evaluate LLM correction prompts and strategies.

Rule: notebook code is exploratory; stable logic should be moved to `app/`.

## `app/` (Core OCR Logic)

Production backend logic (no API transport concerns here).

- `pipeline.py`: orchestrates all OCR stages in order.
- `core/`: shared config, common types, and error definitions.
- `stages/`: each pipeline step (align, detect, recognize, correct) and stage interfaces.
- `helpers/`: reusable helper functions (images, text, files, logging).
- `schemas/`: internal OCR/bill data shapes.
- `services/`: reusable operations like loading models or running OCR tasks.

This separation keeps logic clean and easy to test.

## `api/` (FastAPI Layer)

Service interface for external clients.

- `main.py`: FastAPI application entrypoint.
- `core/config.py`: API-level settings (host, ports, env, runtime flags).
- `core/lifespan.py`: startup/shutdown tasks (load resources on start, release on stop).
- `routes/`: endpoint groups (health, OCR).
- `schemas/`: request/response contracts for API I/O.

Simple view:

- `app/` = "do OCR work"
- `api/` = "expose OCR work over HTTP"

## `repository/` (Persistence Layer)

This is intentionally split into 4 parts so responsibilities do not mix:

### `repository/db/`

Database plumbing:

- connection setup
- session lifecycle
- ORM base metadata

No business queries here.

### `repository/data_access/`

Domain-level database operations:

- save/read bill records
- save/read OCR results
- save/read model version metadata

If it is a query/write on tables, it belongs here.

### `repository/storage/`

File/object storage operations:

- datasets
- saved images
- exported artifacts
- model files in external storage (if used)

If it is a file/blob path operation, it belongs here.

### `repository/migrations/`

Database schema history:

- create/alter tables over time
- track schema versions

Commonly managed with migration tools (for example, Alembic later).

Quick rule for team members:

- DB row/table action -> `data_access`
- File/object action -> `storage`
- Engine/session/base setup -> `db`
- Schema change history -> `migrations`

## `models/` (Model Assets + Runtime Config)

Holds model artifacts and model-level config, not pipeline logic.

- `detection/...`: detector weights/checkpoints (example: YOLO OBB versions).
- `recognition/...`: recognizer weights/assets (example: TrOCR versions).
- `correction/...`: correction prompt sets/templates for LLM post-processing.
- `configs/*.yaml`: task-specific runtime parameters (thresholds, decode options, etc.).
- `model_registry.yaml`: active model version mapping used by runtime.

Why this matters:

- easy model versioning
- reproducibility
- safe rollback
- clear experiment tracking

## `scripts/`

Task automation:

- data fetching/preparation
- synthetic data generation
- training
- evaluation

Keeps repetitive commands standardized for the team.

## `tests/`

Quality layer:

- `unit/`: small fast checks for isolated logic.
- `integration/`: larger checks for connected components (API + pipeline behavior).

## `docs/`

Project docs:

- architecture notes
- API notes
- design decisions for team onboarding and reporting

---

## 5) Suggested Team Workflow

1. Explore and test in `notebooks/`.
2. Move stable logic into `app/`.
3. Expose required functionality via `api/`.
4. Save/read metadata via `repository/data_access`.
5. Save/read files via `repository/storage`.
6. Version model artifacts in `models/`.
7. Add tests for every stable feature.

---

## 6) Naming and Design Principles Used

- Keep names short and clear (`app`, `stages`, `helpers`, `data_access`).
- Separate concerns (logic vs API vs persistence vs artifacts).
- Prefer versioned directories for model assets (`v1`, `v2`, ...).
- Keep this repository implementation-agnostic so backends can change.

---

## 7) Current Status

- Project structure scaffolded.
- Python files are placeholders with docstrings only.
- No implementation logic yet.

Next step is to add minimal interfaces and contracts for each stage without locking into any one model backend too early.

