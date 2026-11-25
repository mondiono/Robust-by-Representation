# Reproducing the results — short steps

Run every command from this folder, one after the other, in the same terminal.

**0. Install** (once). Python 3.10+; Java is needed only for step 7.
```bash
pip install -r requirements.txt
```

**1. Fix Python's hash seed** (once per terminal; without it some numbers change slightly between runs).
```bash
export PYTHONHASHSEED=0
```

**2. Parse the ontology and clean the graph** → `data/clean.pkl`
```bash
python build.py data
```

**3. Data checks and near-duplicate clusters** → `groups.pkl`
```bash
python check1.py
```

**4. (Optional) Export the features for inspection** → `features/all/` and `features/dynamic/`
```bash
python export_features.py
python export_features.py --dynamic
```
Gives `flat.csv`, `full.csv`, `coarse.csv` (949 samples × concepts, 0/1) and `concepts.csv` (what each column means).

**5. Experiments.** 5a and 5b are independent and can run in parallel.
```bash
python run4.py all;  python run4.py dynamic        # 5a: representation x model grid (~15 min each on CPU)
python run6.py all;  python run6.py dynamic        # 5b: certified rules (theta sweep, three literal pools)
python rerank.py all 0.8 results6;  python rerank.py dynamic 0.8 results6   # needs 5b
python eval_rules.py all;  python eval_rules.py dynamic                     # needs 5b and rerank
```

**6. Rebuild the paper's tables and figures**
```bash
python make_assets.py <path-to-unzipped-overleaf-folder>
```
Writes `tables/*.tex`, `figures/*.pdf` and `numbers.json` into that folder.

**7. (Optional) Check the rules with the HermiT reasoner** (needs Java)
```bash
python verify_covering.py all 0
python verify.py data/dataset.owl
```
Every rule should print `MATCH`.

## Notes
- Steps 5–6 overwrite the result files shipped in this folder. To rebuild the tables without retraining,
  skip step 5 and run step 6 directly.
- With `PYTHONHASHSEED=0`, steps 5b–6 reproduce the paper's tables exactly (checked from a clean copy). The
  R-GCN grid (5a) is deterministic on CPU, but an exact second rerun was not checked.
- `data/` contains `dataset.owl` (MAECO-Lite knowledge graph, 1,000 samples), `examples.json` (labels) and
  `manifest.json`.
