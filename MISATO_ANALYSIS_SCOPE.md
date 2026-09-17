# Misato docking analysis — scope and provenance

## Current deliverable

This repository is an exploratory fork. The current deliverable is a reproducible analysis of the supplied MISATO **EquiBind** and **DiffDock** outputs, separate from the published ASD/PLA benchmark.

The work is limited to an immutable archive inventory, reconstruction of the evaluated subset and all exclusions, format-specific normalization, validated scoring only after the receptor/reference coordinate frame is established, method coverage and provisional comparison figures, and a concise methods/provenance report.

It does not include new docking runs, TankBind, full MISATO MD reprocessing, edits to legacy outputs, or a manuscript addendum. Those require a later PI decision.

## Provenance constraint

The supplied folders contain ligand-only SDF predictions. They do not contain receptor structures, input preparation scripts, logs, configurations, or trajectory-frame identifiers. They establish which IDs were run, but not whether Lucas used static experimental receptors or MD-derived conformations. No protein-aligned docking score may be reported until the coordinate frame is validated.

## Decision gate

- **Approved addendum:** freeze source release, manifest, filters, and software configuration, then prepare a reviewed PR with Misato-specific methods and results.
- **Deferred:** retain this fork as the reproducible standalone analysis and do not merge or combine its figures with published claims.
