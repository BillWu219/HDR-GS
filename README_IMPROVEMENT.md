# README_IMPROVEMENT

This repository is a fork of the official HDR-GS implementation. The `improvement`
branch contains the code used for the final project report.

## 1. Summary of Improvements

### 1.1 Early-DTST: Early Dynamic Training Stage Transition

Early-DTST is the main improvement used in the final experiments. It targets the
redundant full-lifecycle joint optimization problem in HDR-GS.

In the original HDR-GS training pipeline, Gaussian geometry, HDR SH color, opacity,
scaling, rotation, and the tone-mapper are optimized throughout the whole training
lifecycle. Early-DTST changes this into a two-stage schedule:

- Stage I, iterations 1-5K: jointly optimize Gaussian geometry, HDR SH color, and
  tone-mapper parameters.
- Stage II, after iteration 5K: freeze Gaussian geometry and HDR SH color, and
  continue optimizing only the tone-mapper.

The implementation is in the root code of this branch:

- `scene/gaussian_model.py`
- `train_real.py`
- `train_synthetic.py`

The corresponding code version is based on commit:

```bash
git checkout bb98f8e -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
```

This method corresponds to the report sections on redundant full-lifecycle joint
optimization and Early-DTST.

### 1.2 HDR-LDR Gradient Decoupler: Full Detach Variant

The second tested modification is HDR-LDR Gradient Decoupler. It targets the
possible gradient conflict caused by using one shared HDR color representation for
both HDR reconstruction and LDR tone-mapped rendering.

The implemented variant is Full Detach. Before the HDR SH color enters the LDR
tone-mapping branch, the color tensor is detached. Therefore, the LDR branch uses
the same color values in the forward pass, but the LDR loss no longer directly
back-propagates gradients to HDR SH color.

This branch keeps Early-DTST as the root main method. The Full Detach code snapshot
is provided separately in:

```text
improvement_variants/hdr_ldr_gradient_decoupler/
```

The corresponding code version is based on commit:

```bash
git checkout 2df0218 -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
```

This method corresponds to the report sections on shared HDR-LDR color gradient
conflict and the HDR-LDR Gradient Decoupler ablation.

## 2. Reproduction Commands

Run the original baseline after restoring the official baseline files:

```bash
git checkout 2012062 -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
python train_synthetic.py --config config/bathroom.yaml --eval --gpu_id 0
python train_real.py --config config/computer.yaml --eval --gpu_id 0 -r 4
```

Run the main Early-DTST version from this branch:

```bash
python train_synthetic.py --config config/bathroom.yaml --eval --gpu_id 0
python train_real.py --config config/computer.yaml --eval --gpu_id 0 -r 4
```

Run the Full Detach HDR-LDR Gradient Decoupler variant:

```bash
git checkout 2df0218 -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
python train_synthetic.py --config config/bathroom.yaml --eval --gpu_id 0
```

The Full Detach experiment is evaluated on the synthetic `bathroom` scene because
HDR ground truth is required to evaluate HDR reconstruction quality.

## 3. Environment

The experiments were conducted with the official HDR-GS codebase and the following
local environment:

- Python: 3.10
- CUDA: 11.8
- PyTorch: 2.0.1
- GPU used in this project: NVIDIA RTX 4060, 8 GB VRAM

The original HDR-GS paper reports experiments on a single NVIDIA RTX A5000 GPU.
Because the RTX 4060 has limited VRAM, the real `computer` scene was trained with
quarter-resolution input using `-r 4`.

Install dependencies following the official HDR-GS instructions:

```bash
conda env create -f environment.yml
conda activate hdr-gs
```

If CUDA extension compilation fails on Windows, make sure that the local CUDA,
PyTorch, and Microsoft Visual Studio Build Tools versions are compatible.

## 4. Experiment Logs and Data Package

Training logs and rendered results are not stored in this GitHub repository to
avoid uploading large experiment outputs. They are provided in the project data
package:

```text
实验数据包/
├─ 渲染结果/
├─ 评估指标的原始数值/
│  ├─ all_metrics.csv
│  ├─ all_metrics.json
│  └─ README_metrics.txt
└─ raw_logs/
```

The `all_metrics.csv` and `all_metrics.json` files contain the raw PSNR, SSIM,
LPIPS, training time, test speed, and evaluation time cost values extracted from
the corresponding `log.txt` files.

## 5. Notes on Code Scope

Only code files related to the proposed improvements are changed. Dataset files,
rendered outputs, model checkpoints, `.pth` files, `.ply` files, and videos are
excluded from this repository and are provided separately in the experiment data
package when needed.
