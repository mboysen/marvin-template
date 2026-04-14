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

### 2026-04-11 - Ollama for Day-to-Day, Claude for Weekly/On-Demand
**Decision:** Split the knowledge pipeline between local Ollama (Qwen 3 8B) and Claude Opus. Ollama handles routine tasks: session summarization, project classification, incremental concept updates. Claude/Opus reserved for full project recompiles and topic consolidations.
**Context:** Running every session summary through Opus was wasteful — simple extraction tasks burn expensive Opus calls for work Qwen handles fine. Split by task complexity: extraction/classification = Ollama (free, fast, ~22s/session), synthesis/cross-referencing = Claude. `naware-sync` uses Ollama by default, `--use-claude` or `--force` for the expensive path.
**Status:** Active

### 2026-04-12 - Morphology Prompts Beat Taxonomy for Fine-Grained Grass Detection
**Decision:** Use morphology-descriptive GroundingDINO prompts ("bushy dark green grass clump", "wide drooping grass leaf clump") instead of taxonomic prompts ("cereal rye plant", "winter wheat plant") for rye/wheat auto-labeling at tillering stage.
**Context:** Taxonomic prompts produced zero discrimination between rye and wheat — GroundingDINO's BERT encoder can't distinguish closely related grass species. Morphology prompts correctly identified most real rye clumps (196 confirmed out of 447 detections) because structural descriptors map to visually grounded features. The 52% false positive rate is manageable via bootstrap + quality filtering.
**Status:** Active — apply morphology-descriptive prompts for any future fine-grained grass/crop discrimination tasks

### 2026-04-13 - Two-Stage Detection Beats Single-Stage for Fine-Grained Discrimination
**Decision:** Use a two-stage pipeline (RF-DETR detector → EfficientNet classifier) instead of a single detector for rye-in-wheat. The detector finds plant clumps at low threshold, the classifier scores each at 256×256 full resolution.
**Context:** RF-DETR at 640×640 sees rye at ~42px — too small for texture discrimination. The patch classifier at 256px achieved 97.3% accuracy on the same data the detector plateaued at 55%. Separating localization (easy) from classification (hard) unlocks the texture signal. Copy-paste augmentation and progressive resolution did NOT help (42.3% and 37.4% vs 55.5% baseline).
**Status:** Active — this is the production architecture for rye detection

### 2026-04-12 - 55% mAP is the Data Ceiling for 96 Tillering Images
**Decision:** Stop iterating training approaches on the current 96-image dataset. Bootstrap (44.2%), COCO pretrained (55.5%), and lawn weed transfer (55.6%) all converge to the same ceiling. New data is needed to improve further.
**Context:** Three independent training runs confirmed the ceiling. Bootstrap on the same closed dataset lost diversity and performed worse. Transfer learning converged faster but to the same point. The bottleneck is dataset size/diversity (96 images, 1 location, 1 growth stage), not model architecture or starting weights.
**Status:** Active — collect heading-stage images (late May–June) and images from additional fields to break through

### 2026-04-12 - Clean Before Tile for Training Data
**Decision:** Always review and correct auto-generated annotations BEFORE tiling into training crops. Never tile unreviewed auto-labels.
**Context:** GroundingDINO B_morphology produced 447 detections; Mark's review confirmed only 196 as real rye (44%). Tiling first would have multiplied 232 false positives 45× alongside real annotations, poisoning the training set. Clean-first reduced the dataset from 447 to 196 boxes but every box is verified.
**Status:** Active — standard practice for all future bootstrap rounds

### 2026-04-09 - Consolidate Wiki Articles Into Single Master Docs
**Decision:** When the LLM compiler generates many small concept articles from related sessions, consolidate them into one comprehensive master document per project/pipeline rather than keeping fragmented articles.
**Context:** The initial compile of 7 session logs produced 17 articles (15 concepts + 2 connections) that were all slices of the same weed detection pipeline story. Navigating 17 interlinked fragments was harder than reading one well-organized doc. The master doc absorbed all unique content — key insights, diminishing returns analysis, file paths, per-species data — and the fragments were deleted.
**Status:** Active — apply this pattern for future knowledge base compilation
