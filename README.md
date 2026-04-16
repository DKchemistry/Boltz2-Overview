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

#### MSA Generation For An Example Protein

For our protein target, we'll use B2AR, sourced from [UniProt](https://rest.uniprot.org/unisave/P07550?format=fasta&versions=258).

I've saved the fasta file to: `./fasta/B2AR.fasta`

Now, we need to make the MSA.

From our current directory, we'll activate the env: 

```sh
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /mnt/data/dk/msa-env
```
We'll save our results to `./msa` and we'll use the default settings:

```sh
colabfold_search fasta/B2AR.fasta /mnt/data/dk/colabfold_db msa
```
This generated `msa/sp_P07550_ADRB2_HUMAN_Beta-2_adrenergic_receptor_OS_Homo_sapiens_OX_9606_GN_ADRB2_PE_1_SV_3.a3m`. 

#### Ligand Preparation

For the example ligand, we'll use the classic B2AR ligand carazolol.

We'll save it as a SMILES file: smiles/carazolol.smi

We'll use Schrodinger's (2025-2) ligprep to make the most likely protomer/tautomer at physiological pH. 

```sh
cd /mnt/data/dk/work/Boltz2-Overview
mkdir -p sdf
cd /mnt/data/dk/work/Boltz2-Overview/sdf
ligprep -s 1 -i 2 -W i,-ph,7.4,-pht,0.0 -t 1  -HOST localhost:1 -ismi ../smiles/carazolol.smi -osd carazolol.sdf
```
Now we will convert this back to a SMILES file. 

```sh
cd /mnt/data/dk/work/Boltz2-Overview
mkdir -p prepared_smiles
$SCHRODINGER/utilities/structconvert sdf/carazolol.sdf prepared_smiles/carazolol.smi
``` 
#### Setting up the Boltz2 YAML File

There are a few examples from the Boltz team on how to set up various types of YAML files [here](https://github.com/jwohlwend/boltz/tree/main/examples).

We'll use the one given in the `affinity.yaml` example, which looks like so generically: 

```yaml
version: 1
sequences:
  - protein:
      id: A
      sequence: <protein sequence>
      msa: <path to .a3m>
  - ligand:
      id: L
      smiles: "<prepared ligand smiles>"
properties:
  - affinity: { binder: L }
```
Our actual YAML lives here: `yaml/b2ar_carazolol_affinity.yaml`. 

And looks like this: 

```yaml
version: 1
sequences:
  - protein:
      id: A
      sequence: MGQPGNGSAFLLAPNGSHAPDHDVTQERDEVWVVGMGIVMSLIVLAIVFGNVLVITAIAKFERLQTVTNYFITSLACADLVMGLAVVPFGAAHILMKMWTFGNFWCEFWTSIDVLCVTASIETLCVIAVDRYFAITSPFKYQSLLTKNKARVIILMVWIVSGLTSFLPIQMHWYRATHQEAINCYANETCCDFFTNQAYAIASSIVSFYVPLVIMVFVYSRVFQEAKRQLQKIDKSEGRFHVQNLSQVEQDGRTGHGLRRSSKFCLKEHKALKTLGIIMGTFTLCWLPFFIVNIVHVIQDNLIRKEVYILLNWIGYVNSGFNPLIYCRSPDFRIAFQELLCLRRSSLKAYGNGYSSNGNTGEQSGYHVEQEKENKLLCEDLPGTEDFVGHQGTVPSDNIDSQGRNCSTNDSLL
      msa: /mnt/data/dk/work/Boltz2-Overview/msa/sp_P07550_ADRB2_HUMAN_Beta-2_adrenergic_receptor_OS_Homo_sapiens_OX_9606_GN_ADRB2_PE_1_SV_3.a3m
  - ligand:
      id: L
      smiles: "CC(C)[NH2+]C[C@@H](O)COc1cccc2[nH]c3ccccc3c12"
properties:
  - affinity: { binder: L }
```
Now, we can run Boltz! We will use mostly default arguments here. The changes here primarily using our local cache and allow overriding of results.

For now, we also have to specify`--no_kernels`, which disables optional Triton-based optimized kernels. This is primarily a compatibility/performance setting and is not intended to change the underlying Boltz2 model or YAML semantics. In practice, it should mainly affect execution speed and stability on a given system. I will look into this more later, but it is not a huge priority. 

Also becareful about exporting the CUDA device on `Tobias`, there is some weirdness between what we see in `nvidia-smi -L` for device number versus the UUID. Be explicit and supply the UUID.

```sh
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate /mnt/data/dk/boltz2-env

export CUDA_VISIBLE_DEVICES=MIG-0c920aac-5e24-5455-9ebe-03a606d5713d
export BOLTZ_CACHE=/mnt/data/dk/.cache/boltz

cd /mnt/data/dk/work/Boltz2-Overview

boltz predict yaml/b2ar_carazolol_affinity.yaml \
  --out_dir boltz_predictions/b2ar_carazolol_boltz \
  --cache "$BOLTZ_CACHE" \
  --no_kernels \
  --override
```
We can check out the results here: `boltz_predictions/b2ar_carazolol_boltz/boltz_results_b2ar_carazolol_affinity`. The output looks normal. 