# Wearable Food and Eating Detection (Prototype)

An early-stage research prototype exploring whether a camera mounted on wearable glasses can detect eating-related actions. The current firmware experiments use the Seeed Studio XIAO ESP32-S3 family and Edge Impulse image-classification models. The intended longer-term product is automatic food logging, but the phone application, nutrition analysis, and end-to-end logging pipeline are not implemented in this workspace.

> **Status:** experimental and incomplete. Initial hardware testing has been performed with a XIAO ESP32-S3 board taped to an ordinary pair of glasses. Performance ranged from moderate to very poor in real-life scenarios, and substantial improvement is needed. See [the testing notes](docs/testing.md) for the known limitations and remaining evaluation details.

## What is in this repository

- `Software/Arduino code/claude/consumption_detection/consumption_detection.ino` — current-looking inference experiment. It captures QVGA JPEG frames, converts them to RGB888 for Edge Impulse inference, reads `food_in_mouth` and `approaching_mouth` scores, and passes them to the event detector.
- `Software/Arduino code/claude/consumption_detection/ConsumptionDetector.h` — fixed-memory temporal post-processing. It blends class scores, calculates adaptive thresholds over a rolling window, applies debounce, and groups nearby detections into a consumption bout.
- `Software/Arduino code/` — other camera, inference, and SD-card experiments. These are exploratory sketches; their relative maturity and compatibility are not yet documented.
- `Eating detection/` — image/video dataset experiments and categorized samples. Labels include looking at food, holding food, bringing food toward the mouth, food in the mouth, ambiguous interactions, and hard negatives such as walking, talking, driving, or using a phone.
- `models/` — exported Edge Impulse Arduino model archives from several iterations. `models/note.txt` says a second model over the first model's outputs may be used to decide whether eating is actually occurring.
- `archive/technical-brief.md` and `archive/product-overview.md` — architecture and product concept notes. These describe future intent and should not be read as a description of features already built.
- `docs/testing.md` — current hardware/model testing status, known challenges, and suggested next evaluation steps.
- `logs and temporial pattern/` — exploratory activity notes and archives.

## Current firmware behavior

The consumption detection sketch initializes the XIAO camera and runs an Edge Impulse classifier repeatedly. It looks up the `food_in_mouth` and `approaching_mouth` labels by name, combines their probabilities, and feeds the score to `ConsumptionDetector`. The detector uses a rolling window and adaptive thresholds, requires consecutive frames to enter or leave an event, drops very short events, caps event duration, and merges nearby events. When a bout closes, the sketch prints its timing, peak score, frame count, and sub-event count to Serial.

This is a prototype event detector. It does **not** currently capture a high-resolution meal photo on a trigger, send it to a phone, use Bluetooth to transfer data, call a nutrition API, or save meal records. The current sketch configures QVGA JPEG capture and RGB conversion for inference; the privacy-oriented low-resolution grayscale monitoring and triggered high-quality capture described in the technical brief are design goals, not demonstrated behavior in this sketch.

## Hardware and build notes

Hardware testing has been done with the Seeed Studio XIAO ESP32-S3 identified by the [Robu product listing](https://robu.in/product/seeed-studio-xiao-esp32s3-2-4ghz-wifi-ble-5-0/), taped to an ordinary pair of glasses. The main inference sketch is configured for the XIAO ESP32-S3 camera pinout. The exact camera module/board configuration used in the test should be recorded: Seeed distinguishes the XIAO ESP32-S3 from the XIAO ESP32-S3 Sense, which includes the camera and microphone hardware. Seeed's [XIAO ESP32-S3 getting-started guide](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/) describes the variants.

When importing an Edge Impulse Arduino library, start with its included example sketch. The required Arduino settings include enabling PSRAM and selecting a suitable flash partition scheme; the best combination depends on the exported model and available application memory. The project has found that a balanced configuration works well, but exact tested settings and version numbers still need to be captured. The sketch comments mention a stable ESP32 Arduino core such as 2.0.11; verify compatibility with the specific Edge Impulse export rather than treating that note as a guaranteed requirement.

The sketch includes `consumption_detection_inferencing.h` and Edge Impulse SDK headers that are supplied by an exported Edge Impulse Arduino library. They are not present alongside the sketch in the listed source tree. To reproduce a build, identify the matching model export, install/import its Arduino library, and confirm the target labels include `food_in_mouth` and `approaching_mouth`. Hardware has been tested, but the firmware/model pairing and quantitative results are not yet recorded well enough to claim reproducible performance.

## Dataset labels

The local label note describes:

| Label | Intended meaning |
| --- | --- |
| `l1` | Inspecting/looking at food without touching it |
| `l2` | Holding food |
| `l3` | Food approaching the mouth |
| `l4` | Food in the mouth |
| `l5` | Ambiguous interactions that can resemble eating or food handling |
| `l6` | Hard negatives such as walking, talking, driving, and phone use |

These are dataset collection labels, not necessarily the names emitted by every exported model. Check each model's label list before using it with the firmware. Dataset rights, participant consent, and redistribution permissions should be confirmed before publishing images or videos.

## Roadmap / known gaps

- Establish one reproducible firmware and model pairing; document exact board/core/library versions and tested build steps.
- Measure precision, recall, false triggers, latency, frame rate, memory use, and battery life on the actual wearable setup. Initial testing indicates real-world performance is currently moderate to poor; class similarity and imperfect data quality are known challenges.
- Test whether the temporal image classifier plus event logic is enough, or whether an additional classifier or sensor is needed.
- Implement and validate a deliberate capture/transfer flow, including user-visible recording state and privacy controls.
- Build the companion app and nutrition estimation/logging only after the detection pipeline is validated.
- Add model provenance, dataset description, evaluation splits/metrics, and licensing/consent information.

## GitHub publishing guidance

Keep the repository focused on reproducible source, documentation, and artifacts you have permission to share. A practical initial layout would keep the primary sketch and header in a clearly named firmware directory, include this README and a license, and add a dataset/model README describing provenance and exact versions.

Review before publishing:

- **Do not commit the bundled Arduino IDE** under `Software/arduino-ide_2.3.10_Windows_64bit/`, IDE installers, shortcuts, or unrelated utilities. Document the required IDE instead.
- **Avoid committing all raw footage by default.** Videos and image datasets can be large and may contain identifiable people or private environments. Confirm consent, ownership, and redistribution rights; publish only a carefully selected, consented, appropriately anonymized subset if needed. Consider Git LFS or an external dataset host for large permitted data.
- **Model archives:** include only the selected, redistributable model export needed to reproduce the firmware. Multiple historical exports add substantial size and ambiguity; retain them only if their provenance and purpose are documented. Check Edge Impulse export terms and model/data rights.
- **Archives and copied examples:** `archive/` includes reference material and prototype copies. Keep only material that is relevant and whose license permits redistribution; preserve required notices for third-party code.
- **Personal activity logs and temporary files:** review `logs and temporial pattern/`, `archive/clipboard.txt`, and shell-script notes for private or irrelevant content before publishing.
- **Build outputs and secrets:** exclude generated binaries, caches, local IDE settings, credentials, Wi-Fi passwords, API keys, and personal data. Keep secrets out of source history, not merely out of the latest commit.

Suggested files to add as the project matures: `LICENSE`, `.gitignore`, `docs/dataset.md`, `docs/hardware.md`, `docs/model-evaluation.md`, and a versioned dependency/build guide. Choose a license only after confirming that the code, model exports, and dataset can each be licensed for public use.

## Project status

This repository captures exploration toward AI-assisted food/eating detection using wearable camera glasses and the XIAO ESP32-S3. It is not a finished calorie tracker, medical device, or validated nutrition system. Nutrition estimates, if later added, should be presented as estimates and not professional medical advice.
