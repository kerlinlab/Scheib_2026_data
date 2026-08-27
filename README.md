# Scheib_2026_data

Raw and registered two-photon imaging data for the two example ROIs shown in **Figure 2** of
Scheib et al., 2026, from the Kerlin Lab at the University of Minnesota, Twin Cities.

This is a companion repository to the analysis repository
**[kerlin-lab/Scheib_2026](https://github.com/kerlin-lab/Scheib_2026)**, which holds the processed
per-animal data, the shared analysis library and the six figure notebooks. Nothing here is required
to run those notebooks — `Fig2.ipynb` reproduces its panels from small, pre-extracted exports that
ship inside the analysis repo. This repository exists so that the *true raw* data behind those two
example panels, and the registered movies derived from it, are available for inspection and
re-derivation.

Because the complete raw dataset is **20.07 TB** across 22 animals and 167 imaging sessions, only
these two sessions are distributed here. The remainder is available upon request (see
[Requesting the rest of the raw data](#requesting-the-rest-of-the-raw-data)).

---

## Citation

[Scheib et al., 2026. Distinct sensorimotor encoding in tuft dendrites and somata associated with
action, correction, and learning. eLife.](https://elifesciences.org/reviewed-preprints/111876)

Please cite the manuscript if you use these data. If you use the analysis code as well, cite the
[Scheib_2026](https://github.com/kerlin-lab/Scheib_2026) repository alongside it.

---

## Repository structure

```
Scheib_2026_data/
└── data/
    └── B00002213784/                  # animal ID (barcode-style, same IDs as the analysis repo)
        ├── 230123/                    # session date, YYMMDD — DENDRITE example (Fig2)
        │   └── run1/
        │       ├── raw/               # unmodified ScanImage .tif stacks as acquired
        │       └── raw_reg/           # motion-registered stacks (SLeD pipeline output)
        └── 230124/                    # session date, YYMMDD — SOMA example (Fig2)
            └── run1/
                ├── raw/
                └── raw_reg/
```

The path convention is `data/<animal ID>/<YYMMDD>/run<N>/<raw | raw_reg>`, matching the layout of
the lab's acquisition server, so paths recorded in the analysis repo's `summaryInfo`
(`imagingRawPath`) resolve against this tree with only the root swapped.

Both example sessions come from the **same animal**, `B00002213784`, on consecutive days.

The raw bpod behavior .mat file is also included for each trial in the raw/ directory.

### The two sessions

| | Dendrite example | Soma example |
|---|---|---|
| Path | `data/B00002213784/230123/run1/` | `data/B00002213784/230124/run1/` |
| Animal | `B00002213784` | `B00002213784` |
| Date | 2023-01-23 | 2023-01-24 |
| Day relative to sensorimotor shift | −6 (pre-shift) | −5 (pre-shift) |
| Compartment / plane label | `L1_Dend` (layer 1 tuft dendrites) | `L5_Soma` (layer 5 somata) |
| Session index | 0 | 0 |
| Example ROI | 4 | 33 |
| Example trials (0-indexed) | 21 (correct-left), 22 (correct-right) | 270 (correct-right), 271 (correct-left) |
| Example trials (File #) | 22 (correct-left), 23 (correct-right) | 271 (correct-right), 272 (correct-left) |
| Frame size | 512 × 512 px | 256 × 256 px |
| Pixel size | 0.4104 µm/px (≈ 210 µm FOV) | 1.2825 µm/px (≈ 328 µm FOV) |
| Frame rate | 44.6 Hz | 22.8 Hz |

Trial numbering, ROI indices, day-relative-to-shift labels and the CL/CR trial-type codes are the
same ones used throughout the analysis repo, so an ROI or trial identified here can be looked up
directly in `masterData`.

---

## `raw/`

Unmodified **ScanImage `.tif` stacks** exactly as written at acquisition — no motion correction, no
cropping, no rescaling, no re-encoding. Each file carries ScanImage's own metadata header, which is
the authoritative record of acquisition settings (frame rate, zoom, PMT gains, channel
configuration, frame/volume counts). Read them with any TIFF reader that preserves the header:

```python
import tifffile as tf

with tf.TiffFile(path) as tif:
    frames   = tif.asarray()                # (frames, Y, X)
    metadata = tif.scanimage_metadata       # acquisition settings as recorded
```

`tifffile` is already pinned in the analysis repo's `requirements.txt` / `Scheib2026_env.yml`, so an
environment built for `Scheib_2026` reads these files without anything extra.

Stacks are split across multiple `.tif` files per session in acquisition order — a session is the
concatenation of its files in filename order, not any single file.

## `raw_reg/`

The same imaging data after **rigid registration output. These are the registered movies every downstream step in the manuscript is
computed from: DeepInterpolation, non-rigid registration, ROI segmentation, NMF trace extraction, ΔF/F, baseline correction, deconvolution, and
the frames `Fig2.ipynb` renders.

`raw_reg/` is a strict transform of `raw/` — same frames, same order, same dimensions, shifted into
alignment.

---

## How this maps onto `Fig2.ipynb`

`Fig2.ipynb` in the analysis repo never touches this repository. It loads three small files per
example ROI from `Scheib_2026/Data/examples/fig2_indiv_examples/`:

| Analysis-repo file prefix | Derived from |
|---|---|
| `B00002213784_230123_Day-6_Sess0_L1_Dend_4_Tr_21CL_22CR_*` | `data/B00002213784/230123/run1/` (this repo) |
| `B00002213784_230124_Day-5_Sess0_L5_Soma_33_Tr_270CR_271CL_*` | `data/B00002213784/230124/run1/` (this repo) |

Those exports are deliberately trimmed: the `_imageData_den` / `_imageData_den_df` files keep only
the **three** movie frames each panel actually draws (the min / median / max activity frames — 3 ×
512 × 512 for the dendrite example out of 1,214 frames; 3 × 256 × 256 for the soma example out of
617), stored as `float16` and scattered back into full-shape arrays at load time. The trace arrays
are likewise trimmed to the ROIs and frames the panels read.

That trimming is what makes `Fig2.ipynb` runnable from a repository that fits on a laptop. This
repository is the untrimmed source those exports were cut from — use it if you want the full movies,
the frames that were dropped, other ROIs in the same field of view, or the original ScanImage
metadata.

---

## Getting the data (Git LFS)

The `.tif` stacks are stored with **[Git LFS](https://git-lfs.com)**. Git LFS must be installed
*before* cloning, or you will get small text pointer files instead of image data.

```bash
# 1. install Git LFS (once per machine)
git lfs install

# 2. clone
git clone https://github.com/kerlinlab/Scheib_2026_data.git
cd Scheib_2026_data
```

### Fetching only part of it

Both sessions together are large. To inspect the tree first and pull only what you need, skip the
LFS download during clone and fetch selectively afterwards:

```bash
# clone pointers only — fast, small
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/kerlinlab/Scheib_2026_data.git
cd Scheib_2026_data

# see what is there, and how big each file is
git lfs ls-files -s

# then pull just one session, or just the registered data
git lfs pull --include="data/B00002213784/230123/run1/raw_reg/**"
```

On Windows PowerShell, set the environment variable separately:

```powershell
$env:GIT_LFS_SKIP_SMUDGE = "1"
git clone https://github.com/kerlinlab/Scheib_2026_data.git
```

### Verifying a checkout

A file that came down as a pointer rather than as real data is a few hundred bytes and begins with
`version https://git-lfs.github.com/spec/v1`. If `tifffile` reports a file as not a TIFF, check that
first — it is almost always an LFS pointer, fixed with `git lfs pull`.

---

## Requesting the rest of the raw data

The full raw dataset — 22 animals, 167 imaging sessions, 336 raw directories, 147,203 files,
**20.07 TB** (11.33 TB imaging + 8.74 TB behavior and camera data) — is far past what any repository
can host. The per-animal inventory is tabulated in the
[Scheib_2026 README](https://github.com/kerlin-lab/Scheib_2026#raw-data-inventory).

For access to sessions beyond the two published here, contact newmanza@umn.edu. 
Method of transfer will be arranged case by case depending on how much data is involved.

---

## Notes

Nothing in this repository is modified from what the acquisition and registration pipelines wrote —
these are the files as produced, only relocated into the tree above.

Please note that a few of the trials are split between two files (ex. run1_00139_00001.tif and run1_00139_00002.tif)
This was due to a triggering artifact but does not affect the data, those files can be merged into a single contiguous trial.

SLeD refers to the lab-specific environment (Synapses, Learning and Dendrites). It is not required
to read anything here; `tifffile` is sufficient.

The preparation of this README was performed with assistance from Claude claude-opus-5. The data,
acquisition and processing pipelines are the work of Zachary Newman, Jackson Scheib and Aaron
Kerlin.

Please send any questions to newmanza@umn.edu.

## License

Released under the [MIT License](LICENSE), matching the
[Scheib_2026](https://github.com/kerlin-lab/Scheib_2026) analysis repository.
