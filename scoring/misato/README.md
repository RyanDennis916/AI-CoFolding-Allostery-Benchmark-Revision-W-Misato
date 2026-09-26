# MISATO result intake

`create_manifest.py` inventories the supplied EquiBind and DiffDock trees without changing them. It writes a SHA-256 raw-file manifest, a per-target cohort manifest, and an inventory summary.

Only `lig_equibind_corrected.sdf` (EquiBind) and `rank1.sdf` (DiffDock) are primary candidates. Other DiffDock `rank*_confidence-*.sdf` files are retained as repeat-run evidence, never silently pooled into the primary result.

```powershell
python scoring/misato/create_manifest.py `
  --equibind-root 'C:\Users\Ryan\Downloads\misato_equibind_results\nfshome\turano\Datasets\misato\equibind_output' `
  --diffdock-root 'C:\Users\Ryan\Downloads\MisatoDiffdockResults\MisatoDiffdockResults' `
  --out-dir misato_output\inventory
```

The delivered files are ligand-only. This tool deliberately does not calculate RMSD: protein-aligned docking metrics require validation of the receptor/reference coordinate frame and ligand identity.

## Compare target IDs with the published MISATO MD splits

`compare_published_splits.py` compares the target manifest against Zenodo's
`train_MD.txt`, `val_MD.txt`, and `test_MD.txt`. It downloads only these three
small text files, verifies their published MD5 checksums, and saves them with
the outputs so the audit can be repeated offline. It never reads `MD.hdf5`.
Lucas reports that these splits were **not** used to select docking inputs:
his ligands came from QM coordinates and the proteins were static RCSB
structures. This script therefore reports MD-split overlap only, not the
actual filtering rate or denominator for his runs.

```powershell
python scoring/misato/compare_published_splits.py `
  --target-manifest misato_output/inventory/target_manifest.csv `
  --out-dir misato_output/split_comparison
```

To rerun without network access, add
`--split-dir misato_output/split_comparison/published_lists`.
The script writes `target_comparison.csv` (one row per ID in either source),
`split_summary.csv`, and `summary.json`. `paired_primary_candidate` means both
expected ligand files are present; it does **not** certify molecular identity,
MD availability, or docking accuracy. Targets outside the published split lists
are retained and labeled, not silently discarded. An absent target tells us
which published MD IDs were not delivered, but not why Lucas excluded them.

`audit_references.py` performs that next, staged check against RCSB experimental
structures. Start with a bounded sample; it caches references locally and
identifies non-polymer residue candidates whose heavy-element signature matches
the paired predictions.

```powershell
python scoring/misato/audit_references.py `
  --identity-csv misato_output\inventory\primary_ligand_identity.csv `
  --cache-dir misato_output\references `
  --out-csv misato_output\inventory\reference_audit_sample.csv `
  --limit 100
```

`audit_hdf5_entries.py` is an optional audit of the original MISATO MD HDF5,
not part of reproducing Lucas's static-receptor docking inputs. It reads only
metadata for requested target groups (not all trajectory coordinates) and
reports the ligand-selection rule used by the public MISATO preprocessing
code. Run it in an environment with `h5py` and `numpy`:

```powershell
python scoring/misato/audit_hdf5_entries.py `
  --h5 'D:\MISATO\MD.hdf5' `
  --target-manifest misato_output\inventory\target_manifest.csv `
  --out-csv misato_output\inventory\misato_hdf5_audit.csv
```
