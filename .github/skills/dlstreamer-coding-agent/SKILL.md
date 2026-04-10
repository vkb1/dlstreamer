---
name: dlstreamer-coding-agent
description: "Build new DL Streamer Python video-analytics applications. Use when: user describes a vision AI pipeline, wants to create a new sample app, combine elements from existing samples, add detection/classification/VLM/tracking/alerts/recording to a video pipeline, or create custom GStreamer elements in Python. Translates natural-language pipeline descriptions into working DL Streamer Python code using established design patterns."
argument-hint: "Describe the vision AI pipeline you want to build (e.g. 'detect faces in RTSP stream and save alerts as JSON')"
---

# DL Streamer Coding Agent

Build new DL Streamer Python video-analytics applications by composing design patterns extracted from existing sample apps.

## When to Use

- User describes a vision AI processing pipeline in natural language
- User wants to create a new Python sample application built on DL Streamer
- User wants to create a new GStreamer command line using DL Streamer elements
- User wants to combine elements from multiple existing samples (e.g. detection + VLM + recording)
- User needs to add custom analytics logic or custom GStreamer elements in Python

##  Directory Layout for a New Sample App

```
<new_sample_app_name>
├── <app_name>.py or .sh        # Main application (Python or shell script)
├── <download_models.py or .sh  # Model download script (if not embedded in main application)
├── README.md                   # Documentation with instructions how to install prerequisites and run the sample
├── requirements.txt            # Python dependencies (if any, including PyGObject)
├── plugins/                    # Only if custom GStreamer elements are needed
│   └── python/
│       └── <element>.py
├── config/                     # Only if config files are needed
│   └── *.txt / *.json
├── models/                     # Created at runtime (cached model exports)
├── videos/                     # Created at runtime (cached video downloads)
└── results/                    # Created at runtime (output files)
```

## Reference Python Samples

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

## Reference Command Line Samples

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

## Procedure

### Step 0 — Gather Pipeline Parameters

Before generating any code, collect the required parameters from the user. If the user's initial message does not provide clear answers to **all required parameters**, use the **ask-questions tool** to present an interactive questionnaire. Skip any parameter the user already specified.

#### Required Parameters

| Parameter | Question | Options / Format |
|-----------|----------|-----------------|
| **Input source** | What is your input video source? | Options: `Local file (.mp4, .avi, etc.)`, `RTSP stream URL`, `USB camera (/dev/video*)`, `Test pattern (videotestsrc)` — also accept freeform path/URL |
| **AI models** | Which AI model(s) should the pipeline use? See **Model Collection** below. | Options: `Face detection (face-detection-adas-0001)`, `Person/vehicle detection (person-vehicle-bike-detection-2004)`, `YOLO (YOLOv8/v11)`, `VLM (LLaVA / Qwen2-VL / Phi-3-Vision)`, `Provide model URL(s)` — multi-select allowed |
| **Application type** | Do you want a Python application or a gst-launch command line? | Options: `Python application` (recommended), `gst-launch-1.0 command line` |
| **Expected output** | What output do you expect from the pipeline? | Options: `Display window with overlays (autovideosink)`, `JSONL metadata file`, `Annotated video file (.mp4)`, `RTSP re-stream`, `Console/stdout`, `Custom callback / appsink` — multi-select allowed |
| **DL Streamer image** | Which DL Streamer Docker image do you want to use for testing? | Options: `intel/dlstreamer:2026.0.0-ubuntu24` (recommended, Ubuntu 24.04), `intel/dlstreamer:2026.0.0-ubuntu22` (Ubuntu 22.04), `intel/dlstreamer:latest` (alias for latest Ubuntu 24), `Local / native install (no Docker)` — also accept freeform image tag |
| **Run application** | Do you want me to run and test the application after generating the code? | Options: `Yes — run and verify output` (recommended), `No — generate code only` |

#### Model Collection (when user selects "Provide model URL(s)" or provides URLs directly)

If the user chooses to provide model URLs or pastes URLs in their initial message, use the **ask-questions tool** to collect the following details **for each model**. Present all models in a single questionnaire.

| Parameter | Question | Options / Format |
|-----------|----------|-----------------|
| **Model URL** | Provide the model URL or identifier | Freeform — accept any of: HuggingFace URL (`https://huggingface.co/...`), HuggingFace model ID (`org/model-name`), Ultralytics model name (`yolov8n`, `yolo11s`), Open Model Zoo model name (`face-detection-adas-0001`) |
| **Pipeline role** | What should this model do in the pipeline? | Options: `Object detection (gvadetect)`, `Classification / attributes (gvaclassify)`, `OCR / text recognition (gvaclassify)`, `VLM / generative AI (gvagenai)`, `Full-frame inference` |
| **Target objects** | What objects or classes should this model detect/classify? | Freeform — e.g. `faces`, `persons, vehicles, bikes`, `license plates` |

**URL resolution rules:**
- **HuggingFace URL/ID** (e.g. `https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct`, `Intel/face-detection-adas-0001`) → use `download_hf_models.py` or `optimum-cli` export
- **Ultralytics model** (e.g. `yolov8n`, `yolo11s-seg`, `https://huggingface.co/Ultralytics/...`) → use `download_ultralytics_models.py`
- **Open Model Zoo name** (e.g. `face-detection-adas-0001`, `person-vehicle-bike-detection-2004`) → use `download_public_models.sh` or `omz_downloader`
- **Direct URL to ONNX/IR** (e.g. `https://.../*.onnx`, `https://.../*.xml`) → download directly with `wget`/`curl`
- If the source is ambiguous, ask the user to clarify

**Multiple models:** Users can provide multiple URLs to build multi-stage pipelines (e.g. a detection model followed by a classification model). Ask the pipeline role for **each** model individually so the agent knows which DL Streamer element to use (`gvadetect`, `gvaclassify`, `gvagenai`) and in what order they chain.

#### Hardware & Performance Profiling

Always ask about the target hardware — even when the user doesn't mention it — because it directly affects device selection, preprocessing backend, batching, and throughput tuning. Use the **ask-questions tool** with the following questions (include them in the same questionnaire as the required parameters above):

| Parameter | Question | Options / Format |
|-----------|----------|------------------|
| **Intel platform** | What Intel hardware platform are you running on? | Options: `Intel Core Ultra (Panther Lake)`, `Intel Core Ultra (Meteor Lake / Lunar Lake / Arrow Lake)`, `Intel Core (12th-14th Gen — Alder Lake / Raptor Lake)`, `Intel Xeon (Sapphire Rapids / Emerald Rapids / Granite Rapids)`, `Intel Arc GPU (A770/A750/A580)`, `Intel Data Center GPU Flex (140/170)`, `Embedded / IoT (Atom / N-series)`, `Not sure — let me run detection` |
| **Available devices** | Which accelerators are available on the system? | Options: `Integrated GPU (Intel iGPU)`, `Discrete GPU (Intel Arc / Flex)`, `NPU (Neural Processing Unit)`, `CPU only`, `Multiple GPUs (GPU.0, GPU.1, ...)` — multi-select allowed |
| **Stream count** | How many video streams will be processed simultaneously? | Options: `1 (single stream)`, `2-4 streams`, `5-16 streams`, `16+ streams` — also accept freeform number |
| **Performance goal** | What is your priority? | Options: `Maximum throughput (highest FPS across all streams)`, `Lowest latency (fastest per-frame response)`, `Balanced (good throughput with acceptable latency)` |

If the user selects **"Not sure — let me run detection"**, instruct them to run:
```bash
# List available OpenVINO devices
python3 -c "from openvino import Core; print(Core().available_devices)"
# Check GPU info
ls /dev/dri/render*
# Check NPU availability
ls /dev/accel* 2>/dev/null || echo 'No NPU found'
```
Then re-ask the hardware questions based on the output.

#### Optional Parameters (ask only if relevant to the user's description)

| Parameter | Question | When to Ask |
|-----------|----------|-------------|
| **Tracking** | Enable object tracking across frames? | User mentions tracking, counting, or re-identification |
| **Recording / NVR** | Enable event-triggered video recording? | User mentions recording, saving clips, or NVR |
| **Multi-camera** | How many camera streams? Provide RTSP URLs or camera count | User mentions multiple cameras or ONVIF |
| **Alert / notification** | What alert conditions and destinations? | User mentions alerts, thresholds, or notifications |
| **Custom analytics** | Describe the custom logic to run per-frame | User mentions custom processing, filtering, or business rules |
| **Output directory** | Where should output files be saved? Default: `./results/` | User mentions specific output path |

#### Gathering Rules

1. Present required parameters as a **single interactive questionnaire** (not one-by-one) using the ask-questions tool
2. For parameters with fixed choices, provide **selectable options** with multi-select where noted
3. Always allow **freeform text** alongside options so users can specify custom values
4. If the user's initial prompt already covers all required parameters, skip the questionnaire and proceed directly
5. After collecting answers, summarize the pipeline configuration back to the user in a brief table before generating code

### Step 1 — Map user Description into DL Streamer Pipeline

Generate a proxy pipeline string that captures the user's intent using DL Streamer elements. Use the [Pipeline Construction Reference](./references/pipeline-construction.md) to identify which elements to use for each part of the pipeline (e.g. source, decode, inference, metadata handling, sink).

### Step 2 — Identify AI Models, Convert to OpenVINO IR, and Generate Download Scripts

DL Streamer inference elements **only accept OpenVINO IR format** (`.xml` + `.bin`). For any model not already in IR format, the generated application must include a conversion step.

#### 2a. Check if model is already supported by existing downloaders

| Model downloader | Typical Models  | Path |
|--------|-------------|------|
| download_public_models.sh | Traditional computer vision models | `samples/download_public_models.sh` |
| download_hf_models.py | HuggingFace models, including VLM models and Transformer-based detection/classification models (RTDETR, CLIP, ViT) | `scripts/download_models/download_hf_models.py` |
| download_ultralytics_hf_models.py | Specialized model downloader for Ultralytics YOLO models | `scripts/download_models/download_ultralytics_models.py` |

If a model is found in one of the above scripts, use that script to download the model and add model download instructions to the application README.

#### 2b. For models NOT in existing downloaders — generate conversion code

If the model does not exist in any downloader, use the **model format** collected in Step 0 and the conversion routing table to generate a `download_models.py` script that:

1. **Downloads** the model from the provided URL or identifier
2. **Converts** the model to OpenVINO IR format using the appropriate tool
3. **Caches** the converted model in the `models/` directory to avoid re-conversion on subsequent runs

**Conversion decision tree:**

```
Is the model already OpenVINO IR (.xml + .bin)?
  └─ YES → Use directly, no conversion needed
  └─ NO  → What is the source format?
       ├─ Ultralytics YOLO (.pt) → YOLO("model.pt").export(format="openvino")
       ├─ HuggingFace Transformers → optimum-cli export openvino --model <id>
       ├─ PaddlePaddle v3 PIR (.json) → paddle2onnx → ovc
       ├─ PaddlePaddle v2 (.pdmodel) → ovc model.pdmodel
       ├─ ONNX (.onnx) → ovc model.onnx
       ├─ TensorFlow SavedModel → ovc saved_model_dir
       ├─ TensorFlow Lite (.tflite) → ovc model.tflite
       ├─ Keras (.h5/.keras) → ovc model.h5
       └─ Generic PyTorch (.pt/.pth) → torch.onnx.export() → ovc model.onnx
```

**Generated `download_models.py` must:**
- Skip conversion if `models/<model_name>/model.xml` already exists (cache check)
- Use `subprocess.run()` for CLI tools (`ovc`, `optimum-cli`, `paddle2onnx`) with `check=True`
- Export Ultralytics models in a **subprocess** if GStreamer is loaded in the same process (avoids OpenVINO runtime conflicts)
- Include `--weight-format int8` for detection/classification models, `--weight-format int4` for VLMs
- Add all required conversion packages to `requirements.txt` (e.g. `paddle2onnx`, `optimum[openvino]`, `ultralytics`)

See [Model Preparation Reference](./references/model-preparation.md) for detailed conversion code patterns for each source format.

### Step 2b — Apply Hardware-Aware Performance Tuning

Using the hardware and performance information collected in Step 0, apply the following tuning rules when setting properties on `gvadetect`, `gvaclassify`, `gvagenai`, and other pipeline elements.

#### Device Assignment Rules

| User Hardware | Detection (`gvadetect`) | Classification (`gvaclassify`) | VLM (`gvagenai`) | Decode | Encode |
|--------------|------------------------|-------------------------------|-------------------|--------|--------|
| iGPU only | `device=GPU` | `device=GPU` | `device=GPU` | VA-API (`decodebin3`) | `vah264enc` |
| iGPU + NPU | `device=GPU` (detection) | `device=NPU` (classification) | `device=GPU` | VA-API | `vah264enc` |
| Arc / Flex dGPU | `device=GPU` | `device=GPU` | `device=GPU` | VA-API | `vah264enc` |
| iGPU + dGPU | `device=GPU.0` (dGPU for heavy models) | `device=GPU.1` (iGPU for lighter models) | `device=GPU.0` | VA-API on dGPU | `vah264enc` |
| CPU only | `device=CPU` | `device=CPU` | `device=CPU` | Software (`decodebin3`) | `x264enc` |
| Xeon (multi-core) | `device=CPU` with `ie-config=PERFORMANCE_HINT=THROUGHPUT` | `device=CPU` with throughput mode | `device=CPU` | Software | `x264enc` |

#### Pre-Process Backend Rules

| Pipeline Memory Path | `pre-process-backend` Value | When to Use |
|---------------------|----------------------------|-------------|
| GPU decode → GPU inference | `vaapi-surface-sharing` | **Recommended default for GPU** — zero-copy, avoids CPU↔GPU transfers |
| GPU decode → CPU inference | `opencv` or `ie` | Frames must be copied to system memory anyway |
| CPU decode → CPU inference | `ie` | All in system memory, OpenVINO preprocessing |
| NPU inference | `ie` | NPU requires system memory input |

#### Throughput vs Latency Tuning

| Performance Goal | `nireq` | `batch-size` | `ie-config` | `inference-interval` |
|-----------------|---------|-------------|-------------|---------------------|
| **Maximum throughput** | `nireq=4` (or higher) | `batch-size=` stream_count × 2 | `PERFORMANCE_HINT=THROUGHPUT` | `inference-interval=2` or `3` (skip frames) |
| **Lowest latency** | `nireq=1` | `batch-size=1` | `PERFORMANCE_HINT=LATENCY` | `inference-interval=1` (every frame) |
| **Balanced** | `nireq=2` | `batch-size=` stream_count | `PERFORMANCE_HINT=LATENCY` | `inference-interval=1` |

#### Multi-Stream Scaling Rules

| Stream Count | Optimization | How |
|-------------|-------------|-----|
| 1 stream | Default settings | No special tuning needed |
| 2–4 streams | Share model instances | Set `model-instance-id=detect0` on all `gvadetect` elements to enable cross-stream batching |
| 5–16 streams | Share instances + batch | `model-instance-id=detect0 batch-size=N nireq=4 ie-config=PERFORMANCE_HINT=THROUGHPUT` |
| 16+ streams | Multi-device + batching | Consider `device=MULTI:GPU,CPU` or split streams across `GPU.0`/`GPU.1`; use `gvafpsthrottle` to cap per-stream FPS |

#### Element-Specific Tuning Cheatsheet

```
# High-throughput GPU detection (multi-stream)
gvadetect model=${MODEL} device=GPU nireq=4 batch-size=8 \
  ie-config=PERFORMANCE_HINT=THROUGHPUT \
  pre-process-backend=vaapi-surface-sharing \
  inference-interval=3 model-instance-id=det0

# Low-latency GPU classification
gvaclassify model=${MODEL} device=GPU nireq=1 batch-size=1 \
  ie-config=PERFORMANCE_HINT=LATENCY \
  pre-process-backend=vaapi-surface-sharing

# NPU classification (offload from GPU)
gvaclassify model=${MODEL} device=NPU nireq=2 \
  pre-process-backend=ie

# VLM on GPU with frame sampling
gvagenai model-path=${VLM_DIR} device=GPU frame-rate=1.0 \
  generation-config="max_new_tokens=100"

# CPU-only Xeon throughput mode
gvadetect model=${MODEL} device=CPU nireq=8 batch-size=4 \
  ie-config=PERFORMANCE_HINT=THROUGHPUT,NUM_STREAMS=AUTO
```

Apply these rules when constructing the pipeline in Step 3a or Step 4. Add `queue` elements between inference stages to decouple threading.

### Step 3a [Command Line Application] — Construct Command Line Pipeline

If the user asks for a command-line application, construct a `gst-launch-1.0` pipeline string using the identified DL Streamer elements. Follow established conventions for element properties, caps negotiation, and metadata handling as seen in the reference command line samples.

### Step 3b [Python Application] — Decompose the User Request into Design Patterns

If the user asks for a Python application or wants to add custom logic as new Python elements, decompose the requested pipeline into one or more of the design patterns listed in the [Design Patterns Reference](./references/design-patterns.md). This will guide the structure of the application, including how to construct the pipeline, where to add callbacks, and how to handle models and metadata.

Map the user's description to one or more of these patterns:

| Pattern | When to Apply |
|---------|---------------|
| **Pipeline Core** | Always — every app needs source → decode → sink |
| **AI Inference** | User wants object detection (`gvadetect`), classification/OCR (`gvaclassify`), or VLM (`gvagenai`) |
| **Pad Probe Callback** | User needs simple custom logic, like per-frame metadata inspection or adding overlays |
| **Custom Python Element** | User needs non-trivial custom analytics logic that runs inside the pipeline |
| **AppSink Callback** | User wants to continue processing of frames or metadata in their own application |
| **Dynamic Pipeline Control** | User wants conditional routing, valve, or tee-based branching |
| **Cross-Branch Signal Bridge** | User has a tee with branches that must exchange state |
| **Model Download & Export** | User references HuggingFace, Ultralytics, or optimum-cli models |
| **Asset Resolution** | User expects auto-download of video or model files |
| **Multi-Camera / RTSP** | User wants to process multiple camera streams |
| **File Output (gvametapublish)** | User wants to save JSONL results — use `gvametapublish file-format=json-lines` as default |

### Step 4 [Python Application] — Assemble the Application

Read the [Coding Conventions Reference](./references/coding-conventions.md) before writing a Python application.
Use the [Application Template](./assets/python-app-template.py) as a starting skeleton. Compose the application by:

1. Selecting the appropriate **pipeline construction** approach — see [Pipeline Construction Reference](./references/pipeline-construction.md)
2. Following the **Pipeline Design Rules** (Rules 1–5) in the Pipeline Construction Reference — prefer auto-negotiation, GPU/NPU inference, `gvaclassify` for OCR, `gvametapublish` for JSON
3. Assembling the **pipeline string** from DL Streamer elements listed in the Pipeline Construction Reference
4. Preparing models using the correct export method — see [Model Preparation Reference](./references/model-preparation.md)
5. Adding **callbacks/probes** as needed
6. Adding **custom Python elements** if the user needs inline analytics
7. Wiring up **argument parsing** and **asset resolution**
8. Adding the **pipeline event loop**

### Step 5 — Generate Sample Application

Generate sample application following the directory structure outlined at the beginning of this document.

If an application requires Python dependencies, list them in `requirements.txt` and then create and activate a local Python environment prior to running the application. If OpenVINO python runtime is required, please make sure it is added to `requirements.txt` with same version as OpenVINO runtime installed with DL Streamer.

```bash
python3 -m venv .<app_name-venv>
source .<app_name-venv>/bin/activate
pip install -r requirements.txt
```

### Step 6 — Run and Verify Application (conditional)

**Only execute this step if the user selected "Yes — run and verify output" in Step 0.**
If the user selected "No — generate code only", skip this step entirely and present the generated code.

#### 6a. Select Execution Environment

Based on the **DL Streamer image** parameter collected in Step 0:

| User Selection | Execution Method |
|---------------|------------------|
| `intel/dlstreamer:2026.0.0-ubuntu24` | Run inside Docker container (see below) |
| `intel/dlstreamer:2026.0.0-ubuntu22` | Run inside Docker container (see below) |
| `intel/dlstreamer:latest` | Run inside Docker container (see below) |
| `Local / native install (no Docker)` | Run directly on host using local DL Streamer install |
| Custom image tag | Run inside Docker container with the provided tag |

#### 6b. Docker Execution

When running inside a Docker container, use the following command template:

```bash
docker run --rm -it \
  --device /dev/dri \
  -v $(pwd)/<app_dir>:/app \
  -w /app \
  <docker_image> \
  bash -c "pip install -r requirements.txt 2>/dev/null; python3 <app_name>.py <args>"
```

**Docker flags:**
- `--device /dev/dri` — required for GPU access (VA-API decode, GPU inference)
- `-v $(pwd)/<app_dir>:/app` — mount the generated application directory
- Add `-e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix` if the pipeline uses `autovideosink` (display output)
- Add `--device /dev/accel0` if the pipeline uses NPU
- Add `-v /dev/video0:/dev/video0` if the pipeline uses a USB camera

#### 6c. Native Execution

When running natively (no Docker), activate the local virtual environment and run:

```bash
cd <app_dir>
python3 -m venv .<app_name-venv>
source .<app_name-venv>/bin/activate
pip install -r requirements.txt
python3 <app_name>.py <args>
```

#### 6d. Verify Output

After running the application, verify the output matches expectations:
- **JSONL output**: Check that the file exists, is non-empty, and contains expected fields (e.g. `objects`, `label`, `confidence`)
- **Video output**: Check that the output `.mp4` file was created and has non-zero size
- **Console output**: Check that metadata or detection results were printed
- **Exit code**: Confirm the application exited with code 0 (no errors)

If the user provided a natural language description of the expected output, verify that the output matches the description. If the pipeline fails, diagnose the error, fix the generated code, and re-run.


