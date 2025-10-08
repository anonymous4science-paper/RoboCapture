# RoboCapture

A toolkit and benchmark for generating, disambiguating, and decomposing ambiguous embodied instructions from real-world tabletop scenes. This repository contains:

- RoboAmbig-Bench: images and an accompanying spreadsheet used to align images with generated instructions and annotations.
- RoboCapture-code: the end-to-end pipeline for multimodal instruction generation and atomic action decomposition driven by an LLM, with a light-weight buffer (few-shot) mechanism.

## Repository Structure

- `RoboAmbig-Bench/`
  - `Images/`: scene images .
  - `merged_all.xlsx`: spreadsheet with image-to-instruction alignment (kept in sync with the new file names). 
- `RoboCapture-code/`
  - `combined.py`: main pipeline to generate ambiguous instructions, analyze operability, perform scene/intent analysis, and decompose into atomic actions. Includes a buffer mechanism using Excel and example images.
  - `buffer_examples_various_scenes/`: translated example workbooks for diverse scenes (e.g., `kitchen_sink_scene_buffer_examples.xlsx`).
- `Appendix 1.pdf`, `Appendix 2.pdf`, `Appendix 3.pdf`: supplementary materials.

## Key Features

- Multimodal prompting: builds messages with a main image plus optional few-shot example images.
- Ambiguous instruction generation (multiple ambiguity types), followed by automatic parsing.
- Atomic action sequence decomposition with basic structural validation.
- Optional buffer write-back: on success across all types, results are appended to an Excel buffer and the image is copied with a stable sequential name.

## Requirements

Python 3.9+ is recommended. Install dependencies:

```bash
pip install openai openpyxl pandas pillow
```

Notes:
- `openai`: used via a configurable API endpoint; the code currently references a custom `base_url` and expects an API key.
- `openpyxl`: Excel I/O and embedded image handling.
- `pandas`: tabular processing for the buffer spreadsheet.
- `pillow` (PIL): resizing/embedding images into Excel.

## Configuration

`RoboCapture-code/combined.py` exposes the following key parameters:

- API configuration
  - `API_SECRET_KEY`: your API key (RECOMMENDED: set via env var and read in code, not hard-coded).
  - `BASE_URL`: the API endpoint (defaults to `https://api.zhizengzeng.com/v1/`).
  - `LLM_PARAMS`: model name and generation settings.
- Buffer (few-shot) configuration
  - `BUFFER_EXCEL_PATH`: default `buffer_examples.xlsx`.
  - `BUFFER_IMAGE_DIR`: default `buffer_images/`.
  - `SAMPLES_FROM_BUFFER`: number of groups sampled per run.
  - `BUFFER_IMAGE_COLUMN_NAME`: image filename column header.

Recommended: export credentials and read them from environment variables in the script (e.g., `os.environ.get("API_SECRET_KEY")`). Remove or override any accidental secrets in the code before committing.

## Data Preparation

1) Place your scene images under `RoboAmbig-Bench/Images/` (any `.jpg/.png` supported by the pipeline).
2) Ensure `RoboAmbig-Bench/merged_all.xlsx` references the stable image names (`img_XXXX.ext`). If you rename images, update this file accordingly.
3) (Optional) Prepare buffer examples:
   - `RoboCapture-code/buffer_examples_various_scenes/*.xlsx` contain translated examples for different scenes.
   - Example images referenced by those workbooks should be saved under `RoboCapture-code/buffer_images/` with names like `buffer_image01.jpg`.

## Usage

1) Configure your API access in `combined.py` (or via environment variables).
2) Set the input image directory in `combined.py` (variable `image_directory`). By default it points to a Windows path in the sample; change it to your local path, e.g.:

```python
image_directory = "RoboAmbig-Bench/Images"
```

3) Run the pipeline:

```bash
cd RoboCapture-code
python combined.py
```

4) Outputs:
   - An Excel report under `RoboCapture-code/output/` named like `final_output_YYYYMMDD_HHMMSS.xlsx`, embedding the image and including the generated ambiguous instruction, clarification Q&A (if any), and atomic action sequence per ambiguity type.
   - If all ambiguity types succeed for an image and the probabilistic condition is met, results are appended to the buffer workbook and the image is copied into `buffer_images/` with a sequential name (`buffer_imageXX.ext`).

## Reproducibility and Naming Hygiene

- Image names in `RoboAmbig-Bench/Images` are sanitized to be time-independent (e.g., `img_0001.jpg`), with all Chinese/Weixin markers removed.
- `merged_all.xlsx` is kept in sync with the current image names; backups of earlier states (e.g., `*.bak.xlsx`) may exist for traceability.
- Maintenance utilities (used during cleanup) also wrote a `image_name_mapping.csv` in the image folder to trace old→new names.

## Safety & Ethics

- The pipeline enforces real-world assumptions and basic safety constraints in prompts (e.g., avoiding harmful actions, not treating scenes as toys/simulations).
- Always review model outputs before executing on physical systems.

## License

Unless otherwise noted in subfolders, this project is released for academic, non-commercial use. Please verify third-party assets (images, appendices) for their respective terms.

## Contact

For questions or collaboration inquiries, please open an issue in the GitHub repository.
