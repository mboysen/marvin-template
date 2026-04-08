# Current State

Last updated: 2026-04-07

## Active Priorities

1. **Deploy production model** — 90.8% mAP model ready. Checkpoint: `outputs/bootstrap_run4_quality/checkpoint_best_ema.pth`
2. **Determine production robot camera** — Need sample images for field validation.
3. **Active learning pipeline design** — Deploy model on robot, save uncertain detections for human review to push toward 95%

## Open Threads

- **Best model: 90.76% mAP 50:95, 93.95% mAP 50** (Run 4 quality-filtered + DT), checkpoint at `outputs/bootstrap_run4_quality/checkpoint_best_ema.pth`
- Full pipeline: 80.2% → 84.3% → 87.5% → 87.8% → **90.8%** across 4 iterations
- Digital twin integrated (930 frames, 3,532 annotations, 29 species — only crabgrass missing)
- Zeus powered off
- DGX Spark has all checkpoints and data
- Blackwell GPU workarounds applied (torch.prod monkey-patch, transformers 5.x bertwarper fix)
- LinkedIn proxy token expires 2026-04-17

## Recent Context

- 2026-04-07: Quality-filtered run hit **90.8% mAP** — production-grade. Two-stage (87.4%) didn't beat bootstrap approach.
- 2026-04-06: Bootstrap 3 completed (87.8% mAP), Zeus shut down
- 2026-04-05: Bootstrap 1 (84.3%), Bootstrap 2 (87.5%), DT integration, Bootstrap 3 started
- 2026-04-04: Root cause found (flower-focused DINO prompts), whole-plant re-annotation, pipeline fixes

---

*This file is the source of truth for session continuity. MARVIN reads it on startup and updates it on every /end and /update.*
