<div align="center">

  # 🧠 TypeR Models

  ### AI Models for TypeR Photoshop Extension

  [![TypeR](https://img.shields.io/badge/TypeR-Extension-blue?style=for-the-badge)](https://github.com/darkmax159159357/TypeR)
  [![Discord](https://img.shields.io/badge/Discord-Join%20Server-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/PZhSh9bJ)

  ---

  This repository documents the AI models used by [TypeR](https://github.com/darkmax159159357/TypeR) for automatic text detection and removal in manga, comics, and webtoons.

  **All models are downloaded automatically by TypeR on first use — no manual setup required.**

  </div>

  ---

  ## 📦 Models Overview

  | Model | Purpose | Size | Architecture | Auto-Download |
  |-------|---------|------|-------------|---------------|
  | **big-lama.pt** | Text removal (inpainting) | ~410 MB | LaMa (Large Mask Inpainting) | ✅ Yes |
  | **public.pt** | Text detection (2-class) | ~52 MB | YOLOv8 | ✅ Yes |
  | **best.pt** | Bubble detection (1-class) | ~22 MB | YOLOv8 | Manual only |

  ---

  ## 🧹 LaMa — Text Removal Model

  ### big-lama.pt

  | Property | Details |
  |----------|---------|
  | **Architecture** | FFCResNetGenerator (Fast Fourier Convolution ResNet) |
  | **Paper** | [Resolution-robust Large Mask Inpainting with Fourier Convolutions](https://arxiv.org/abs/2109.07161) |
  | **Input** | 4-channel (RGB image + binary mask) |
  | **Output** | 3-channel RGB (inpainted result) |
  | **Resolution** | Any (resolution-robust) |
  | **Size** | ~410 MB |
  | **Format** | PyTorch Lightning checkpoint |
  | **Source** | Google Drive (auto-downloaded) |
  | **Device** | CUDA (GPU) or CPU — auto-detected |

  ### How It Works
  1. User selects a text region in Photoshop
  2. TypeR captures the selection area + surrounding context as input image
  3. A binary mask is generated for the text area
  4. LaMa processes the 4-channel input (image + mask) and outputs a clean result
  5. The inpainted result is placed as a new layer in Photoshop

  ### Network Architecture
  ```
  Input (4ch: RGB + Mask)
      │
      ├── 3× Downsampling Convolutions
      │       └── Progressive channel increase: 64 → 128 → 256
      │
      ├── 18× FFC ResNet Blocks
      │       └── Fast Fourier Convolution (ratio: 0.75)
      │       └── Spectral transform for global context
      │       └── Real + imaginary frequency processing
      │
      ├── 3× Upsampling Convolutions
      │       └── Progressive channel decrease: 256 → 128 → 64
      │
      └── Output (3ch: RGB, sigmoid activation)
  ```

  ### Configuration
  ```yaml
  generator:
    kind: ffc_resnet
    input_nc: 4          # RGB + mask
    output_nc: 3         # RGB output
    ngf: 64              # Base feature channels
    n_downsampling: 3    # Downsample steps
    n_blocks: 18         # FFC ResNet blocks
    add_out_act: sigmoid # Output activation
    resnet_conv_kwargs:
      ratio_gin: 0.75    # FFC ratio
      ratio_gout: 0.75
  ```

  ---

  ## 🔍 YOLOv8 — Text Detection Models

  ### public.pt (Primary)

  | Property | Details |
  |----------|---------|
  | **Architecture** | YOLOv8 (Ultralytics) |
  | **Classes** | 2: `text_bubble`, `text_free` |
  | **Input** | Full manga/comic page image (PNG) |
  | **Output** | Bounding boxes with class labels and confidence scores |
  | **Size** | ~52 MB |
  | **SHA-256** | `10bc9f702698148e079fb4462a6b910fcd69753e04838b54087ef91d5633097b` |

  ### best.pt (Alternative)

  | Property | Details |
  |----------|---------|
  | **Architecture** | YOLOv8 (Ultralytics) |
  | **Classes** | 1: `bubble` |
  | **Size** | ~22 MB |
  | **Usage** | Manual placement only (not auto-downloaded) |

  ### Detection Classes

  | Class | Description | Usage |
  |-------|-------------|-------|
  | `text_bubble` | Text inside speech/thought bubbles | LaMa inpainting removes the text |
  | `text_free` | Free-floating text (SFX, narration boxes) | LaMa inpainting removes the text |

  ### How Auto Clean Works
  ```
  Full Page Image (PNG)
          │
          ▼
     ┌─────────┐
     │ YOLOv8  │ ──→ Detected regions: [{x1,y1,x2,y2, class, conf}, ...]
     └─────────┘
          │
          ▼ (for each detected region)
     ┌──────────────────┐
     │ Create Selection  │ ──→ Photoshop rectangular selection
     └──────────────────┘
          │
          ▼
     ┌──────────────────┐
     │ Capture + Mask    │ ──→ Selection area + context padding + binary mask
     └──────────────────┘
          │
          ▼
     ┌──────────────────┐
     │ LaMa Inpainting   │ ──→ Clean result (text removed)
     └──────────────────┘
          │
          ▼
     ┌──────────────────┐
     │ Apply Result      │ ──→ New layer in Photoshop
     └──────────────────┘
  ```

  ---

  ## 📁 Storage Locations

  Models are stored locally after first download:

  | OS | Base Path |
  |----|-----------|
  | **Windows** | `%LOCALAPPDATA%\TypeR_lama\` |
  | **macOS** | `~/Library/Application Support/TypeR_lama/` |

  ### Directory Structure
  ```
  TypeR_lama/
  ├── lama-big/
  │   ├── big-lama.pt          # LaMa inpainting model (~410 MB)
  │   └── config.yaml          # Model configuration
  ├── detection/
  │   └── public.pt            # YOLOv8 detection model (~52 MB)
  └── lama_clean.py            # Inference script (auto-copied from extension)
  ```

  ---

  ## ⚙️ Dependencies

  TypeR automatically installs all required Python packages on first use:

  ### For LaMa Clean (Text Removal)
  | Package | Purpose |
  |---------|---------|
  | `torch` (CPU) | PyTorch deep learning framework |
  | `pillow` | Image loading and saving |
  | `numpy` | Numerical operations |

  ### For Auto Clean (Detection + Removal)
  | Package | Purpose |
  |---------|---------|
  | `ultralytics` | YOLOv8 inference engine |
  | `opencv-python-headless` | Image processing for detection |
  | `torchvision` | PyTorch vision utilities |

  ### Python Requirements
  - **Python 3.8+** (auto-detected from system PATH)
  - **CUDA GPU** (optional) — Significantly speeds up processing, but CPU works fine

  ---

  ## 🔒 Integrity Verification

  ### Detection Model (public.pt)
  Downloaded models are verified using SHA-256 hash:
  ```
  SHA-256: 10bc9f702698148e079fb4462a6b910fcd69753e04838b54087ef91d5633097b
  ```

  ### Minimum Size Check
  - Detection model: Must be ≥ 5 MB (rejects corrupted/incomplete downloads)
  - LaMa model: Must be ≥ 100 MB

  ---

  ## 🔄 Auto-Setup Flow

  ```
  User clicks "LaMa Clean" or "Auto Clean"
          │
          ▼
     ┌──────────────────────┐
     │ Detect Python 3.8+    │ ──→ Searches PATH, common install locations
     └──────────────────────┘
          │
          ▼
     ┌──────────────────────┐
     │ Install pip packages  │ ──→ torch (CPU), pillow, numpy
     │ (if not installed)    │     + ultralytics, opencv, torchvision (for Auto Clean)
     └──────────────────────┘
          │
          ▼
     ┌──────────────────────┐
     │ Download model(s)     │ ──→ big-lama.pt (~410MB) from Google Drive
     │ (if not present)      │     public.pt (~52MB) with SHA-256 check
     └──────────────────────┘
          │
          ▼
     ┌──────────────────────┐
     │ Copy inference script │ ──→ lama_clean.py from extension to data dir
     └──────────────────────┘
          │
          ▼
     ┌──────────────────────┐
     │ Run inference         │ ──→ Process image through model pipeline
     └──────────────────────┘
  ```

  ---

  ## 📚 References

  - **LaMa**: Suvorov et al., *"Resolution-robust Large Mask Inpainting with Fourier Convolutions"*, WACV 2022 — [Paper](https://arxiv.org/abs/2109.07161) | [GitHub](https://github.com/advimman/lama)
  - **YOLOv8**: Ultralytics, *"YOLOv8: State-of-the-Art Object Detection"* — [GitHub](https://github.com/ultralytics/ultralytics) | [Docs](https://docs.ultralytics.com)
  - **PyTorch**: Meta AI, *"PyTorch: An Imperative Style, High-Performance Deep Learning Library"* — [Website](https://pytorch.org)

  ---

  <div align="center">

  **Part of the [TypeR](https://github.com/darkmax159159357/TypeR) project**

  *Bringing AI-powered tools to the typesetting community*

  </div>
  