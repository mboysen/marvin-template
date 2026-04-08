# Decision Log

Rolling log of decisions made during MARVIN sessions. Referenced by `/start` for recent context.

---

### 2026-04-04 - Synthetic Images Are Supplementary Only
**Decision:** Synthetic image generation (FLUX/SDXL) is useful for bootstrapping zero-data species but won't break the 80% mAP ceiling.
**Context:** Run 8 added 4K synthetic images to 38K real images. Result: 79.4% mAP vs Run 6's 80.2% with zero synthetic. Domain gap between AI-generated and real images limits transfer. Model annotating its own synthetic images is circular.
**Status:** Active

### 2026-04-04 - 152K Old Dataset Paused
**Decision:** Paused species annotation of the 152K old binary weed dataset pending quality assessment.
**Context:** Images are 640x640 heavily compressed phone photos (48-54KB), not robot camera images as previously documented. Dead pixel artifact present. Quality too poor to improve the model — may actually hurt it.
**Status:** Active — needs visual review via `\\192.168.1.214\weed_152k`

### 2026-04-04 - Path to 90%+ Requires Real Robot Camera Data
**Decision:** Focus on collecting high-quality images from the production robot camera instead of more synthetic or low-quality data.
**Context:** The 80% ceiling is a data quality problem, not quantity. Run 6 hit 80.2% with 33K quality images. Adding 4K synthetic or 152K low-quality phone photos won't break through. Active learning from the real robot is the path forward.
**Status:** Active

### 2026-04-04 - Re-annotate with Whole-Plant DINO Prompts (v4a Pipeline)
**Decision:** Re-run the v4a annotation pipeline using whole-plant prompts ("dandelion plant . dandelion rosette") instead of flower-focused prompts ("dandelion weed"). Follow with RF-DETR bootstrap and digital twin integration.
**Context:** Root cause of 80% mAP ceiling identified: all annotations since day one were flower-focused because DINO prompts targeted flowers, not whole plants. v4a hit 94% using whole-plant prompts + bootstrap + digital twin. Initial 5,359-image test with correct prompts looks good. SAM expansion approach was tested and rejected — can't separate overlapping plants in dense scenes.
**Status:** Complete — achieved 90.8% mAP through 4 bootstrap iterations + quality filtering

### 2026-04-07 - Quality Filtering Is the Key to Breaking 90%
**Decision:** Use the trained model as a quality gate — only accept images where the model detects the target species at >50% confidence with reasonable box size. Reject noisy, mislabeled, or ambiguous images.
**Context:** Run 4 with quality-filtered data (5,519 accepted out of 6,432) hit 90.8% mAP vs 87.8% without filtering. Fewer but cleaner images > more noisy images. The 913 rejected images were actively hurting training.
**Status:** Active
