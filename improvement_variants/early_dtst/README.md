# Early-DTST Variant

This folder provides a standalone snapshot of the Early-DTST implementation used as the final improvement in this project.

Early-DTST means Early Dynamic Training Stage Transition. The method follows the original HDR-GS joint optimization in the early stage, then freezes Gaussian geometry and HDR SH color after 5K iterations and continues optimizing the tone-mapper only. This is intended to reduce redundant late-stage optimization while preserving LDR rendering quality.

The root-level files in the `improvement` branch also correspond to this Early-DTST main method. This folder is kept as a clear variant snapshot for review.

## Included files

- `train_real.py`
- `train_synthetic.py`
- `scene/gaussian_model.py`
- `gaussian_renderer/__init__.py`

## Code version

This implementation corresponds to the Early-DTST code version restored from:

```bash
git checkout bb98f8e -- train_real.py train_synthetic.py scene/gaussian_model.py gaussian_renderer/__init__.py
```
