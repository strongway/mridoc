---
title: Neurocommand
draft: false
tags:
  - Neurocommand
---

Neurocommand is a command-line interface to [Neurodesk](https://neurodesk.org), which is designed to run on virtual machines or HPC.

> [!note]
> **Neurocommand vs Neurodesk.** *Neurodesk* is the full containerised desktop (graphical environment, comes with everything pre-wired). *Neurocommand* is the lightweight `module`-style wrapper that exposes the same Neurodesk container collection on the command line — no desktop, just `module load fsl` etc. We use Neurocommand on our lab servers; reach for full Neurodesk only when you genuinely need the GUI desktop.

> [!note]
> **Apptainer first, Singularity fallback.** Neurodesk's modern docs assume **Apptainer**. On lab workstations use `module load apptainer` and set `APPTAINER_*` env vars. On the LRZ Linux cluster the module is still `singularity` — keep the `SINGULARITY_*` variables there. The CLIs are interchangeable; see [[LRZ Cloud Computing]] and [[2.3 fmriPrep with Docker]].

To get Neurocommand to work, you need to tell the bash shell system where to find the container folder and modules. For set-up users the following lines are already in your `.bashrc` (workstation, Apptainer):

```bash
export PATH=~/bin:/usr/local/go/bin:${PATH}
source /etc/profile.d/lmod.sh

# Apptainer (workstations) — same variable names with APPTAINER_ prefix
export APPTAINER_BINDPATH='/cvmfs,/mnt,/home,/dss'
export APPTAINERENV_TEMPLATEFLOW_HOME=/dss/.templateflow
module use /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*
```

On the LRZ cluster (or any host still using Singularity) substitute `SINGULARITY_BINDPATH` / `SINGULARITYENV_TEMPLATEFLOW_HOME` / `module load singularity` — the rest is identical.

> [!warning]
> The container module path `/cvmfs/neurodesk.ardc.edu.au/containers/modules/` is served via **CVMFS** and may not be mounted on every system. If `ls /cvmfs/neurodesk.ardc.edu.au` is empty, CVMFS is not available there — ask the admins or fall back to a locally built container (see [[2.3 fmriPrep with Docker]] and [[Software Versions]] for pinned versions used by the lab).

> [!tip]
> List everything Neurocommand exposes after sourcing the init script:
> ```bash
> module avail              # all modules in the current MODULEPATH
> module avail fsl          # filter by name
> module spider fmriprep    # show all versions and how to load them
> ```

## 1 Command line and bash file usage

```bash
module load module_name
module list
```

For example, if you want to load module, you can use
```bash
module load fsl
module list
```
it shows
```
Currently Loaded Modules:
  1) fsl/6.0.7.16
```

`module avail` shows all modules available from Neurocommand; `module spider <name>` lists every version of a single tool and tells you exactly how to load it.

> [!tip]
> When you launch a tool through a Neurocommand module, it runs inside an Apptainer/Singularity container under the hood. If you build your own ad-hoc `apptainer run` command for a similar workflow, pass `--cleanenv` so your shell variables don't leak in and confuse the container — same convention as in [[2.3 fmriPrep with Docker]].

**Scenario 1: Interactive Analysis**

If you tunnel to the server with option `-Y` (i.e., `ssh -Y account@lora`), you should be able to send back fsl interface back to your local computer. 

```bash 
module load fsl
Feat &
```

It will show the 'Feat' interface locally. 

>[!note]
> Tunnelling via SSH for the interface may be slow, particularly when you want to open the design matrix with multiple inputs. A better way to do this is via VNC interface, remotely running Feat or related software. 

**Scenario 2: Batch analysis using bash files**

The following is an example of using the `bet` function from the `fsl` package in a bash file.

```bash
#!/bin/bash
module load fsl
# Set the main BIDS directory (modify if needed)
BIDS_DIR="/dss/open/ds000102"
# Loop through subject folders
for subj in ${BIDS_DIR}/sub-*; do
	subj_id=$(basename ${subj}) # Extract subject ID (e.g., sub-01)
	# Define input and output paths
	anat_dir="${subj}/anat"
	input_image="${anat_dir}/${subj_id}_T1w.nii.gz"
	output_image="${anat_dir}/${subj_id}_T1w_brain.nii.gz"
	# Check if input file exists
	if [ -f "${input_image}" ]; then
		echo "Running BET on ${input_image}..."
		bet "${input_image}" "${output_image}" -R -f 0.25 -g 0 # Adjust parameters as needed
	else
		echo "T1w image not found for ${subj_id}, skipping..."
	fi
done

echo "Brain extraction completed for all subjects."
```

## 2 Use in Python

Python does not inherit the bash module environment automatically, so you have to recreate it at the top of your script or notebook. On a workstation (Apptainer):

```python
import os
os.environ["APPTAINER_BINDPATH"] = "/cvmfs,/mnt,/home,$HOME/dss"  # adjust to your account
os.environ["LMOD_CMD"] = "/usr/share/lmod/lmod/libexec/lmod"
os.environ["APPTAINERENV_TEMPLATEFLOW_HOME"] = f"{os.environ['HOME']}/dss/.templateflow"
os.environ["MODULEPATH"] = "/cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*"
```

On LRZ (still Singularity), swap the two `APPTAINER*` lines for `SINGULARITY_BINDPATH` and `SINGULARITYENV_TEMPLATEFLOW_HOME`.

Then you can load modules via `lmod`
```python
# need to load base (Python) from miniconda (all packages installed)
import lmod
#await lmod.purge(force=True)
await lmod.load('fsl')
await lmod.list()
```

### 3. Use with Google Colab

The official guidance is [here](https://www.neurodesk.org/docs/getting-started/hosted/googlecolab/)

At the beginning of your notebook, you need to add: 

```python
import os
os.environ["LD_PRELOAD"] = "";
os.environ["APPTAINER_BINDPATH"] = "/content"
os.environ["MPLCONFIGDIR"] = "/content/matplotlib-mpldir"
os.environ["LMOD_CMD"] = "/usr/share/lmod/lmod/libexec/lmod"

!curl -J -O https://raw.githubusercontent.com/NeuroDesk/neurocommand/main/googlecolab_setup.sh
!chmod +x googlecolab_setup.sh
!./googlecolab_setup.sh

os.environ["MODULEPATH"] = ':'.join(map(str, list(map(lambda x: os.path.join(os.path.abspath('/cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/'), x),os.listdir('/cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/')))))
```

Then, you can use it:
```python
import lmod
await lmod.avail()

await lmod.load('fsl/6.0.4')
!bet

```

