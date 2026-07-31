# Data manifest

The calculation requires the following input files, which are included under `data/` with the paths shown below before
running the code in [CT Exciton Final Code v8.nb](CT%20Exciton%20Final%20Code%20v8.nb).

The notebook and the Wolfram Language modules in `src/` all use the same top-level `data/` layout.

## 1. Molecular dynamics frame

| File | Location | Format | Purpose |
|---|---|---|---|
| `npt.gro` | `data/npt.gro` | GROMACS `.gro` | One equilibrated frame of the amorphous P3HT:IDTBR blend (128 P3HT chains × 24 monomers + 300 O-IDTBR molecules; coordinates in nm). See paper §2.3. |

The atom-index slicing in `src/01_import_configuration.wl` is hard-wired to the
specific atom count/order of this frame.

## 2. Machine-learning interchain hopping

| File | Location | Format | Purpose |
|---|---|---|---|
| `predictedt600.csv` | `data/predictedt600.csv` | CSV, one value per row | ML-predicted interchain hopping matrix element `t_ab` (eV) for each of the 237 close contacts (Fig. 6). |

`src/04_interchain_hopping_ML.wl` also **exports** the six geometric features
per contact to `results/SixParametersCTexciton1.txt`, which is the input to the
external ML model that produced `predictedt600.csv`. The model used for this
prediction is the one described in citation 46 of the manuscript.

## 3. DFT energies (intrachain hopping)

All DFT calculations use B3LYP/6-311g(d) (paper §2.3). Each file contains one
energy (Hartree) per row, one row per dimer configuration. Place them under:

```
data/energies_extracted/
├── ThDimer_gsE.txt
├── RhBT1_gsE.txt
├── RhBT2_gsE.txt
├── ThBT1_gsE.txt
├── ThBT2_gsE.txt
└── Unrestricted/
    ├── Cation_ThDimer.txt
    ├── Anion_ThDimer.txt
    ├── cations/
    │   ├── RhBT1_cationU.txt
    │   ├── RhBT2_cationU.txt
    │   ├── ThBTD1_cationU.txt
    │   └── ThBTD2_cationU.txt
    └── anions/
        ├── RhBT1_anionU.txt
        ├── RhBT2_anionU.txt
        ├── ThBTD1_anionU.txt
        └── ThBTD2_anionU.txt
```

| File stem | Dimer type | Used for |
|---|---|---|
| `ThDimer` | thiophene–thiophene (P3HT backbone) | `t_Th` (Fig. 4) |
| `RhBT1` / `RhBT2` | rhodanine–BT (left / right of IDTBR) | `t_BTRh` (Fig. 5b,d) |
| `ThBT1` / `ThBT2` | thiophene–BT (left / right of IDTBR) | `t_ThBT` (Fig. 5a,c) |

The phenylene–thiophene coupling of the rigid IDT core is computed from planar
optimized geometries and is hardcoded in `src/05_intrachain_hopping_DFT.wl`
(`pOptAnion`, `pOptCation`).

## Notes

- Monomer cation/anion/ground-state energies (Table 1) are embedded as literal
  constants in `src/05_intrachain_hopping_DFT.wl`.
- The ML model itself is not part of this repository; only its pre-computed
  output (`predictedt600.csv`) is consumed.
