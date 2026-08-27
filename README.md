# eeg-pain-sensitivity-classifier
EEG classification of pain sensitivity from laser-evoked potentials (OpenNeuro ds005284) — LDA/SVM with LOOCV and permutation testing on n=26 subjects
# EEG Pain Analysis — Laser-Evoked Potentials

A Python/MNE-Python pipeline for extracting cortical pain biomarkers from EEG, applied across all 26 subjects of [OpenNeuro ds005284](https://openneuro.org/datasets/ds005284) — a laser-evoked potential (LEP) pain study.

**Author:** Sai Sasi Sekhar Kongala — MSc Brain Sciences, University of Glasgow (2024–2025)

---

## Motivation

My MSc dissertation characterised cold-selective spinal projection neurons using patch-clamp electrophysiology — the ascending nociceptive pathway at the spinal level. This project asks the natural follow-up question: **do those spinal nociceptive circuits produce measurable cortical signatures in human EEG?**

Laser-evoked potentials are a well-established way to probe this. A brief nociceptive laser stimulus produces a characteristic cortical response — the N2 and P2 components — reflecting spino-thalamo-cortical transmission and cortical salience processing of pain, respectively. Alongside this, resting alpha oscillations are a known EEG biomarker of pain sensitisation. This project builds an independent Python pipeline to extract both, across the full 26-subject cohort, and checks the results against the published reference pipeline and literature.

---

## Data

**Dataset:** [ds005284](https://openneuro.org/datasets/ds005284) — 64-channel BioSemi EEG from 26 healthy adults (avg. age 21, 18 female), each receiving 16 fixed-intensity CO₂ laser stimuli to the dorsum of the hand, ~20 seconds apart. Sampled at 1024 Hz, BIDS-compliant, ~1.6 GB total.

**Citation:**
Xiangyue, Z. et al. *A comprehensive EEG dataset of laser-evoked potentials for pain research.* Scientific Data 12, 1536 (2025). [doi.org/10.1038/s41597-025-05900-1](https://doi.org/10.1038/s41597-025-05900-1)
Dataset DOI: [10.18112/openneuro.ds005284.v1.0.0](https://doi.org/10.18112/openneuro.ds005284.v1.0.0)

Raw EEG (`.bdf`) files are **not included in this repository** — download directly from OpenNeuro. The dataset is released under CC0 1.0 (public domain); attribution above is provided as good scientific practice, not a legal requirement.

---

## Pipeline

Each subject's raw `.bdf` recording goes through the same preprocessing chain before feature extraction:

1. Bad channel detection and removal (channel A2 — amplitude outlier, ~1593 µV)
2. Average reference applied
3. Bandpass filter, 1–40 Hz
4. Notch filter, 50 Hz (mains noise)

From the cleaned signal, two parallel analyses run:

- **Alpha band:** power spectral density computed, peak frequency and power extracted in the 8–12 Hz range
- **Laser-evoked potential:** stimulus onsets detected from the status channel (event ID 54), epoched -200 ms to +1000 ms around each of the 16 stimuli per subject, averaged, and the N2/P2 components extracted

Results for all 26 subjects are compiled into `group_results.csv`.

---

## Results

### Single subject (sub-001)

| | |
|---|---|
| Peak alpha frequency | 8.0 Hz (reference: ~10 Hz in healthy adults) |
| Alpha power | 1.4448 µV²/Hz |
| N2 latency / amplitude | 226.6 ms / 48.42 µV (reference: 150–300 ms) |
| P2 latency / amplitude | 460.0 ms / 153.42 µV (reference: 300–500 ms) |
| NP amplitude | 105.00 µV (reference: 50–150 µV) |

### Group (all 26 subjects)

| Metric | Mean ± SD | Reference range |
|---|---|---|
| N2 latency | 244.1 ± 64.8 ms | 150–300 ms |
| P2 latency | 381.4 ± 99.8 ms | 300–500 ms |
| NP amplitude | 24.5 ± 18.1 µV | 15–30 µV |
| Peak alpha | 8.5 ± 1.3 Hz | ~10 Hz (healthy) |

Group N2 and P2 latencies fall within published reference ranges for nociceptive laser stimulation. Peak alpha (8.5 Hz) sits below the typical healthy-adult reference of ~10 Hz — consistent with the pain-related alpha slowing reported by Furman et al. (2020, *Pain*).

Group distribution plots (N2, P2, alpha, NP across all subjects) are included as `06_group_distributions.png`.

---

## How This Compares to the Published Pipeline

The original paper (Xiangyue et al., 2025) used MATLAB/EEGLAB with a more extensive preprocessing pipeline than this independent Python implementation. The main differences, and why they matter:

| Step | This analysis | Published paper |
|---|---|---|
| Artifact rejection | Amplitude threshold only | Full ICA (`runica`) |
| Baseline window | -200 to 0 ms | -1000 to 0 ms |
| Epoch window | -200 to +1000 ms | -1000 to +2000 ms |
| Low-pass filter | 40 Hz | 100 Hz (30 Hz for display) |
| Peak channel | A1 (vertex-adjacent) | Cz (exact vertex) |

These differences plausibly explain the higher latency variability seen here relative to the published values. The N2 (244 ms) and P2 (381 ms) group means are directionally consistent with the LEP literature despite the simpler pipeline — but the gap is worth being upfront about rather than glossing over.

---

## Honest Limitations

- No ICA-based artifact rejection was applied — the published pipeline used EEGLAB's `runica`, which would likely reduce amplitude variability here.
- A1 was used as a Cz approximation rather than the exact vertex electrode, which affects amplitude accuracy.
- Automated peak detection hit boundary effects in some subjects (N2 pinned at exactly 150 or 300 ms, P2 at 250 or 500 ms) — a sign the automated peak wasn't clearly defined for those subjects. Manual verification would be needed before treating these as publication-grade values.
- Baseline window (-200 ms) is shorter than the published pipeline's (-1000 ms).
- This cohort received a single fixed stimulus intensity; the full released dataset spans a wider intensity range (2.5–4.5 J) not used here.

---

## Reproducing This

```bash
pip install mne numpy scipy pandas matplotlib jupyter openneuro-py
```

**Single subject (sub-001):**
1. Download `sub-001_task-26ByBiosemi_eeg.bdf` and `sub-001_task-26ByBiosemi_events.tsv` from [OpenNeuro](https://openneuro.org/datasets/ds005284)
2. Place them in `data/sub-001/eeg/`
3. Run `EEG_Pain_LEP_Analysis_OpenNeuro_ds005284.ipynb`

**Full group (all 26 subjects):**
1. Run the download cell in `02_Group_Analysis_26_Subjects.ipynb` — this fetches all 26 subjects automatically via `openneuro-py`
2. Run the rest of the notebook — results are saved to `group_results.csv`

---

## Repository Contents

```
├── EEG_Pain_LEP_Analysis_OpenNeuro_ds005284.ipynb   # single-subject pipeline (sub-001)
├── 02_Group_Analysis_26_Subjects.ipynb              # full 26-subject pipeline
├── group_results.csv                                # N2, P2, alpha per subject, all 26
├── figures/                                         # output plots (see below)
└── README.md
```

**Figures included:**
- `01_raw_psd.png` — raw power spectral density before filtering
- `02_filtering_comparison.png` — before vs. after filtering
- `03_alpha_band.png` — frequency bands with peak alpha annotated
- `04_LEP_waveform.png` — averaged laser-evoked potential waveform
- `05_N2_P2_components.png` — N2/P2 components annotated
- `06_group_distributions.png` — group-level distributions across all 26 subjects

Raw `.bdf` files are not included — see Data above.

---

## Next Steps

This pipeline was also the feature-extraction source for a downstream project: [eeg-pain-sensitivity-classifier](https://github.com/Sai-sasi/eeg-pain-sensitivity-classifier), which uses `group_results.csv` to test whether these EEG features can predict individual pain sensitivity.

Remaining improvements to this pipeline itself:
- Apply ICA-based artifact rejection
- Map to the exact Cz channel rather than the A1 approximation
- Manually verify N2/P2 peaks flagged at detection-window boundaries

---

## Environment

| Package | Version |
|---|---|
| Python | 3.x |
| MNE-Python | 1.2.3 |
| openneuro-py | 2026.4.1 |
| NumPy, pandas, SciPy, matplotlib | latest |

---

## Licence

Code in this repository is released under the MIT Licence. The underlying dataset is licensed CC0 1.0 Universal by its original authors — see [OpenNeuro](https://openneuro.org/datasets/ds005284) for terms.

---

## References

- Xiangyue, Z. et al. *A comprehensive EEG dataset of laser-evoked potentials for pain research.* Scientific Data 12, 1536 (2025). [doi.org/10.1038/s41597-025-05900-1](https://doi.org/10.1038/s41597-025-05900-1)
- Xiangyue, Z. et al. *26 By Biosemi.* OpenNeuro (2024). [doi.org/10.18112/openneuro.ds005284.v1.0.0](https://doi.org/10.18112/openneuro.ds005284.v1.0.0)
- Gramfort, A. et al. *MEG and EEG data analysis with MNE-Python.* Frontiers in Neuroscience 7, 267 (2013). [doi.org/10.3389/fnins.2013.00267](https://doi.org/10.3389/fnins.2013.00267)
- Markiewicz, C.J. et al. *The OpenNeuro resource for sharing of neuroscience data.* eLife 10, e71774 (2021). [doi.org/10.7554/eLife.71774](https://doi.org/10.7554/eLife.71774)
- Furman, A.J. et al. (2020). Pain-related modulation of alpha oscillations. *Pain.*
- Kongala, S.S.S. *Electrophysiological Characterisation of Calbindin- and Phox2a-Lineage Spinal Neurons in Response to Somatosensory Stimuli.* MSc Dissertation, University of Glasgow (2025). BBSRC-funded. [github.com/Sai-sasi/Electrophysiological-recordings](https://github.com/Sai-sasi/Electrophysiological-recordings)

---

## Related Repositories

- [Electrophysiological-recordings](https://github.com/Sai-sasi/Electrophysiological-recordings) — BBSRC-funded MSc dissertation, spinal pain circuit electrophysiology (patch-clamp, p=0.033)
- [Bioimaging-Colocalization-Analysis](https://github.com/Sai-sasi) — ImageJ/JACoP colocalization analysis, University of Glasgow BIOL5261 (Grade A5)
- [bioimaging-cell-counting-BBBC001](https://github.com/Sai-sasi) — automated cancer cell counting pipeline, CellProfiler 4, Pearson r=0.996 vs. published ground truth
- [eeg-pain-sensitivity-classifier](https://github.com/Sai-sasi/eeg-pain-sensitivity-classifier) — downstream ML project using this pipeline's output features

---

**Contact:** sasisasisekhark@gmail.com · [github.com/Sai-sasi](https://github.com/Sai-sasi) · Glasgow, UK

Part of an ongoing portfolio in computational pain neuroscience, building toward PhD research in translational neurophysiology.

