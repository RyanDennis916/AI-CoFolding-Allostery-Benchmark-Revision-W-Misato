# MISATO input-provenance findings

Generated from the delivered output folders and the public MISATO source code.
This is an evidence log, not a statement of final benchmark results.

## Delivered-output audit

- EquiBind: 8,569 target directories; 8,565 contain the expected
  `lig_equibind_corrected.sdf`; four are empty (`1A37`, `2Z4W`, `3MBZ`,
  `4GD6`).
- DiffDock: 8,419 target directories; each has a canonical `rank1.sdf`.
  A further 84,936 rank/confidence SDFs are retained as unselected
  candidate/repeat-run evidence.
- Shared primary cohort: 8,379 PDB IDs after the three shared EquiBind
  absences are excluded with an explicit reason.
- All 8,379 paired primary ligands have the same heavy-element composition.
  8,261 have matching connectivity; 1,712 match exactly and 6,549 differ in
  stereochemical representation. The 118 remaining pairs parse as coordinate
  SDFs but do not yield an RDKit hydrogen-normalized graph descriptor.

The generated local manifests in `misato_output/inventory/` contain the full
per-ID disposition and SHA-256 record for every delivered SDF.

## Reference audit: why PDB alone is insufficient

The first 100 target IDs were resolved against RCSB biological assemblies:

- 68 had one single-component non-polymer candidate matching the predicted
  ligand's heavy-element signature;
- 2 had one composite candidate;
- 6 were ambiguous; and
- 24 had no matching component combination of up to three residue types.

For example, `11GS` has a 39-heavy-atom predicted ligand. Its experimental
structure contains separate 20-heavy-atom `GSH` and 19-heavy-atom `EAA`
components; their combined signature matches the prediction. This shows that
the results may represent a ligand assembly rather than one crystallographic
residue.

## What the original MISATO data provides

The public MISATO processing code represents every PDB ID as an HDF5 group
with `trajectory_coordinates`, `atoms_type`, `atoms_number`, `atoms_residue`,
`atoms_element`, and `molecules_begin_atom_index`. Its preprocessing selects
ligand atoms where `atoms_residue == 0`, falling back to the final molecule for
peptide ligands. This selection can naturally produce a multi-component ligand
assembly. See [preprocessing_db.py](https://github.com/t7morgen/misato-dataset/blob/master/src/data/processing/preprocessing_db.py).

This was verified against the repository's supplied `tiny_md.hdf5` sample.
For `10GS`, `11GS`, and `16PK`, the `atoms_residue == 0` atom selection has
the exact same heavy-element signature as the delivered EquiBind SDF. The
`11GS` selection contains 66 total atoms / 39 heavy atoms, consistent with the
combined `GSH` + `EAA` representation above. Each sample group has 100 stored
trajectory frames; the sample alone does not identify which frame Lucas used.

## Required before final scoring

We need the exact original inputs for the evaluated IDs, or an equivalent
reconstruction from the MD HDF5:

1. which trajectory frame(s) supplied each receptor and ligand;
2. the exact ligand atom-selection/preparation rule; and
3. the coordinate transform, if any, applied before EquiBind and DiffDock.

The full MISATO archive is not needed merely to inspect the schema. However,
the relevant per-ID HDF5 entries (plus topology/restart information if needed
to reconstruct the receptor) are needed to reproduce Lucas's inputs and to
compute protein-aligned pose, pocket, QS, and lDDT-PLI metrics honestly.

Until then, direct RMSD against a substituted RCSB receptor would be a
provisional, non-equivalent analysis and must not be reported as the original
MISATO docking benchmark.
