# Machine Learning-Guided Solvent Optimization Using MACCS Keys Fingerprints

Code and data for the machine learning-guided solvent optimization described in:

> A. Kitamura, Y. Edo, Y. Kondo, H. Morimoto, T. Ohshima,
> *Machine Learning-Guided Solvent Optimization Using MACCS Keys Fingerprints:
> Application to an Sc(OTf)*₃*-Catalyzed Synthesis of N-Unsubstituted Ketimines*,
> **Synlett**, submitted.

## Overview

Solvents are described using only their molecular structure. Each solvent is
converted from a SMILES string into a 167-bit MACCS Keys fingerprint with RDKit,
and a partial least squares (PLS) regression model is trained on the
experimental yields of a small number of solvents. The trained model then ranks
all candidate solvents, including those that have not been examined
experimentally. No experimentally measured solvent properties (dielectric
constant, Kamlet–Taft parameters, donor number, etc.) are required.

```
data/solvent.csv                29 candidate solvents (name, SMILES, yield)
        |
        | keep only solvents with a measured yield (10 solvents)
        v
    SMILES strings
        | RDKit: SMILES -> molecule -> MACCS Keys (167 bits)
        v
    X  (10 x 167 binary matrix)
        | StandardScaler, then PLS regression (3 latent variables)
        v
    trained model
        | apply to the MACCS Keys of all 29 solvents
        v
output/pred_yield.csv           candidate solvents ranked by predicted yield
```

## Repository contents

| Path | Description |
|---|---|
| `0_main.ipynb` | Main pipeline: fingerprint generation, PLS training, prediction for all 29 solvents |
| `1_pls.ipynb` | PLS regression coefficients and score plot |
| `2_pca.ipynb` | Loading plot of the trained model |
| `3_nbo.ipynb` | Extraction of NBO charges and HOMO energies from Gaussian output; correlation with yield |
| `4_parse.ipynb` | Extraction of Cartesian coordinates and thermochemical corrections for the Supporting Information |
| `data/solvent.csv` | 29 candidate solvents with SMILES and, where available, experimental yields |
| `data/maccskeys_meaning.csv` | Substructure definition of each MACCS Keys bit |
| `qm_nbo_t6311++g/` | Gaussian output files (B3LYP/6-311++G(d,p)) |
| `output/` | Generated files (fingerprint table, trained model, predicted yields, NBO summary) |
| `solopt_ui/` | Web interface (FastAPI backend, React frontend) for running the workflow with MACCS Keys, ECFP, or RDKit 2D descriptors |

## Requirements

- Python 3.12
- RDKit 2026.03.5
- scikit-learn 1.7.2
- pandas, numpy, matplotlib, seaborn
- cclib (for parsing the Gaussian output in `3_nbo.ipynb` and `4_parse.ipynb`)

```bash
pip install rdkit scikit-learn pandas numpy matplotlib seaborn cclib
```

> `model.coef_` changed shape in scikit-learn 1.1. The notebooks assume
> `(n_targets, n_features)`, i.e. scikit-learn 1.1 or later.

## Usage

Run the notebooks in order. `0_main.ipynb` must be run first, since it writes
the trained model to `output/pls_model.npy`, which `1_pls.ipynb` and
`2_pca.ipynb` load. `3_nbo.ipynb` and `4_parse.ipynb` are independent and can be
run on their own.

To apply the workflow to a different reaction, replace `data/solvent.csv` with
your own solvent screening table. Only the `smiles` column and the measured
yields are required; rows with an empty yield are treated as candidates to be
predicted.

For the web interface:

```bash
cd solopt_ui
./run.sh
```

## Scope and limitations

The model is intended for **rank-ordering** candidate solvents prior to
experimental verification, not for the quantitative prediction of individual
yields. With ten training solvents, leave-one-out cross-validation gives
RMSE = 30.9% and *Q*² = 0.44 (three components), compared with 41.1% for a model
that always returns the mean observed yield. See the Supporting Information of
the paper for the full validation, including *y*-randomization.

Solvents whose distinguishing MACCS Keys bits are invariant across the training
set receive identical predictions. In the present dataset this affects four
pairs (1,2-dichlorobenzene / chlorobenzene, CHCl₃ / CCl₄, *o*- / *m*-xylene, and
DMF / DMAc). This is a consequence of the structural coverage of the training
set rather than a limitation of the fingerprint, and is removed by including a
solvent that carries the relevant bit.

## Citation

If you use this code, please cite both the paper and this repository.

## License

MIT License. See [LICENSE](LICENSE).
