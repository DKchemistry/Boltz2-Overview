# Boltz2 on Tobias

This write up is to detail the install of Boltz2 on Tobias to use it in various projects. Because I had my home directory set up outside of /mnt/data and instead in a much smaller partition on our drives, it difficult to keep track of how I should be using Boltz2 on this machine. This is to make this easier.

## Core paths

- Boltz2 env: `/mnt/data/dk/boltz2-env`
- MSA env: `/mnt/data/dk/msa-env`
- Offline database root: `/mnt/data/dk/colabfold_db`
- Boltz cache: `/mnt/data/dk/.cache/boltz`

## What each thing is for

### Boltz2 env

Use this env to run inference with `boltz predict`.

Activate with:

```
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /mnt/data/dk/boltz2-env
```
- Python 3.10.18
- boltz 2.2.0
- torch 2.8.0

### MSA env

Use this env for local MSA generation with MMseqs / ColabFold tools.

Activate with:

```
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /mnt/data/dk/msa-env
```
- Python 3.10.18
- mmseqs2 18.8cc5c
- colabfold 1.5.5
- foldseek 10.941cd33

### Offline databases

Local MSA databases are stored at:

/mnt/data/dk/colabfold_db

These were downloaded so that we can perform offline MSA generation as input to Boltz, without requiring the online webserver. 

These were downloaded via: `/mnt/data/dk/setup_databases.sh`. I did not write this script myself, it is sourced from the [ColabFold GitHub](https://github.com/sokrypton/ColabFold/blob/main/setup_databases.sh?). Only some of these datasets are used for MSA generation:

	•	uniref30_2302
	•	colabfold_envdb_202108

Other resources downloaded by the shell script are not strictly necessary for the Boltz2 workflow, but I haven't removed them. These are files like: 

	•	pdb100_230517
	•	pdb100_foldseek_230517
	•	local pdb/ mmCIF content

It may be prudent to update the sequence relevant databases which we can check for here: 

- ColabFold MSA server database history:
  https://github.com/sokrypton/ColabFold/wiki/MSA-Server-Database-History

### Boltz cache

Out of an abundance of precation, we set Boltz' cache in /mnt/data/ as it could otherwise default to home and overload the drive.

Boltz checkpoints and cached molecule data are stored at:

/mnt/data/dk/.cache/boltz

Set:

export BOLTZ_CACHE=/mnt/data/dk/.cache/boltz

### Example Workflows

Here, we'll set up some basic examples of how to run Boltz2 on this machine. 

For our protein target, we'll use B2AR.

