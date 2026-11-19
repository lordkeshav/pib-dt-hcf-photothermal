# pib-dt-hcf-photothermal
Code and CSV datasets for a physics-informed Bayesian digital twin (PIB-DT) that separates gas-concentration changes from offset and pump-gain faults in hollow-core fiber photothermal spectroscopy, abstains on compound faults, and reports calibrated concentration intervals.
What the method does
In hollow-core fiber (HCF) photothermal spectroscopy, a change in the measured phase can be caused by:
a real change in gas concentration,
a common-mode interferometric offset, or
a loss of effective pump power (gain).
With one phase channel these causes look the same. PIB-DT adds an off-resonance phase reference and a pump-power monitor, and then:
computes a correlated-noise contrast between two time windows;
compares Bayesian evidence for no change, gas, offset and gain, using physically derived signature directions (with an exact Gauss–Hermite likelihood for the gain fault);
abstains ("unmodelled") when no single cause explains the change, using a residual adequacy gate;
reports the concentration with an interval that includes detector, monitor and calibration uncertainty.
Key results ( benchmark, 700 episodes, seed 20261025)
Result	Value
Single-cause accuracy, exact evidence	560/560 (100%; 95% Wilson CI 99.32–100%)
Single-cause accuracy, original linearized evidence	548/560 (97.86%)
Compound gas + gain episodes rejected as unmodelled	140/140 (GLR and trained classifiers: 0/140)
Concentration RMSE: PIB-DT vs differential-only	1.182 ppm vs 7.691 ppm
Coverage of nominal 95% intervals: PIB-DT vs differential-only	93.86% vs 42.71%
Coverage over 30 independent seeds	94.76 ± 0.92%
GLR and trained classifiers that use the same three channels also reach 100% on single-cause episodes. What only PIB-DT adds is abstention on compound faults and calibrated intervals. See the manuscript for limitations.
Repository structure
```
pib-dt-hcf-photothermal/
├── README.md
├── requirements.txt
├── LICENSE
├── CITATION.cff
├── .gitignore
├── src/
│   ├── simulate\_original.py   # original benchmark script (only the output folder was changed)
│   ├── pibdt.py               # simulation and inference engine (reproduces the original exactly)
│   ├── experiments.py         # all experiments E1–E11 -> data/results/\*.csv
│   └── figures.py             # Figs. 1–12 from the CSV files -> figures/
└── data/
    ├── README.md              # data dictionary for every CSV file
    ├── original/              # outputs of the original benchmark
    └── results/               # outputs of the extended experiments
```
The `figures/` folder is not stored in the repository; `src/figures.py` regenerates it from the CSV files.
Installation
Requires Python 3.10 or later.
```bash

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\\Scripts\\activate
pip install -r requirements.txt
```
Usage
Run every command from the repository root.
1. Regenerate the figures from the stored CSV files (about 2 minutes)
```bash
python src/figures.py
```
Writes `figures/Fig01\_architecture.png` … `Fig12\_latency\_trace.png` (600 dpi) and matching vector PDFs.
2. Reproduce the original benchmark (a few seconds)
```bash
python src/simulate\_original.py
```
Writes to `data/original\_reproduced/`. The CSV files should be byte-identical to `data/original/`:
```bash
cmp data/original\_reproduced/monte\_carlo\_trials.csv data/original/monte\_carlo\_trials.csv
```
3. Re-run all extended experiments (about 10 minutes on 2 CPU cores)
```bash
python src/experiments.py
```
This overwrites `data/results/\*.csv` (and writes a `summary.json`, which is git-ignored).
Experiments
ID	Experiment	Output file(s)
E1	Main benchmark: exact vs linearized evidence, GLR and ML baselines, CIs, McNemar	`E1\_main\_\*`, `T\_classification\_with\_CI`, `T\_mcnemar`, `T\_quantification\_with\_CI`, `T\_scenario\_with\_CI`, `T\_reliability`, `T\_confusion`, `T\_joint\_labels`, `T\_statistics`
E2	Replication over 30 seeds	`E2\_seed\_replication`
E3	Ablations	`T\_ablation`
E4	Fault-amplitude / SNR sweep	`E4\_amplitude\_sweep`
E5	Window length and noise sweeps	`E5\_window\_noise`
E6	Prior sensitivity	`E6\_prior\_sensitivity`
E7	Gate operating characteristic; compound-fault amplitude sweep	`E7\_gate\_oc`, `E7\_joint\_escape`
E8	Calibration-prior mismatch; temperature-model stress tests	`E8\_calibration\_mismatch`, `E8\_temperature`
E9	Out-of-distribution amplitudes for trained classifiers	`E9\_ml\_ood`
E10	Online (rolling) detection delay	`E10\_latency`
E11	Run time per decision	`T\_timing`
Column definitions are in `data/README.md`.
Reproducibility
Item	Value
Main seed	20261025 (bootstrap 20261026)
Replication seeds	20270001–20270030
ML training seed	11 (2000 episodes)
Tested environment	Python 3.13.18, NumPy 2.5.3, SciPy 1.18.1, pandas 3.0.5, scikit-learn 1.9.1, Matplotlib 3.11.2
Results of `experiments.py` were produced with the versions above. Other library versions may change the trained-classifier and timing results slightly; the model-based results should be unchanged.
Using the engine in your own code
```python
import sys; sys.path.insert(0, "src")
from pibdt import Cfg, run\_benchmark, class\_metrics, quant\_metrics

cfg = Cfg(evidence="exact")              # exact Gauss–Hermite gain evidence
df = run\_benchmark(cfg, seed=20261025)   # 700 episodes, one row per episode
print(class\_metrics(df, "bayes"))        # accuracy, macro-F1, false alarm, compound rejection
print(quant\_metrics(df, "twin"))         # RMSE, MAE, coverage, interval width
```
`Cfg` switches include `evidence` (`"linear"`/`"exact"`), `cov` (`"full"`/`"diag"`), `channels` (`"all"`/`"no\_ref"`/`"no\_pump"`), `temp\_norm`, `gate`, `gate\_alpha`, `noise\_mult`, `W`, `prior`, `scales`, `cal\_sd\_true`, `cal\_sd\_model`, `amp\_scale`, `temp\_amp`, `temp\_meas\_sd` and `ELOW\_model`.
Citation
If you use this code or data, please cite the manuscript (details will be added on publication) and this repository (see `CITATION.cff`).
License
Code and data are released under the MIT License (see `LICENSE`).
