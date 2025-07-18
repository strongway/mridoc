---
title: Neurocommand
draft: false
tags:
  - Neurocommand
---

Neurocommand is a command-line interface to [Neurodesk](https:/neurodesk.org), which is designed to run on virtual machine or HPC. 

To get NeuroCommand to work, you need to tell the bash shell system where to find the singularity folder and modules. For those set up users, the following codes are already in your `.bashrc`. 

```bash
export PATH=~/bin:/usr/local/go/bin:${PATH}
source /etc/profile.d/lmod.sh

export SINGULARITY_BINDPATH='/cvmfs,/mnt,/home,/dss'
export SINGULARITYENV_TEMPLATEFLOW_HOME=/dss/.templateflow
module use /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*
```

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

`module avail` can show all modules available from the Neurocommand. 

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

Unfortunately, the Python environment cannot directly read the bash environment. So you need to set up the environment first in your Python. If you are using Jupyter Notebook, copy the following code to the beginning of your notebook:

```python
import os
os.environ["SINGULARITY_BINDPATH"] = "/cvmfs,/mnt,/home,/home/lu32pog/dss"
os.environ["LMOD_CMD"] = "/usr/share/lmod/lmod/libexec/lmod"
os.environ["SINGULARITYENV_TEMPLATEFLOW_HOME"] = "/home/lu32pog/dss/.templateflow"
os.environ["MODULEPATH"] = "/cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*"
```

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

