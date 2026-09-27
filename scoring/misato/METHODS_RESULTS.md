# MISATO delivered-run analysis: Methods and results record

This record applies to **Lucas's delivered DiffDock rank-1 and EquiBind
corrected ligand poses**, not to every attempted run or to the complete
MISATO MD data. It was checked against
`misato_output/methods_accounting_v7/` and
`misato_output/score_summary_heavy_selected_v2/` on 25 September 2026.
Those local, Git-ignored directories contain the machine-readable per-ID
accounting, full scores, paired table, and RMSD quality-control list.
`METHODS_AND_RUN_PROTOCOL.md` defines the reference and scoring rules.

## Dataset and run completion

The checksum-verified public `QM.hdf5` contains **19,413 ID groups**. Every
delivered target ID occurs in it. The delivered folders contain 8,419
DiffDock primary poses and 8,565 EquiBind primary poses, spanning 8,606
distinct IDs: 8,379 with both poses, 40 DiffDock-only, and 186
EquiBind-only. Thus only 43.4% and 44.1% of the QM IDs, respectively, have
a delivered primary pose. The exact successful IDs and each method's score
status are in the tracked, path-free
[`delivered_cohort_status.csv`](delivered_cohort_status.csv); the richer
local inventory remains under `misato_output/inventory/`. Neither method's
successful outputs cover the entire QM set. All 16,984 primary poses match their corresponding QM
ligand's heavy-element composition, atom order, and untyped connectivity.
This does not verify their bond orders, stereochemistry, protonation, or
Lucas's starting conformer.

Lucas reported preparing 19,392 ligand SDFs from those 19,413 QM IDs (21
sanitization failures) and saving 14,057 RCSB proteins (5,356 missing).
He reported 14,037 DiffDock input pairs, 8,419 poses and 5,610
RDKit-unreadable SDF skips; these last two counts leave **eight input pairs
unexplained**. He reported 8,565 EquiBind poses and 10,827 failures, of
which 5,354 were attributed to missing proteins and 5,473 have no finer
verified status. These preparation, submission, and failure counts are
**collaborator-reported**, not reconstructed from input files or logs.
Consequently, the delivered successful subset is known exactly, but the
complete attempted cohort and reasons for all absent outputs are not.
The MISATO paper's 19,443 starting PDBbind structures are a different
processing stage from the 19,413 verified QM groups. The 16,972 official
MD-split IDs are not a docking denominator; 7,482 delivered IDs overlap
them, while 1,124 do not. Lucas reports using static RCSB receptors and one
QM-derived ligand conformer rather than MD frames.

## Crystal reference, scoring, and denominators

Only an unambiguous, single native ligand with a successful independent
heavy-atom graph and coordinate audit is eligible for the conservative
comparison. This gives 3,982 individually eligible DiffDock poses, 3,963
EquiBind poses, and **3,953 paired IDs** before OpenStructure scoring.
For 925 of those pairs the two predicted SDFs have the same exact ligand
graph/stereochemical representation; 2,979 pairs differ in stereochemical
representation and 49 require other representation review. The 925 form the
conservative paired subset, not a proof of matching *native-crystal*
stereochemistry. The 3,953 form a broader sensitivity cohort. Multi-copy
native ligands selected using a prediction, composite references, unmatched
ligands, and failed graph checks are excluded from these comparisons.

The reference-selection audit partitions each method's delivered poses as
follows. Selection categories are mutually exclusive; the later graph check
reduces the single-candidate counts to the strict eligible counts above.

| Reference-selection category | DiffDock | EquiBind |
| --- | ---: | ---: |
| Single candidate before graph check | 4,007 | 3,990 |
| Copy chosen by DiffDock proximity (excluded) | 2,753 | 2,743 |
| Copy chosen by EquiBind proximity (excluded) | 0 | 61 |
| Ambiguous crystal copies | 110 | 201 |
| Composite reference requiring review | 131 | 131 |
| No native heavy-element match | 1,418 | 1,439 |

The 25 DiffDock and 27 EquiBind single-candidate poses that did not enter the
strict cohort failed the independent graph/coordinate check. The full audit
retains each target's selection and failure reason rather than treating all
excluded poses as equivalent.

The existing `plb_bench` code was run locally under WSL Ubuntu with Python
3.11.6, OpenStructure 2.11.1, RDKit 2026.03.1, and Gemmi 0.7.5. The
protein coordinates came from the RCSB crystal reference; nonpolymer
ligands and waters were removed before adding each predicted SDF. The
reference passed to OpenStructure was restricted to the independently
selected native residue. Explicit hydrogens were removed **only from a
temporary scoring copy** of the SDF; original poses and heavy-atom
coordinates were preserved. Full-ligand matching, rather than
substructure-only matching or best-of-ranks selection, was required.
BiSyRMSD and lDDT-PLI were calculated separately. Missing values were
retained as missing; the comparison denominator is the number of IDs on
which **both** methods have that metric. Pocket Cα RMSD and whole-complex
QS were not compared because the model receptor was copied from the
crystal. The pathological target `5EQQ` was predeclared as a runtime
exclusion after a prior >20-minute OpenStructure stall; both poses remain
in the delivered and eligible denominators.

| Cohort / outcome | DiffDock | EquiBind |
| --- | ---: | ---: |
| Delivered primary poses | 8,419 | 8,565 |
| Strict-reference eligible | 3,982 | 3,963 |
| Reference/graph excluded | 4,437 | 4,602 |
| Runtime excluded (`5EQQ`) | 1 | 1 |
| Scoring exception | 1 | 1 |
| No BiSyRMSD (or only partial metrics) | 391 | 917 |
| BiSyRMSD obtained | 3,589 | 3,044 |
| lDDT-PLI obtained | 3,589 | 3,047 |

The two scoring exceptions are the same target, `4FBX`, where the selected
native residue was not recovered by the OpenStructure residue/element
check. Of the 1,308 partial-metric rows, 1,305 have neither metric and
three EquiBind rows have lDDT-PLI only. No failed score was filled with
zero or silently counted as success.

| Paired analysis | Eligible IDs | Both BiSyRMSD | Both lDDT-PLI | DiffDock median BiSyRMSD (Å) | EquiBind median BiSyRMSD (Å) | DiffDock median lDDT-PLI | EquiBind median lDDT-PLI |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Exact prediction graph / strict reference | 925 | 749 | 749 | 0.573 | 3.042 | 0.959 | 0.532 |
| Broader topology-matched / strict reference | 3,953 | 2,926 | 2,928 | 0.887 | 3.417 | 0.906 | 0.488 |

In the exact-graph cohort, DiffDock has lower BiSyRMSD in 696 of the 749
scored pairs, and higher lDDT-PLI in 710 of 749 (three lDDT ties).
The corresponding 2 Å BiSyRMSD counts are 662/749 for DiffDock and
178/749 for EquiBind. These are comparisons **among scoreable, paired,
delivered poses**, not dataset-wide success rates.

## Coordinate-frame quality control and interpretation

OpenStructure BiSyRMSD locally superposes the binding site and can map
equivalent protein chains. It therefore is **not** identical to the
fixed-frame pose displacement in the original crystal coordinates. Against
the independently computed provisional, untyped-graph direct RMSD, 171
scored method rows differ by >0.5 Å; 152 differ by >2 Å and 94 by >10 Å.
The full list is
`misato_output/score_summary_heavy_selected_v2/rmsd_qc_disagreements.csv`.
The differences are retained and flagged, not silently removed. For
example, target `4WSK` DiffDock has BiSyRMSD 1.31 Å versus fixed-frame
direct RMSD 113.23 Å; it cannot be described as accurate placement in
the original crystal frame solely from BiSyRMSD.

For all 925 exact-graph/strict-reference pairs, the provisional fixed-frame
direct RMSD median is 0.639 Å for DiffDock and 3.181 Å for EquiBind; these
are **exploratory sensitivity results**, not the original OpenStructure
metric. A post hoc subset of 717 exact-graph pairs has both formal scores
and ≤0.5 Å formal-versus-direct disagreement for both methods. It is
reported only as a frame-consistency sensitivity check, not substituted
for the prespecified 925-pair cohort. Native-ligand chemical identity and
large alignment discrepancies require adjudication before interpreting
the formal numbers as manuscript-ready docking accuracy.

The outstanding provenance request to Lucas is the per-ID prepared input
and status manifest, the exact RCSB receptor files/download and cleaning
procedure, the QM conformer/SDF-generation rule, and run logs for both
models. The full MD HDF5 is **not** needed to score these static poses, but
is needed for any additional MD-derived half of the earlier benchmark.
The current local `MD.hdf5` has a transfer sidecar and fails HDF5 root
parsing despite its apparent full size, so it was not used.

## Updated delivered-pose extension (27 September 2026)

The preceding strict-reference tables are retained as a historical baseline.
The versioned all-copies and multi-residue extension below supersedes their
*coverage* counts; it does not overwrite the old scores or imply that a
manuscript-ready comparison has already been exported. Gnina rescoring is
still in progress locally, and final figures have not been regenerated.

### Delivered denominator and selection limitation

The denominator for every docking coverage percentage is the delivered
primary-pose cohort: 8,419 DiffDock poses and 8,565 EquiBind poses (16,984
method rows). Roughly eleven thousand of the 19,413 QM IDs were never posed
by either method. Lucas attributed missing inputs chiefly to failed RCSB
protein downloads and RDKit ligand-read failures; his input-pipeline counts
are collaborator-reported and do not form a measured model-failure rate.
Neither the official MD train/validation/test split nor trajectory frames
were used for these rigid-receptor docking runs.
