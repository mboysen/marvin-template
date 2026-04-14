# Current State

Last updated: 2026-04-13

## Active Priorities

1. **Deploy production model** — 93.7% mAP model ready. Checkpoint: `outputs/MN_weedmodel_final/checkpoint_best_ema.pth`
2. **Determine production robot camera** — Need sample images for field validation.
3. **Active learning pipeline design** — Deploy model on robot, save uncertain detections for human review to push toward 95%
4. **Wyoming rye-in-wheat detection** — Round 0 complete at **55.5% mAP** (97% precision, 69% recall). Ceiling confirmed at ~55% for 96 tillering-stage images. Next: heading-stage collection (late May–June Wyoming) + iNaturalist rye scrape. Best checkpoint: `outputs/wyoming_round0/checkpoint_best_ema.pth`. Plan at `.claude/plans/zippy-mapping-naur.md`.

## Open Threads

- **Best model: 93.7% mAP 50:95, 96.1% mAP 50** (Run 5), checkpoint at `outputs/MN_weedmodel_final/checkpoint_best_ema.pth`
- Full pipeline: 80.2% → 84.3% → 87.5% → 87.8% → 90.8% → **93.7%** across 5 iterations
- Digital twin integrated (930 frames, 3,532 annotations, 29 species — only crabgrass missing)
- Zeus powered off
- DGX Spark has all checkpoints and data
- Blackwell GPU workarounds applied (torch.prod monkey-patch, transformers 5.x bertwarper fix)
- LinkedIn proxy token expires 2026-04-17
- **Rye detection: two-stage pipeline is the architecture** — Detector (55.5% mAP) + Classifier (97.3% accuracy) = 239 high-quality detections. Round 2 detector training running on classifier-verified labels. Classifier at `outputs/wyoming_classifier/best_classifier.pth`.
- Label Studio running on sparky:8080 (mark@mark.com / mark) with Wyoming project
- Wyoming heading-stage image collection needed late May–June for MVP
- Strategic plan: `.claude/plans/zippy-mapping-naur.md`, tactical plan: `.claude/plans/wyoming-go-nogo-experiment.md`

## Recent Context

- 2026-04-08: Run 5 bootstrap hit **93.7% mAP** — best-in-class, matches v4a on 30 species
- 2026-04-07: Quality-filtered run hit 90.8% mAP. Two-stage (87.4%) didn't beat bootstrap approach.
- 2026-04-06: Bootstrap 3 completed (87.8% mAP), Zeus shut down
- 2026-04-05: Bootstrap 1 (84.3%), Bootstrap 2 (87.5%), DT integration, Bootstrap 3 started
- 2026-04-04: Root cause found (flower-focused DINO prompts), whole-plant re-annotation, pipeline fixes

---

*This file is the source of truth for session continuity. MARVIN reads it on startup and updates it on every /end and /update.*
