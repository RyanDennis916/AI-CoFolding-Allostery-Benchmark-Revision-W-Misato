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
