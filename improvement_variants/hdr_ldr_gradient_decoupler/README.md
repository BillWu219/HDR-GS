# HDR-LDR Gradient Decoupler Variant

This folder contains a code snapshot for the Full Detach HDR-LDR Gradient
Decoupler experiment.

Corresponding command:

```bash
git checkout 2df0218 -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
```

Main implementation files:

- `scene/gaussian_model.py`
  - Adds `decouple_ldr_hdr_grad`
  - Adds `decouple_hdr_color_for_ldr()`
  - Returns `hdr_color.detach()` when decoupling is enabled
- `gaussian_renderer/__init__.py`
  - Calls `pc.decouple_hdr_color_for_ldr(colors_precomp)` before the LDR
    tone-mapping branch

This variant was evaluated mainly on the synthetic `bathroom` scene because HDR
ground truth is required for HDR PSNR, SSIM, and LPIPS evaluation. The experiment
is a negative result in the report: hard Full Detach does not improve HDR
reconstruction, suggesting that LDR supervision also provides useful regularization
for HDR color learning.
