# ListCheck execution plan

Prepared 2026-10-20. Status: planning only, not activated. Public repository: [123ssk321/cv-listcheck](https://github.com/123ssk321/cv-listcheck), already created and verified by the parent. This task creates planning artifacts only; no implementation or remote mutation.

Source: *Quick-to-product computer-vision projects for a solo builder*, research date 2026-10-16, “Project 2 — ListCheck” and common data/license guidance. This standalone plan contains the required decisions; access to the unpublished parent report is unnecessary. Paths below are relative to the ListCheck repository root. Parent supplies shared agent instructions.

## 1. Product boundary and finish

Target one job: a small seller/catalog agency operator reviews a batch of product photographs before uploading them. Hypothesis: evidence-linked checks and repeatable reports reduce rework. Demand, willingness to pay, acceptable thresholds, and saved time are unverified.

Supported domain: opaque, predominantly single-product JPEG/PNG photos against a plain background. Deliver:

- Safe decoding, upright dimensions/aspect, alpha detection, explicit unsupported-input errors.
- Border color/variation measurements, accurately named as border measurements.
- Conservative foreground bounding-box estimate and crop/occupancy advice, with abstention.
- Sharpness/exposure metrics as inspection advice, not human-readability judgments.
- Versioned internal policy, evidence overlays, review reasons, local JSON export.
- Months 5–6: sequential catalog batches, review resolution, CSV export and bounded policy editing.

Excluded throughout the funded plan: semantic classification/counting, background removal, enhancement, OCR/barcode readability, logos/IP/moderation, video/PDF/SVG/HEIC/animated inputs, marketplace certification or automatic publishing, official marketplace packs, cloud uploads, managed APIs, model training/weights, server endpoints, accounts/billing, native mobile apps. The report's OCR and barcode features are deliberately deferred to keep the first product bounded.

Month-4 functional result means a stranger can run the checker from a public demo or downloadable runnable release, with generator, benchmark, failure gallery and limitations. A public README, screenshots, or local build alone does not meet it. Month-5/6 readiness means a working batch workflow and pilot protocol, not proof a pilot occurred.

Portfolio boundaries: FormTrace is document extraction; a separate e-invoice product must deliver rules, exceptions and reconciliation beyond extraction. EdgeDefect is anomaly triage; industrial metrology must deliver calibrated dimensions/assembly checks beyond triage. ListCheck is photo-presentation review, not any of these.

## 2. Data corrections and rights gates

Existing human-labeled public datasets are permitted when their rights cover the intended operation. Historical annotation labor is not a user labeling dependency. No user or contractor must collect, label, curate or supply a corpus. Debugging an automatically generated failure gallery must not turn into manual ground-truth creation or cherry-picking.

Noncommercial licenses do not automatically allow commercially motivated evaluation, internal R&D, threshold selection, or demos. Unknown rights block the affected operation; obtain explicit permission or omit that data. This supersedes permissive R&D assumptions in the source report. No MVTec/PlantDoc dependency exists.

Maintain separate records for source/model code, runtime, weights, training-data provenance, datasets, annotations and assets. A runtime license says nothing about weights. Baseline has no learned model or weights: record “not used,” not an incomplete audit. Proposed project code license is MIT and generated fixture release is CC0 for self-authored content; confirm ownership and notices before adoption. Neither licenses third-party inputs.

Default generator uses self-authored geometric silhouettes, procedural texture and backgrounds; no fonts, stock assets, logos or meshes. It emits exact visible/amodal geometry, masks, pixel facts and transformation parameters. Blur sigma/JPEG quality are exact transformation labels, **not exact labels of human acceptability**.

Keep three evidence tracks separate:

1. Synthetic robustness: behavior against exact scene facts and controlled corruption.
2. Public-data evaluation: existing annotation tasks, e.g. bounding-box localization/truncation, subject to rights and domain mismatch.
3. Customer validation: actual voluntary operator observations and workflow outcomes. Neither of the first two proves this third track.

Optional Open Images branch: metadata-first, 20 images initially, maximum 100 after rights clearance. Deterministic class/ID selection and automated eligibility rules only. Save original-license evidence per image; missing/ambiguous rights => exclude. No user curation. Annotation flags with unknown values remain unknown; partially annotated images cannot establish “exactly one object.” Public annotations do not provide listing suitability or blur acceptability labels. If none qualify, produce a zero-sample “rights unavailable” report and finish using synthetic data. Timebox this branch to LC-018's 16 hours.

## 3. Concrete implementation defaults

Baseline engine: custom deterministic TypeScript pixel routines, Canvas JPEG/PNG decoding and Web Worker execution. UI: Vite with plain HTML/CSS/TypeScript. Tests: Vitest plus Playwright; JSON validation: Ajv against JSON Schema 2020-12. Tools execute browser rendering/evaluation using a pinned Playwright Chromium. No OpenCV, neural inference runtime, managed API or model weights.

LC-005 selects supported Node LTS and exact dependency versions, pins the lockfile/browser revision, and checks distribution/transitive licenses. Version numbers are intentionally resolved at activation, not guessed months ahead. TypeScript/Vite source-license references were checked; remaining package/distribution licenses are gating work, not assumed approved.

Bounded alternatives: if worker/offscreen canvas support differs, decode with a hidden main-thread Canvas then transfer pixels; analysis stays in the worker. If geometry fails evaluation, ship the dimensions/border checker with experimental geometry panel. On slow hardware use a versioned 512-pixel/10-file profile. No switch to pretrained segmentation or cloud OCR within this budget; such a change requires Astra design review and rights/budget revision.

| Planned module | Interface | Responsibility |
|---|---|---|
| `src/contracts/types.ts`, `schemas/*.schema.json` | Policy, Request, Result, SceneRecord, RightsRecord, ReviewReport | Versioned types and runtime validation |
| `src/io/decode.ts` | `decode(blob, limits): Promise<DecodedImage>` | Header checks, upright RGBA, structured errors |
| `src/metrics/quality.ts`, `border.ts` | `measureQuality(pixels)`, `measureBorder(pixels, policy)` | Pure numeric observations |
| `src/metrics/foreground.ts` | `estimateForeground(pixels, policy): ForegroundEstimate` | Border flood-fill/components; eligibility reasons |
| `src/policy/evaluate.ts` | `evaluate(measurements, policy): RuleResult[]` | Thresholds, evidence, abstention |
| `src/core/analyze.ts` | `analyze(decoded, policy): AnalysisResult` | No DOM, network, seed, mask, or ground truth access |
| `src/worker/analyze.worker.ts`, `client.ts` | `analyze/cancel -> result/error` | Transfer buffers, ignore stale IDs, terminate on timeout |
| `src/ui/review.ts`, `session.ts` | render, acknowledge, reset | Original + overlays, in-memory state |
| `src/batch/queue.ts` | `run(files, policy, signal)` | Sequential processing, failure isolation |
| `src/export/report.ts`, `csv.ts` | `toJson(report)`, `toCsv(report)` | Safe versioned exports |
| `tools/generate.ts`, `oracle.ts`, `evaluate.ts` | CLI below | Seeded fixtures, independent oracle, evidence |
| `tools/public-data.ts` | audit/select/evaluate | Optional rights-gated subset |

DecodedImage: `{width,height,rgba:Uint8ClampedArray,orientationApplied,sourceSha256,mime}`. Upright original coordinates are canonical. Analyze a reduced copy, preserve original and scale mapping. Release buffers after use.

Policy: `{schemaVersion:"1.0",id,revision,source:"internal-assumption",enabledRules,thresholds,intendedDomain}`. Initial `generic-catalog-v1` thresholds:

- minWidth/minHeight 1000; aspect width/height 0.8–1.25.
- border band 2% of each side; reference RGB [255,255,255]; per-channel maximum difference 20; at least 95% matching pixels.
- crop margin 2% per edge; occupancy is bounding-box area/image area, 0.35–0.90.
- foreground eligibility initially requires per-channel border stddev <=12, dominant component >=95% of candidate foreground area, occupancy 0.05–0.95, mask IoU >=0.90 between color-distance thresholds 15/20/25.

These are internal assumptions, not marketplace rules. Freeze operating points on dev data before holdout. Border-connected foreground, white-on-white, empty masks, alpha, multiple components, gradient background or unstable masks => review. Components are not semantic object counts; eligibility is heuristic, not calibrated confidence. A missing foreground must never count as safe crop.

AnalysisRequest: `{schemaVersion:"1.0",kind:"analyze",requestId,policy,image:ArrayBuffer,mimeHint}`; cancel request `{kind:"cancel",requestId}`. Worker success `{kind:"result",requestId,result}`; error `{kind:"error",requestId,code,message}`. Validate policy and enforce allowlisted errors. Cancellation terminates/replaces a busy worker when cooperative stopping cannot interrupt synchronous work.

AnalysisResult: `{schemaVersion:"1.0",requestId,sourceSha256,engineVersion,policy:{id,revision,sha256},image:{width,height,mime},overall,checks,unsupportedReasons,timingMs}`.

RuleResult: `{id,status,reasonCode,observed,threshold,evidence,suggestion,methodVersion}`. Status is `pass|fail|review|not_applicable`; observed is number/object/null; evidence entries `{kind:"box"|"border"|"whole_image",box:[x,y,w,h]|null}`. Boxes use normalized upright [0,1] coordinates, top-left origin, half-open pixel bounds before normalization. No invented confidence score.

Decision semantics:

- Decode/limit errors are per-file errors, not fabricated scores.
- Dimensions/aspect are hard rules. Border mismatch can fail only the explicitly named BORDER_COLOR rule, never certify the entire background.
- Crop/occupancy/quality are advisory: suspicious/ineligible => review; eligible ordinary case => pass with caveat. Never hard-fail these in v1.
- Overall `fail` if any enabled hard rule fails; else `review` if a required check is unavailable/review; else `pass_scoped`. Disabled optional checks are not_applicable, excluded from pass denominators.
- UI says “Selected checks passed; listing acceptance not verified.” Suggested correction is inspect/retake/change dimensions, not automatic editing.

SceneRecord: `{schemaVersion,id,seed,baseSceneId,split,family,generatorVersion,imageSha256,rgbaSha256,width,height,instances:[{id,amodalBox,visibleBox,maskPath,visibleFraction}],backgroundRgb,transforms,truth:{dimensions,borderPixelStatistics,cropMargins,occupancy},labelSemantics,assetIds}`. Negative amodal coordinates are legal in scene metadata, not UI boxes. PNG/JPEG encoded hashes depend on pinned encoder; RGBA/scene facts are the reproducibility reference.

RightsRecord: `{id,artifactKind,version,url,licenseId,licenseTextSha256,checkedAt,attribution,rights:{commercialEvaluation,training,redistribution,publicDisplay},decision,evidencePath}`. Each right and decision is `allow|deny|unknown`; artifactKind distinguishes code/model_code/runtime/weights/data/annotation/asset. Unknown blocks use in that operation.

ReviewReport: `{schemaVersion,results,decisions:[{sourceSha256,ruleId,resolution:"acknowledged"|"retake"|"override",note}],exportedAt}`. Decisions never change immutable engine results or become training labels. Use session aliases; omit pixels, EXIF and full filenames by default.

## 4. Commands, limits and privacy

Future package scripts, not implemented commands today:

```text
npm ci
npm run typecheck
npm run test -- --run
npm run test:e2e
npm run data:generate -- --suite smoke --seed 4100 --count 32
npm run data:generate -- --suite dev --seed 4200 --count 200
npm run data:generate -- --suite holdout-v1 --seed 4300 --count 600
npm run eval -- --suite holdout-v1 --policy policies/generic-catalog-v1.json
npm run data:audit -- --manifest data/rights.json
npm run eval:public -- --manifest data/public/approved.jsonl --limit 100
npm run budget:check
npm run build
npm run preview -- --host 127.0.0.1
```

Generator output: `data/generated/<suite>/manifest.jsonl`, images/masks. Evaluator output: `reports/<suite>/metrics.json`, `report.md`, `gallery/index.html`. Every report records commit, policy/lockfile hashes, seeds/IDs, sample counts/exclusions, browser/device, command and exit code. Empty public manifest yields “skipped: no approved inputs,” zero samples, no score.

Input limits: 10 MiB/file, 24 megapixels, side <=8192; sniff headers before allocation. Reject unsupported/animated content. Analysis long side <=1024; batch <=50 files and <=250 MiB total encoded bytes; one active worker job, 10-second timeout. Targets: 8 GB CPU-only laptop, p95 decode+analysis <=2 seconds, cancel <=1 second, no UI stall >200 ms. Unmeasured targets must become evidence before claims.

No image upload, analytics, external fonts/CDNs, persistence, service worker or EXIF logging. Page hosting sees ordinary page-request metadata, so do not promise anonymous access. Revoke object URLs/release buffers on reset, cancel, replacement and completion. Images remain local; user exports results explicitly. Bundle assets; after load processing works without network. CSP and request-interception tests check zero image egress. Test malformed input, decompression limits, filename escaping and CSV formula injection.

## 5. Evaluation and small-data progression

Start 32 smoke fixtures, <=20 MiB; then 200 dev fixtures only after oracle/contracts pass. Use independent analytical rectangle/circle tests to verify transforms, clipping and direct pixel-count border truth. Runtime detector must not share oracle logic. Rendering/labels precede analysis; evaluator passes pixels only.

Final holdout: 600 images from 200 base scenes, three variants each; at least 100 known-valid and 100 known-invalid examples for every hard rule, at least 30 base scenes per primary stress family. Primary groups: crop/scale, contrast/background, blur/texture, compression/noise, exposure, ambiguity/occlusion. Generator constraints must satisfy counts or fail. Split before transformations; disjoint base IDs, template families and random seeds across dev/holdout. Corruption mechanisms may recur across splits; held-out parameter combinations/families challenge generalization. No manual success selection.

B0 baseline: dimensions/aspect and explicit border measurements. B1 adds foreground and advisory quality. Compare identical fixed inputs with always-pass, always-review and simple bounding-rectangle baselines. No training is required.

| Measurement | Proposed release gate / interpretation |
|---|---|
| Dimensions/aspect | 100% independent arithmetic agreement |
| Border rule | >=99% reference-pixel oracle agreement, failure causes disclosed |
| Eligible foreground boxes | Median IoU >=0.90; occupancy/margin errors reported |
| Unsafe synthetic crop | >=95% flagged review, <=10% erroneous crop review on eligible safe scenes |
| Eligibility coverage | >=50% intended-domain synthetic bucket; prevents always-review gaming |
| Sharpness | Spearman >=0.8 versus blur severity within informative texture strata; undefined textureless cases reported |
| Resources/privacy | Latency/cancel/egress limits above; batch soak without retained-image accumulation |

These are engineering gates, not current results or commercial guarantees. Report per-rule confusion matrices including review/not_applicable, denominators, false-fails/false-passes, coverage and every stress stratum. Report all failed cases in an automatically generated gallery. Blur rank does not prove human readability.

Use 95% Wilson intervals for per-image rates with correlation warning; primary synthetic uncertainty uses 1,000 fixed-seed scene-cluster bootstrap resamples. Count independent scenes, not only derivatives. Public evaluation reports its actual subset and annotation semantics separately.

If geometry/quality gates fail, demote them to experimental panel and release the objective checker/report workflow in month 4. Freeze thresholds before evaluation; changing thresholds requires a new policy and untouched holdout, not tuning on the failed test set.

## 6. Discovery and customer validation

Month 1: write one persona/job, five falsifiable assumptions, correction-cost taxonomy and 15-minute discussion script. Review dated primary workflow descriptions. Up to five voluntary discussions, only with explicitly authorized outreach. No automatic messaging or private image requests. No response means “hypothesis,” not invented demand; proceed with bounded technical prototype.

Months 5–6: prepare consent, counterbalanced task protocol, measurement sheet and generated catalog packs. Pilot readiness does not require customer data or participation.

Month 7: optional three operators, preferably two organizations, perform manual-vs-assisted review of 20 generated images each. Randomize/counterbalance order; measure time, hard-rule misses, overrides and willingness to repeat. Exact labels remain generator-derived; operator feedback is not a training corpus. Proposed exploratory utility gate: median time reduction >=20%, no observed increase in hard-rule misses, at least two operators willing to repeat. Report small n and synthetic-task limitations.

If unavailable, spend the bounded allocated time on disclosed self-run workflow checks and public workflow analysis. Mark customer validation “not obtained.” Technical delivery still completes; do not claim customer savings, validated pricing or demand. Pricing remains a per-seat/per-batch hypothesis.

Paid operational use would require evidence on authorized real workflows, support cost, acceptable error cost and rights. No acceptance guarantees or commercial readiness follow from synthetic/public benchmarks. Private images are optional, not a hidden prerequisite to any funded technical milestone.

## 7. Relative schedule and complete budget

Activation is when the user selects ListCheck and allocates capacity. Public repository creation/planning publication does not activate implementation. This is one alternative using at most the whole portfolio's 16 h/week, not a concurrent allocation.

| Milestone | Window after activation | Packages | Work hours | Exit |
|---|---|---|---:|---|
| M1 | Month 1 | LC-001–008 | 64 | Scope/rights/contracts and 32 fixtures |
| M2 | Month 2 | LC-009–016 | 64 | Safe browser baseline and dev evidence |
| M3 | Month 3 | LC-017–020 | 64 | Frozen evaluation/internal candidate |
| M4 | Month 4 | LC-021–024 | 64 | First functional public result |
| M5 | Month 5 | LC-025–027 | 64 | Batch/review MVP |
| M6 | Month 6 | LC-028–030 | 64 | Pilot-ready package |
| M7 | Month 7 | LC-031–032 | 64 | Use-case evidence and value decision |
| M8 | Month 8 | LC-033–034 | 64 | Bounded iteration/operations |
| M9 | Months 9–10 | LC-035–036 | 64 | Final audit/handoff/stop-go |
| Reserve | Across months 1–10 | Separate ledger | 64 | Actual draw reasons |
| Total | About 40 working weeks | 36 packages | 640 | 576 work + 64 reserve |

Each work block is four 16-hour weeks distributed within its calendar window. Four months provide roughly 277 hours (256 planned); eight months roughly 555 hours (512 planned), leaving about 43 hours for early reserve. Remaining work/reserve fits months 9–10. Use an actual weekly ledger at activation; calendar approximations use 52 weeks/year. If capacity is shared, rebaseline before execution.

Mutually exclusive cost categories across the 576 hours: discovery/use-case validation 96; learning/spikes 40; implementation/synthetic pipeline 232; testing/evaluation/security 120; docs/releases/operations 88. These categorize issue time, not additional budgets. Every estimate includes reading, implementation, checks, review and documentation. Log by issue and one category.

Reserve: compatibility/performance 24, evaluation/oracle defects 16, learning/overrun 16, release recovery 8 =64. At 48 hours drawn, narrow scope. Record draws and actuals in future `planning/status.json`; do not add parent and child slice estimates twice. No paid annotator/API or hardware budget is assumed. No spend is authorized here.

Target completion month 10; absolute stop month 12. The approximately 832-hour annual ceiling does not authorize 192 extra hours beyond this 640-hour plan. If time slips, use fallback or close with truthful failure evidence; no indefinite experiment/pilot phase.

## 8. Deployment, operations and gates

Static deployment default: GitHub Pages for the open-source demonstration, subject to current terms confirmation in LC-021. Mandatory portable fallback: downloadable `dist` release and localhost instructions. No hosted SaaS/backend. Configure `/cv-listcheck/`; test worker/assets at that base path. LC-021 supplies exact deployment/rollback commands after terms/access are known. Preserve hashes and last known-good artifact/policy. Publication uses previously granted authority when available; this planning task grants no new remote authority.

If hosting is unavailable, an authorized public runnable artifact can satisfy functional release; a local artifact cannot. Mark any missing public access as blocked, while completing independent local work.

Operations within LC-034: dependency/license review, tested browser matrix, issue template discouraging private images, seed/version reproduction, known-limitations log, rollback drills for bad policy/worker build/unsafe dependency. No 24/7 SLA or indefinite maintenance promise.

Gates: M1 narrow job/rights; M2 oracle and safe decoding; M3 frozen evidence and fallback choice; M4 functional public artifact, notices/privacy/reproduction; M6 batch/export/support readiness; M7 observed utility or explicit no-evidence result; M9 technical closure and truthful commercial stop/go.

No clinical or calibrated-hardware dependency exists. Hardware fallback is 512-pixel analysis/10-file batches, versioned and re-evaluated; algorithm fallback is dimensions/border-only with review/export. Stop commercial expansion if the desired job is merely one-off editing, review burden negates saved time, or demand remains unsupported.

## 9. Conservative-model handoff

Future executors: Luna medium or GPT-5.6 medium, one dependency-ready package at a time. Escalate design changes to Astra HIGH. These are routing instructions, not a claim about this planning session's model configuration.

Ready: all dependency IDs done, rights allowed or documented optional-data fallback selected, budget available, implementation actually activated. Read parent instructions, README, this plan, issue body and dependency evidence. Preserve unrelated changes. Do not ask for routine confirmation within authorized scope.

Every >12-hour issue is an epic with <=12-hour child slices included in the parent estimate; do not create duplicate estimated children. Record per-slice state outside the exact backlog schema in `planning/status.json`.

Done: outputs exist; acceptance and named validation pass, or the expressly permitted skipped/fallback outcome is documented. `docs/reviews/<ID>.md` records diff/commit, actual hours, commands/exit codes, artifacts, rights, failure examples and limitations. A separate review pass must verify claims against evidence; assertion is insufficient. Status fields: issueId/state/slices/actualHours/evidencePaths/blocker/nextAction; retain activation date and reserve ledger.

Blocked: reproducible environment failure, contract conflict, unresolved required rights, or absent authority/access for a necessary remote action. Record exact cause and safe alternatives; continue independent ready work. Optional public-data/customer absence produces a documented unavailable outcome, not a technical blocker. Do not mark skipped tests as passed.

Escalate only for changed schema/interface, evidence meaning, new model/data/host dependency, weakened gate, increased budget, scope or threat-model changes. Supply affected IDs/paths, evidence, alternatives and recommendation; pause only dependent work. No rights bypass or needless reconfirmation.

Handoff: completed files, review evidence, failures/limits, next ready IDs, actual/remaining hours and reserve, design decisions outstanding. At every milestone reconcile schedule, budget and claims. Final handoff ends in maintain/prototype-freeze/commercial-investigation decision with separately budgeted next work.

## 10. Sources and assumptions

Primary pages checked during planning on 2026-10-19:

- [Open Images description/licensing](https://storage.googleapis.com/openimages/web/factsfigures_v7.html): annotations CC BY 4.0; listed image licenses require individual verification.
- [Open Images formats/downloads](https://storage.googleapis.com/openimages/web/download_v7.html): subset access and existing boxes/flags; no listing-policy ground truth.
- [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/): noncommercial restriction; no automatic commercial-R&D exemption assumed.
- [GitHub Pages overview](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages): static hosting capability. Limits page retrieval failed; exact eligibility/quotas/terms remain LC-021 gates.
- [MDN Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers): worker/message mechanism; target-browser behavior must be tested.
- [TypeScript license](https://github.com/microsoft/TypeScript/blob/main/LICENSE.txt) and [Vite license](https://github.com/vitejs/vite/blob/main/LICENSE): source-license evidence, not clearance of every installed dependency.

Unverified: demand, pricing, real-photo generalization, thresholds, device latency, hosting eligibility and pinned distribution/transitive licenses. No benchmark score, implemented capability, customer pilot, deployed demo or dataset clearance is claimed.
