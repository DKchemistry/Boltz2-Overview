# Boltz2 Overview

## Purpose

This repository documents the local Boltz2 inference setup on `tobias` for user `dk`, including:
- environment activation
- offline MSA generation
- local databases
- template usage
- inference examples
- smoke tests
- troubleshooting

## Host and storage layout

- Hostname: `tobias`
- User: `dk`
- Root/home volume: `/` (~983 GB)
- Large data volume: `/mnt/data` (~41.9 TB)

Large persistent assets are intentionally stored under `/mnt/data/dk`:
- conda envs
- ColabFold/MMseqs databases
- Boltz model cache
- project working directories

## Important paths

- Boltz2 env: `/mnt/data/dk/boltz2-env`
- MSA env: `/mnt/data/dk/msa-env`
- Offline database root: `/mnt/data/dk/colabfold_db`
- Boltz cache: `/mnt/data/dk/.cache/boltz`
- Example project: `/mnt/data/dk/boltz_gpr3_test`

## Environment versions

### Boltz2 inference env
Path: `/mnt/data/dk/boltz2-env`

- Python 3.10.18
- boltz 2.2.0
- torch 2.8.0

### Local MSA env
Path: `/mnt/data/dk/msa-env`

- Python 3.10.18
- mmseqs2 18.8cc5c
- colabfold 1.5.5
- foldseek 10.941cd33

## Offline database status

Database root: `/mnt/data/dk/colabfold_db`

Readiness markers present:
- `DOWNLOADS_READY`
- `UNIREF30_READY`
- `COLABDB_READY`
- `PDB_READY`
- `PDB100_READY`
- `PDB_MMCIF_READY`

### Main sequence/MSA databases

These are the main logical sequence databases used for local MSA generation:
- `uniref30_2302_*`
- `colabfold_envdb_202108_*`

### Main structure/template resources

These are the main structure/template resources:
- `pdb100_230517*`
- `pdb100_a3m.ffdata`
- `pdb100_a3m.ffindex`
- `pdb/`

## Model cache

Path: `/mnt/data/dk/.cache/boltz`

Observed files:
- `boltz2_aff.ckpt`
- `boltz2_conf.ckpt`
- cached ligand molecule files

## Known example project

Path: `/mnt/data/dk/boltz_gpr3_test`

Observed assets include:
- FASTA inputs
- YAML inputs
- ligand TSV input
- template YAMLs
- local MSA files
- run scripts
- prior output directories

## Open questions

- Canonical smoke test command
- Exact YAML conventions used in prior successful runs
- Which workflows require explicit templates
- Which workflows reuse precomputed MSAs
- What the batch scripts do
- Minimal documented workflow for a new project

