# dynamic-object-removal (numpy-only)

Detector-free dynamic map cleaning via [dynamic-object-removal](https://github.com/rsasaki0109/dynamic-3d-object-removal) — `pip install` from PyPI, numpy only.

## Install

```bash
pip install "dynamic-object-removal>=0.5"
```

## Run

From this folder (`methods/dor_numpy/`):

```bash
python main.py --data_dir /path/to/00 --algorithm fusion      # highest accuracy
python main.py --data_dir /path/to/00 --algorithm range
python main.py --data_dir /path/to/00 --algorithm scan_ratio
python main.py --data_dir /path/to/00 --algorithm temporal
```

`fusion` is the slowest of the four: ~11 min on seq 00 and ~29 min on seq 05 with
the default `--fusion-workers 6`; the others run in a few minutes.

## Evaluate

Each command writes `dor_<algorithm>_output.pcd` into `data_dir`. Export and score
with the benchmark tools:

```bash
./build/export_eval_pcd /path/to/00 dor_fusion_output.pcd 0.05
python scripts/py/eval/evaluate_all.py
```

`evaluate_all.py` reads its `Result_Folder`, `algorithms`, and `all_seqs` settings
from the constants at the top of the file — add `dor_fusion` (or the algorithm you
ran) to the `algorithms` list before running it.

## Semantic-KITTI teaser results (seq 00 / 05)

| algorithm | seq 00 SA | seq 00 DA | seq 00 AA | seq 05 SA | seq 05 DA | seq 05 AA |
|---|---|---|---|---|---|---|
| fusion | 98.9 | 98.3 | **98.6** | 98.0 | 98.1 | **98.0** |
| range | 99.6 | 34.5 | 58.6 | 99.8 | 25.9 | 50.9 |
| scan_ratio | 98.0 | 92.8 | 95.4 | 96.0 | 97.9 | 96.9 |
| temporal | 97.0 | 46.6 | 67.2 | 97.3 | 25.9 | 50.2 |

`fusion` (library v0.5.0) OR-combines three evidence channels computed per scan
against the accumulated map: ray-sampled free-space carving with per-scan hit
precedence, DUFOMap-style eroded void confirmation (hit inflation + full
26-neighborhood erosion), and the `scan_ratio` votes at a stricter fraction. The
channels fail in complementary regimes — fractional free-space voting nails
transient traffic (seq 00), absolute void counts catch slow movers and late
leavers (seq 05) — so the union scores high on both.

`scan_ratio` normalizes votes per point: a map point is removed only when a majority
of the scans that actually revisit its polar column flag it as vacated (library default
since v0.4.0). Rarely-observed static points no longer accumulate spurious votes over
the sequence, which lifts SA to ~96-98% at near-unchanged DA.

Reproduce end-to-end (download + eval) from the upstream library repo:

```bash
git clone https://github.com/rsasaki0109/dynamic-3d-object-removal.git
python3 dynamic-3d-object-removal/scripts/run_dynamicmap_benchmark.py --sequences 00 05
```
