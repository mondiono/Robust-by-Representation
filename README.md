# Robust by Representation — code

Code for the paper *Robust by Representation: Ontology-Lifted Knowledge Graph Features and Certified
Description-Logic Rules for Malware Detection under Behaviour Substitution*. The dataset is included in `data/`.

**Short reproduction steps: see `REPRODUCE.md`.**

## Requirements
Python 3.10+, `numpy`, `pandas`, `scikit-learn`, `torch` (CPU is enough for 1k samples), `matplotlib`,
`owlready2`, and Java (for HermiT, used only by `verify.py`). See `requirements.txt`.

Set `PYTHONHASHSEED=0` for every run: Python's set ordering otherwise changes the edge order of the graph and
produces small numerical differences in the neural models.

## Data layout
The files in `data/`:
- `data/dataset.owl` — MAECO-Lite knowledge graph (OWL/XML, TBox merged)
- `data/examples.json` — `{"positive_examples": [...], "negative_examples": [...]}` (sample IDs)

## Pipeline (paper section in brackets)
```bash
export PYTHONHASHSEED=0
python build.py data                 # parse OWL; anonymous processes; process closure; self-loops removed   [Sec. 3]
python check1.py                     # technique provenance check; near-duplicate clusters -> groups.pkl      [Sec. 3]
python run4.py all                   # representation x model grid, grouped 5-fold CV, 3 perturbations        [Sec. 7.1-7.3]
python run4.py dynamic
python run6.py all                   # precision-guided covering on TRUE labels, 3 literal pools, theta sweep [Sec. 5, 7.4]
python run6.py dynamic
python rerank.py all 0.8 results6    # subsumption check + marginal-recall ranking of the certified rules      [Sec. 5.2, 7.5]
python rerank.py dynamic 0.8 results6
python eval_rules.py all             # saved rules on exactly the grid's perturbations (incl. swap-to-seen)  [Tab. 3-5]
python eval_rules.py dynamic
python make_assets.py ../overleaf    # all figures and table bodies of the paper
python verify_covering.py all 0 && python verify.py data/dataset.owl       # HermiT check of one fold's rules [Sec. 4.2]
```
On a 2-core CPU `run4.py` takes about 15 minutes per feature set; everything else takes seconds to a few minutes.

## Modules
- `lattice.py` — the concept lattice: concept families, membership by upward closure over the TBox (flat or closed),
  specificity, ancestor/parent maps, the family-top exclusion.
- `rules.py` — `perturb()`: sibling substitution (any sibling, or only siblings in a given pool); rule post-processing.
- `models.py` — R-GCN over TBox + ABox (schema→instance messages, inductive) and a neural-DNF rule head.
- `run_helpers.py` — helpers for the neural-DNF variant (used inside `run4.py`, not reported in the paper).
- `export_features.py` — writes the flat, full and coarse lattice features as CSV files for inspection.
- `run6.py` — the certified covering learner (`learn_rule`, `covering`) and its evaluation.

## Result files included
`results4_*.json` (grid), `results6_*.json` (covering), `eval_rules_*.json`, `rerank_results6_*.json`,
`ranked_fold0_*.json`, `groups.pkl` (near-duplicate cluster IDs). With these, `make_assets.py` reproduces the
paper's figures and tables without retraining.

## Scaling to 10k samples
The scripts make no assumption about dataset size. For ~10k samples run them on a GPU machine: move the R-GCN
to CUDA in `models.py` (`train_rgcn`, `rgcn_proba`), and rerun `check1.py`, since its near-duplicate search is
quadratic in the number of samples.
