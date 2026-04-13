---
name: dlstreamer-coding-agent
description: "Build new DL Streamer video-analytics applications (Python or gst-launch command line). Use when: user describes a vision AI pipeline, wants to create a new sample app, combine elements from existing samples, add detection/classification/VLM/tracking/alerts/recording to a video pipeline, or create custom GStreamer elements in Python. Gathers pipeline requirements interactively — input sources, AI models (URLs or names), target Intel hardware, expected outputs — then generates working DL Streamer code using established design patterns."
argument-hint: "Describe the vision AI pipeline you want to build (e.g. 'detect faces in RTSP stream and save alerts as JSON')"
---

# DL Streamer Coding Agent

Build new DL Streamer video-analytics applications (Python or gst-launch command line) by composing design patterns extracted from existing sample apps.

NOTE: This feature is in PREVIEW stage — expect some rough edges and missing features, and please share your feedback to help us improve it!

## When to Use

- User describes a vision AI processing pipeline in natural language
- User wants to create a new Python sample application built on DL Streamer
- User wants to create a new GStreamer command line using DL Streamer elements
- User wants to combine elements from multiple existing samples (e.g. detection + VLM + recording)
- User needs to add custom analytics logic or custom GStreamer elements in Python

See [example prompts](./examples) for inspiration.

## Directory Layout for a New Sample App

```
<new_sample_app_name>
├── <app_name>.py or .sh        # Main application (Python or shell script)
├── export_models.py or .sh     # Model download and export script
├── requirements.txt            # Python dependencies for the application
├── export_requirements.txt     # Python dependencies for model export scripts
├── README.md                   # Documentation with instructions on how to install prerequisites and run the sample
├── plugins/                    # Only if custom GStreamer elements are needed
│   └── python/
│       └── <element>.py
├── config/                     # Only if config files are needed
│   └── *.txt / *.json
├── models/                     # Created at runtime (cached model exports)
├── videos/                     # Created at runtime (cached video downloads)
└── results/                    # Created at runtime (output files)
```

## Procedure

### Step 0 — Fast Path (Pattern Table Match)

Before proceeding with the full procedure, check if the user's prompt maps directly to
a row in the [Common Pipeline Patterns table](./references/pipeline-construction.md#common-pipeline-patterns).
If a match is found **and** the prompt is unambiguous (input source, model type, and
expected output are all clear or can be confidently inferred):

1. Skip Step 1 (prompt refinement)
2. Read **only** the specific Design Patterns listed in the matching row (not all references)
3. Proceed directly to Steps 2–6 using the listed templates and patterns

This fast path avoids unnecessary clarification questions and reduces context loading
for well-defined use cases.

### Step 1 — Refine User Prompt

The user's prompt may be ambiguous or incomplete. Before proceeding further, make sure the following details are clarified:
1) Input source (video file vs RTSP stream, single vs multi-camera, etc.); ask for a specific video file if possible
2) AI model types (detection, classification, OCR, VLM, etc.) and specific models if possible (e.g. "YOLOv8 for detection and PaddleOCRv5 for OCR")
If the user does not have specific models in mind, try to infer the most likely model choice based on the task description and the list of models supported by DLStreamer (`../../../../docs/user-guide/supported_models.md`).
3) Sequence of operations in the pipeline (e.g. detection → tracking → classification, or detection + VLM in parallel branches, etc.)
4) Expected output (e.g. JSON file with license plate text, annotated video file, etc.)
5) Performance requirements (e.g. real-time processing, batch processing, etc.)

### Step 2 — Identify Models and Start Environment Setup (early, async)

> **Parallelization rule:** Steps 2, 3, and 4 overlap. The venv creation and `pip install`
> from Step 2 are **network-bound** and take minutes but require **no reasoning**. Start
> them in an **async terminal** immediately after creating the requirements file, then
> continue with Steps 3 and 4 (Docker check + pipeline design) while the install runs
> in the background. Come back to run the actual model export in Step 2b only after
> `pip install` has finished.

**2a — Create export scripts and kick off venv + pip install**

Check which AI models the user wants to use. Search whether the requested or similar models appear in the list of models supported by DLStreamer.

| Model exporter | Typical Models  | Path |
|--------|-------------|------|
| download_public_models.sh | Traditional computer vision models | `samples/download_public_models.sh` |
| download_hf_models.py | HuggingFace models, including VLM models and Transformer-based detection/classification models (RTDETR, CLIP, ViT) | `scripts/download_models/download_hf_models.py` |
| download_ultralytics_models.py | Specialized model downloader for Ultralytics YOLO models | `scripts/download_models/download_ultralytics_models.py` |

If a model is found in one of the above scripts, extract the model download recipe from that script and create a local script in the application directory for exporting the specific model to OV IR format; add model export instructions to the application README.
If a model does not exist, check the [Model Preparation Reference](./references/model-preparation.md) for instructions on how to prepare and export the model for DLStreamer, then write a new model download/export script using the [Export Models Template](./assets/export-models-template.py) as a starting point and add instructions to the application README.

Create the `export_requirements.txt` file if the model export script requires additional Python packages (e.g. HuggingFace transformers, Ultralytics, optimum-cli, etc.). Add comments in `export_requirements.txt` to indicate which model export script requires a specific package. Use **exact pinned versions** from the [Model Preparation Reference → Requirements](./references/model-preparation.md#requirements).

**As soon as** `export_requirements.txt` and `export_models.py` are written, start the
virtual-environment creation and dependency installation in an **async terminal** so it
runs in the background while you continue reasoning:

```bash
# Run in async mode — do NOT wait for completion
python3 -m venv .<app_name>-export-venv && \
source .<app_name>-export-venv/bin/activate && \
pip install -r export_requirements.txt
```

> **Important:** When running terminal commands that may take a long time (e.g. `pip install`,
> model downloads, model export), do **not** pipe output through `tail`, `head`, or other
> filters that hide progress. Let the full output stream to the terminal so the user can
> see download/install progress and is not left waiting with no feedback.

Now **proceed immediately** to Steps 3 and 4 while `pip install` runs.

**2b — Run model export (after pip install completes)**

After Steps 3 and 4 are done (or earlier, if `pip install` finished), check the async
terminal output to confirm all dependencies were installed successfully, then run the
model export:

```bash
source .<app_name>-export-venv/bin/activate
python3 export_models.py  # or bash export_models.sh
```

### Step 3 — Check and Setup Deployment Environment

Check if the user's machine has DLStreamer installed:
```bash
gst-inspect-1.0 gvadetect 2>&1 | grep Version
```

The command should return plugin details. If it does, check if the plugin version matches the latest official release of DLStreamer.

If the plugin is not found, or the version is older than the latest release, download the latest weekly DLStreamer docker image.

**Discovering the latest Docker tag:**
```bash
# Check already-pulled images:
docker images | grep dlstreamer

# If no local image exists, browse available tags at:
# https://hub.docker.com/r/intel/dlstreamer/tags?name=weekly-ubuntu24
# Then pull a specific tag, e.g.:
docker pull intel/dlstreamer:2026.1.0-20260407-weekly-ubuntu24
```

***Important*** — While the DLStreamer Coding Agent is still in preview, ALWAYS download the latest weekly build even if the user has the latest official version of DLStreamer installed, as the latest weekly build may contain important bug fixes and improvements that are not yet in the official release.

Recommended workflow: develop the application locally on your host machine and prepare/export models using a Python virtual environment. Once models are exported to OpenVINO IR format, run the application inside the DLStreamer container with your local directory mounted. This approach maintains development flexibility while leveraging the container for consistent runtime execution.

### Step 4 — Define DLStreamer Pipeline from User Description

Generate a DLStreamer pipeline string that captures the user's intent using DLStreamer elements. Use the [Pipeline Construction Reference](./references/pipeline-construction.md) to identify which elements to use for each part of the pipeline (e.g. source, decode, inference, metadata handling, sink).

For common use cases, go straight to file generation using the [use-case → template/pattern mapping table](./references/pipeline-construction.md#common-pipeline-patterns) in the Pipeline Construction Reference.

For complex cases, search the existing repository of sample applications for guidance.

If the user wants to add custom application logic, always check if this logic can be implemented using existing GStreamer elements or their combination. If it cannot, add a custom Python element to the pipeline and implement the logic there. Follow the [Custom Python Element Conventions](./references/coding-conventions.md#custom-python-element-conventions) for implementation details.

#### Reference Python Samples

Before generating code, read the relevant existing samples to understand established conventions:

| Sample | Key Pattern | Path |
|--------|-------------|------|
| hello_dlstreamer | Minimal pipeline + pad probe | `samples/gstreamer/python/hello_dlstreamer/` |
| face_detection_and_classification | Detect → classify chain, HuggingFace model export | `samples/gstreamer/python/face_detection_and_classification/` |
| prompted_detection | Third-party model integration (YOLOE), appsink callback | `samples/gstreamer/python/prompted_detection/` |
| open_close_valve | Dynamic pipeline control, tee + valve, OOP controller | `samples/gstreamer/python/open_close_valve/` |
| vlm_alerts | VLM inference (gvagenai), argparse config, file output | `samples/gstreamer/python/vlm_alerts/` |
| vlm_self_checkout | Computer Vision detection and VLM classification, multi-branch tee, custom frame selection for VLM | `samples/gstreamer/python/vlm_self_checkout/` |
| smart_nvr | Custom Python GStreamer elements (analytics + recorder), chunked storage | `samples/gstreamer/python/smart_nvr/` |
| onvif_cameras_discovery | Multi-camera RTSP, ONVIF discovery, subprocess orchestration | `samples/gstreamer/python/onvif_cameras_discovery/` |
| draw_face_attributes | Detect → multi-classify chain, custom tensor post-processing in pad probe callback | `samples/gstreamer/python/draw_face_attributes/` |
| coexistence | DL Streamer + DeepStream coexistence, Docker orchestration, multi-framework LPR | `samples/gstreamer/python/coexistence/` |

#### Reference Command Line Samples

Before generating code, read the relevant existing samples to understand established conventions:

| Sample | Key Pattern | Path |
|--------|-------------|------|
| face_detection_and_classification | Detection + classification chain (`gvadetect` → `gvaclassify`) | `samples/gstreamer/gst_launch/face_detection_and_classification/` |
| audio_detect | Audio event detection + metadata publish | `samples/gstreamer/gst_launch/audio_detect/` |
| audio_transcribe | Audio transcription with `gvaaudiotranscribe` | `samples/gstreamer/gst_launch/audio_transcribe/` |
| vehicle_pedestrian_tracking | Detection + tracking (`gvatrack`) | `samples/gstreamer/gst_launch/vehicle_pedestrian_tracking/` |
| human_pose_estimation | Full-frame pose estimation/classification | `samples/gstreamer/gst_launch/human_pose_estimation/` |
| metapublish | Metadata conversion and publish (`gvametaconvert`/`gvametapublish`) | `samples/gstreamer/gst_launch/metapublish/` |
| gvapython/face_detection_and_classification | Python post-processing via `gvapython` | `samples/gstreamer/gst_launch/gvapython/face_detection_and_classification/` |
| gvapython/save_frames_with_ROI_only | Save ROI frames with `gvapython` | `samples/gstreamer/gst_launch/gvapython/save_frames_with_ROI_only/` |
| action_recognition | Action recognition pipeline | `samples/gstreamer/gst_launch/action_recognition/` |
| instance_segmentation | Instance segmentation pipeline | `samples/gstreamer/gst_launch/instance_segmentation/` |
| detection_with_yolo | YOLO-based detection/classification | `samples/gstreamer/gst_launch/detection_with_yolo/` |
| geti_deployment | Intel® Geti™ model deployment | `samples/gstreamer/gst_launch/geti_deployment/` |
| multi_stream | Multi-camera / multi-stream processing | `samples/gstreamer/gst_launch/multi_stream/` |
| gvaattachroi | Attach custom ROIs before inference | `samples/gstreamer/gst_launch/gvaattachroi/` |
| gvafpsthrottle | FPS throttling with `gvafpsthrottle` | `samples/gstreamer/gst_launch/gvafpsthrottle/` |
| lvm | Image embeddings generation with ViT/CLIP | `samples/gstreamer/gst_launch/lvm/` |
| license_plate_recognition | License plate recognition (detector + OCR) | `samples/gstreamer/gst_launch/license_plate_recognition/` |
| gvagenai | VLM usage with `gvagenai` | `samples/gstreamer/gst_launch/gvagenai/` |
| g3dradarprocess | Radar signal processing | `samples/gstreamer/gst_launch/g3dradarprocess/` |
| g3dlidarparse | LiDAR parsing pipeline | `samples/gstreamer/gst_launch/g3dlidarparse/` |
| gvarealsense | RealSense camera capture | `samples/gstreamer/gst_launch/gvarealsense/` |
| custom_postproc/detect | Custom detection post-processing library | `samples/gstreamer/gst_launch/custom_postproc/detect/` |
| custom_postproc/classify | Custom classification post-processing library | `samples/gstreamer/gst_launch/custom_postproc/classify/` |
| face_detection_and_classification_bins | Detection + classification using `processbin`, GPU/CPU VA memory paths | `samples/gstreamer/gst_launch/face_detection_and_classification_bins/` |
| motion_detect | Motion region detection (`gvamotiondetect`), ROI-restricted inference | `samples/gstreamer/gst_launch/motion_detect/` |

### Step 0 — Gather Requirements Interactively

Before generating any code, use the `vscode_askQuestions` tool to collect pipeline requirements
from the user. This ensures the generated application matches their exact needs.

Pre-fill options based on the user's initial prompt where possible, but always present
the questions for explicit confirmation. **Always present all questions, even if the
user's prompt implies answers** — the user must explicitly confirm or adjust.

Use a **single** `vscode_askQuestions` call with all of the following questions:

#### Section 1 — Input

| Question Header | Question | Options (if applicable) |
|----------------|----------|------------------------|
| `Input Type` | What type of video input will you use? | `Local file path`, `HTTP URL`, `RTSP stream URI` |
| `Input Value` | Provide the video input path or URL (e.g. `/path/to/video.mp4`, `https://...`, `rtsp://...`) | Free text |

#### Section 2 — AI Models

Pre-fill the `Model URLs` field with recommendations from the known-good models table
when the use case is clear, but still ask for confirmation.

| Question Header | Question | Options (if applicable) |
|----------------|----------|------------------------|
| `Model 1` | Provide the first model URL or name and its role (e.g. `yolo11n — vehicle detection`). Pre-filled with a recommendation when the use case is clear. | Free text (pre-fill from known-good models table when use case is clear) |
| `Model 2 (optional)` | If you need a second model, provide its URL or name and role (e.g. `paddlepaddle/PaddleOCR-rec-en — plate text OCR`). Leave empty if not needed. | Free text (pre-fill if use case needs multiple models, e.g. detection + OCR) |

> Add more `Model N` questions only if the use case clearly requires 3+ models.
> The agent infers the source ecosystem (HuggingFace / Ultralytics / direct URL) and
> the pipeline task (detection / classification / OCR / VLM / segmentation) automatically
> from the model URL or name — no need to ask the user separately.

> **Model URL handling:** When the user provides URLs, determine the source ecosystem:
> - `huggingface.co/<org>/<model>` → Extract repo ID, use `optimum-cli` or `huggingface_hub` download
> - Ultralytics model names (e.g. `yolo11n`, `yoloe-26s-seg`) → Use `ultralytics` Python API
> - Direct `.onnx` / `.pt` / `.tflite` URLs → Use the universal conversion path in [Model Preparation Reference](./references/model-preparation.md) § 8
> - If the user says "recommend a model", suggest from the known-good models list below

**Known-good models for common tasks:**

| Task | Recommended Model | Source |
|------|-------------------|--------|
| General object detection | `yolo11n` or `yolo11s` | Ultralytics |
| Face detection | `arnabdhar/YOLOv8-Face-Detection` | HuggingFace (Ultralytics) |
| Person/vehicle detection | `yolo11n` | Ultralytics |
| Image classification | `dima806/fairface_age_image_detection` | HuggingFace (optimum-cli) |
| OCR / text recognition | `paddlepaddle/PaddleOCR-rec-en` | HuggingFace (PaddlePaddle) |
| VLM scene description | `OpenGVLab/InternVL3_5-2B` | HuggingFace (optimum-cli) |
| VLM alerting (small) | `HuggingFaceTB/SmolVLM2-2.2B-Instruct` | HuggingFace (optimum-cli) |
| Open-vocabulary detection | `yoloe-26s-seg` | Ultralytics |
| License plate detection | `yolo11n` + PaddleOCR | Ultralytics + HuggingFace |

#### Section 3 — Target Hardware & Optimization

| Question Header | Question | Options (if applicable) |
|----------------|----------|------------------------|
| `Intel Platform` | What Intel hardware will this run on? | `Intel Core Ultra (Panther Lake) — CPU + Xe3 GPU + NPU`, `Intel Core Ultra (Lunar Lake / Arrow Lake) — CPU + GPU + NPU`, `Intel Core Ultra (Meteor Lake) — CPU + GPU + NPU`, `Intel Xeon (server) — CPU only`, `Intel Arc discrete GPU`, `Intel Core (older, no NPU) — CPU + GPU`, `Not sure / detect at runtime` |
| `Available Accelerators` | Which accelerators are available? | `GPU (/dev/dri/renderD128)`, `NPU (/dev/accel/accel0)`, `CPU only` (multiSelect) |
| `Optimization Priority` | What matters most? | `Maximum throughput (FPS)`, `Lowest latency per frame`, `Power efficiency (NPU preferred)`, `Balanced (default)` |

**Canonical Intel Platform Reference** (single source of truth — all other files derive from this table):

| Questionnaire Option | Marketing Name | Code Name | Accelerators | Recommended Device Args | Recommended Batch Size |
|----------------------|----------------|-----------|--------------|------------------------|------------------------|
| Intel Core Ultra (Panther Lake) — CPU + Xe3 GPU + NPU | Intel® Core™ Ultra Series 3 | Panther Lake | CPU, GPU (Xe3/Battlemage), NPU | `--device GPU` (default) | GPU: 4–8, NPU: 1–2 |
| Intel Core Ultra (Lunar Lake / Arrow Lake) — CPU + GPU + NPU | Intel® Core™ Ultra Series 2 | Lunar Lake / Arrow Lake | CPU, GPU (Arc), NPU | `--device GPU` or `--device NPU` for classification | GPU: 4–8, NPU: 1–2 |
| Intel Core Ultra (Meteor Lake) — CPU + GPU + NPU | Intel® Core™ Ultra Series 1 | Meteor Lake | CPU, GPU (Arc), NPU | `--device GPU` (default) | GPU: 4–8, NPU: 1–2 |
| Intel Xeon (server) — CPU only | Intel® Xeon® 6 (Granite Rapids) | Granite Rapids | CPU, AMX | `--device CPU --batch-size 8` | CPU: 4–8 |
| Intel Arc discrete GPU | Intel® Arc™ A-Series | — | CPU, Discrete GPU | `--device GPU --batch-size 8` | GPU: 4–8 |
| Intel Core (older, no NPU) — CPU + GPU | Intel® Core™ (12th–14th Gen) | Alder Lake / Raptor Lake | CPU, GPU (Iris Xe) | `--device GPU` (default) | GPU: 4, CPU: 1–4 |
| Not sure / detect at runtime | (auto-detect) | — | (detected) | Auto-detect (see runtime detection in hardware-optimization.md) | (per device defaults) |

Use this table to populate `{{HARDWARE_TABLE}}` in the generated README and to select
device/batch-size settings in the pipeline. Only include rows matching the user's selection.

> **After gathering answers**, read the [Hardware Optimization Reference](./references/hardware-optimization.md)
> to map the user's platform and priorities to specific `device=` settings, `batch-size` values,
> and weight format choices for model export.

#### Section 4 — Output

| Question Header | Question | Options (if applicable) |
|----------------|----------|------------------------|
| `Output Format` | What outputs do you need? | `Annotated video (.mp4)`, `JSON metadata (.jsonl)`, `JPEG snapshots`, `Display window`, `All of the above` (multiSelect) |

If the user's initial prompt already implies answers (e.g. "detect faces using YOLOv8 on
RTSP camera with GPU"), pre-select the matching options as `recommended` but still present
all questions so the user can confirm or adjust.

#### Section 5 — Application Type

| Question Header | Question | Options (if applicable) |
| `Application Type` | Python application or gst-launch command line? | `Python application` (recommended), `gst-launch command line` |

#### Section 6 — Docker Image

| Question Header | Question | Options (if applicable) |
|----------------|----------|------------------------|
| `Docker Image` | Which DL Streamer Docker image to use? | See table below (multiSelect=false, allowFreeformInput=true) |

Present these options grouped by category. The agent should use the selected image
in all `docker pull` and `docker run` commands in the generated README and when
running the application for verification.

**Fetching available images at runtime:**

Before presenting the Docker Image question, fetch the latest tags from Docker Hub
by running this command in a terminal:

```bash
curl -s "https://hub.docker.com/v2/repositories/intel/dlstreamer/tags/?page_size=30&ordering=last_updated" \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
release, weekly = [], []
for t in data.get('results', []):
    name = t['name']
    if name in ('latest',) or 'sources' in name or 'dev' in name or 'rc' in name:
        continue
    if 'weekly' in name:
        weekly.append(name)
    else:
        release.append(name)
print('RELEASE:', ' '.join(release[:6]))
print('WEEKLY:', ' '.join(weekly[:4]))
"
```

Use the output to populate the `Docker Image` question options dynamically:

1. Always include `intel/dlstreamer:latest` as the first option (recommended)
2. Add the top 3 **release** tags from the `RELEASE` line (e.g. `2026.0.0-ubuntu24`, `2026.0.0-ubuntu22`, `2025.2.0-ubuntu24`)
3. Add the top 2 **weekly** tags from the `WEEKLY` line (e.g. `2026.1.0-20260407-weekly-ubuntu24`, `2026.1.0-20260407-weekly-ubuntu22`)

Format each option label as `intel/dlstreamer:<tag>` with a short description derived
from the tag (Ubuntu version, release/weekly, date).

> The user can also type a custom image tag (e.g. a locally built image or a
> specific RC tag like `intel/dlstreamer:2026.0.0-ubuntu24-rc3`).
> Full list of available tags: https://hub.docker.com/r/intel/dlstreamer/tags


### Quick Recipes

For common use cases, skip reading reference samples and go straight to file generation
using these recipes:

| Use Case | Templates | Design Patterns | Key Model Export |
|----------|-----------|-----------------|------------------|
| Detection + save video + JSON | `python-app-template.py` | 1 + 4 + 11 | Ultralytics |
| Detection + classification/OCR + save | `python-app-template.py` + `export-models-template.py` | 1 + 4 + 11 + 13 | YOLO + PaddleOCR/optimum-cli |
| VLM alerting + save | `python-app-template.py` | 1 + 9 + 11 | optimum-cli |
| Detection + custom analytics | `python-app-template.py` | 1 + 4 + 6 + 11 | Ultralytics |
| Detection + tracking + recording | `python-app-template.py` | 1 + 4 + 5 + 7 | Ultralytics |
| Multi-camera RTSP | `python-app-template.py` | 1 + 12 | (per camera) |

For use cases matching a recipe above, generate all files directly from the templates
and model-preparation reference without reading existing samples. Only read reference
samples when the use case doesn't match any recipe or requires unusual element combinations.

### Step 1 — Create Model Download and Export scripts

Based on the model information gathered in Step 0, determine the export path for each model.

**If the user provided model URLs or IDs**, classify each by source ecosystem:

| URL / ID Pattern | Ecosystem | Export Method |
|-----------------|-----------|---------------|
| `huggingface.co/<org>/<model>` or bare HF ID | HuggingFace | `optimum-cli export openvino` or `huggingface_hub` + Ultralytics |
| Ultralytics name (`yolo11n`, `yoloe-*`) | Ultralytics | `model.export(format="openvino")` |
| Direct URL ending in `.onnx` | ONNX | `ovc` directly |
| Direct URL ending in `.pt` / `.pth` | PyTorch | `torch.onnx.export` → `ovc` |
| Direct URL ending in `.tflite` | TensorFlow Lite | `ovc` directly |
| Direct URL to SavedModel `.tar.gz` / dir | TensorFlow | `ovc` directly |

For non-native models (ONNX, PyTorch, TF, etc.), follow the universal conversion path in
[Model Preparation Reference](./references/model-preparation.md) § 8.

**If the user did not provide URLs**, search if the requested or similar models are in the list of models supported by DL Streamer:

| Model exporter | Typical Models  | Path |
|--------|-------------|------|
| download_public_models.sh | Traditional computer vision models | `samples/download_public_models.sh` |
| download_hf_models.py | HuggingFace models, including VLM models and Transformer-based detection/classification models (RTDETR, CLIP, ViT) | `scripts/download_models/download_hf_models.py` |
| download_ultralytics_models.py | Specialized model downloader for Ultralytics YOLO models | `scripts/download_models/download_ultralytics_models.py` |

If a model is found in one of the above scripts, extract model download recipe from that script and create a local script in application directory for exporting the specific model to OV IR format; add model export instructions to the application README.
If a model does not exist, check the [Model Preparation Reference](./references/model-preparation.md) for instructions on how to prepare and export the model for DL Streamer, then write a new model download/export script using the [Export Models Template](./assets/export-models-template.py) as a starting point and add instructions to the application README.

Create the `export_requirements.txt` file if the model export script requires additional Python packages (e.g. HuggingFace transformers, Ultralytics, optimum-cli, etc.). Add comments in `export_requirements.txt` to indicate which model export script requires a specific package. Use specific version numbers for packages to ensure reproducibility.

Run the model export script to verify that the models can be downloaded and exported correctly to OpenVINO IR format. 
Create and set up a Python virtual environment to isolate dependencies:

```bash
python3 -m venv .<app_name>-export-venv
source .<app_name>-export-venv/bin/activate
pip install -r export_requirements.txt
python3 export_models.py  # or bash export_models.sh
```

### Step 2 — Define DL Streamer Pipeline from User Description

Generate a DL Streamer pipeline string that captures the user's intent using DL Streamer elements. Use the [Pipeline Construction Reference](./references/pipeline-construction.md) to identify which elements to use for each part of the pipeline (e.g. source, decode, inference, metadata handling, sink).

**Apply hardware-aware device assignment** based on the target platform gathered in Step 0:
- Read the [Hardware Optimization Reference](./references/hardware-optimization.md) for device mapping
- Set `device=` property on each inference element (`gvadetect`, `gvaclassify`, `gvagenai`) according to the user's available accelerators and optimization priority
- Set `batch-size=` based on the target device (GPU: 4–8, NPU: 1–2, CPU: 1–4)
- For multi-model pipelines, distribute inference across GPU and NPU to balance load
- Add runtime device detection fallback if the user selected "detect at runtime"

### Step 2a [Command Line Application] — Construct Command Line Pipeline for Simple Use Cases


If the user asks for a command-line application, construct a `gst-launch-1.0` pipeline string using the identified DL Streamer elements. Follow established conventions for element properties, caps negotiation, and metadata handling as seen in the reference command line samples.

### Step 2b [Python Application] — Construct Python Applications for Complex Use Cases and Custom Application Logic

If the user asks for a Python application or wants to add custom logic as new Python elements, decompose the requested pipeline into one or more of the design patterns listed in the [Design Patterns Reference](./references/design-patterns.md). This will guide the structure of the application, including how to construct the pipeline, where to add callbacks, and how to handle models and metadata.

Map the user's description to one or more patterns using the [Pattern Selection Table](./references/design-patterns.md#pattern-selection-table) in the Design Patterns Reference.

Read the [Coding Conventions Reference](./references/coding-conventions.md) before writing a Python application.
Use the [Application Template](./assets/python-app-template.py) as a starting skeleton.

Compose the application by:
1. Selecting the appropriate **pipeline construction** approach — see [Pipeline Construction Reference](./references/pipeline-construction.md)
2. Following the **Pipeline Design Rules** (Rules 1–8) in the Pipeline Construction Reference — prefer auto-negotiation, GPU/NPU inference, `gvaclassify` for OCR, `gvametapublish` for JSON, multi-device assignment on Intel Core Ultra, fragmented MP4 for robustness (Rule 7), audio track handling (Rule 8)
3. Assembling the **pipeline string** from DLStreamer elements listed in the Pipeline Construction Reference
4. Preparing models using the correct export method — see [Model Preparation Reference](./references/model-preparation.md)
5. Adding **callbacks/probes** as needed
6. Adding **custom Python elements** if the user needs inline analytics
7. Wiring up **argument parsing** and **asset resolution**
8. Adding the **pipeline event loop** — see [Pattern 12: Pipeline Event Loop](./references/design-patterns.md#pattern-12-pipeline-event-loop)

### Step 6 — Generate Sample Application

Generate the sample application following the directory structure outlined at the beginning of this document.
Use the [README Template](./assets/README-template.md) to generate the `README.md` file — replace `{{PLACEHOLDERS}}` with application-specific content and remove HTML comments.

If an application requires Python dependencies, list them in `requirements.txt` and then create and activate a local Python environment prior to running the application. If OpenVINO python runtime is required, please make sure it is added to `requirements.txt` with same version as OpenVINO runtime installed with DL Streamer.

```bash
source .<app_name>-venv/bin/activate
pip install -r requirements.txt
python3 <app_name>.py  # or bash <app_name>.sh
```

Recommended workflow: develop the application locally on your host machine and prepare/export models using a Python virtual environment. Once models are exported to OpenVINO IR format, run the application inside the DL Streamer container with your local directory mounted. This approach maintains development flexibility while leveraging the container for consistent runtime execution.

**Running in the DL Streamer container** — use the Docker image the user selected in
Step 0 Section 6 (`<DOCKER_IMAGE>`, default `intel/dlstreamer:latest`). Always pull first.
Mount `/dev/dri` (GPU) and `/dev/accel` (NPU, when present). The container lacks render/accel
group membership, so add them via `--group-add` + `stat`. Use `-u` to preserve file ownership.

```bash
docker pull <DOCKER_IMAGE>

docker run -it --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd)":/app -w /app \
    --device /dev/dri \
    --group-add $(stat -c "%g" /dev/dri/render* | head -n1) \
    --device /dev/accel \
    --group-add $(stat -c "%g" /dev/accel/accel* | head -n1) \
    -e ZE_ENABLE_ALT_DRIVERS=libze_intel_npu.so \
    <DOCKER_IMAGE> \
    python3 <app_name>.py
```

Omit the `--device /dev/accel`, its `--group-add` line, and `ZE_ENABLE_ALT_DRIVERS` when NPU is not available on the host.

Once the environment is set up, update instructions in generated README.md file and verify the application runs correctly when following instructions. If the user provided a natural language description of the expected output, verify that the output matches the description (e.g. check that JSONL files have the expected fields, check that video outputs have the expected overlays, etc.).

### Step 7 — Run, Debug and Validate Application

Once the environment is set up, update the instructions in the generated README.md file and verify that the application runs correctly when following them. If the user provided a natural-language description of the expected output, verify that the output matches the description (e.g. check that JSONL files have the expected fields, check that video outputs have the expected overlays, etc.).

If the application is running for a long time (>1 minute), make sure there is some output in the terminal to indicate progress and avoid leaving the user wondering if the application is stuck. Switch focus to the terminal output so the user can see logs and progress.
If the application has a continuous input stream (RTSP camera source) or large input video files, send an EOS signal to the application.

Refer to the [Debugging Hints](./references/debugging-hints.md) for Docker testing conventions, common gotchas, and the post-run [Validation Checklist](./references/debugging-hints.md#validation-checklist).

### Step 8 — Report Coding Agent Analytics

Report the time taken for the entire activity, from prompt to working application. Report the following metrics:
1) AI reasoning time (time spent on understanding the prompt, designing the pipeline, writing code, etc.).
2) Environment setup time (time spent waiting for `pip install`, model export, Docker image pull, etc.).
3) Debug and Validation time (time spent running the application, checking outputs, and fixing issues).
4) Time waiting for user action (time spent waiting for user input or confirmation).
5) Total activity time (please note some phases may overlap, so the total time is not necessarily the sum of individual phases).
This will help us understand how much of the process is automated vs how much requires human input and waiting time.

## Examples
See [example prompts](./examples) for inspiration on how to write effective prompts for DLStreamer Coding Agent, and to see how the above procedure can be applied in practice to generate new sample applications.

