---
title: LRZ Cloud Computing
draft: false
tags:
  - Cloud
  - HPC
---
The following information is useful for those who can access Linux cluster at LRZ.de

> [!note]
> **Apptainer vs Singularity.** Apptainer is the community fork of Singularity; the two CLIs are drop-in compatible for everything we do (same `build`, `run`, `exec`, `shell`, `--bind`, `--cleanenv` flags). The lab convention is:
> - On **workstations** (`Lora`, `Emma`) and modern HPC sites: use `apptainer`.
> - On the **LRZ Linux cluster**: the module is still `singularity` — use that name there. Commands below labelled "on LRZ" use `singularity` deliberately.
> See also [[2.3 fmriPrep with Docker]] for the canonical container-run pattern and env-var conventions (`$BIDS_DIR`, `$DERIVS_DIR`, `$FMRIPREP_DIR`, `$WORK_DIR`, `$FS_LICENSE`).

## Access Cloud Computing

Msense Lab also has two cloud computing systems, `Lora` and `Emma`. `Emma` is accessible from the external internet, while `Lora` is walled by the intranet. 

> [!tips]
> For convenience, it's better that you create aliases for those servers. For example, in your `.zshrc` or `.bashrc`, put similar lines:
> `alias lora="10.195.2.##"` (please change the ip accordingly)
> 

### 1. __Via `SSH`__
```
# access to Lora
ssh -Y your_account@lora

```
### 2. __Via VNC__

2. Via VNC
First, you need to tunnel to the server and initiate the VNC server (this only needs to be done once). The following commands should be run on the server side. 

2.1 Setup VNC password
```bash 
vncpasswd
```

You’ll be prompted to enter and verify your password. This password is required when connecting with your VNC client.

2.2 Start VNC server
```bash 
vncserver
```

It will show you "New 'X' desktop is lora:#". Remember this, we will first kill it and edit the file -  `xstartup` and restart it again. 

2.3 Edit VNC `xstartup` file

First, kill the VNC service
```bash 
vncserver -kill :# (replace the number)
```

Now edit the xstartup
```bash
nano ~/.vnc/xstartup
```

Edit/Put the following text in the file:
```
#!/bin/sh
xrdb $HOME/.Xresources

# Start Xfce
startxfce4 &
```

2.4 Start VNC server localhost

```
vncserver -localhost -geometry 1024x768
```

Depending on your preference, you can specify the dimension of the virtual desktop. If you want to change it, you have to kill the service first, and restart it again (see above instructions). And remember which port number it started. You can check it in the `~/.vnc/` folder, by finding the lora:#.pd file.

2.5 Local connection

Now you can exit the server, and start **a local command window**, type: 
```
# Access to Lora and replace the # with your VNC port number
ssh -L 590#:localhost:590# -C -N -l your_account 10.195.?.?
```
Replace the correct server IP, depending on your server.  

Next, you can start your local VNC (under Mac) using "Finder --> Go --> Connect to Server", type in:
```
vnc://localhost:590#
```
The connection password is the password you set earlier on. 

### 3. Via Code Server

The code server is configured individually. If your account is set up, you can first establish the tunnel:

```bash
ssh -Y -N -L 808@:localhost:808@ your_account@lora_ip
```
Please replace the port number, account name, and IP above. And leave this command window open. 

Next, open your browser and type localhost:808@. A code server interface will appear. Code Server is convenient when you don't have a local installation of Visual Studio Code. However, if you accidentally close the brower tab, you may lose some working that is running.

### 4. Via Visual Studio Code

You need to install 'remote-SSH' and 'Remote Explorer' plugins. After you install these plugins, by click the left icon of 'Remote Explorer', add remote SSH, using the option 1 'Via SSH' to connect to the server. 

When the VS Code connects to the server, it will show the connection in the lower-left corner. The first time it opens nothing. You need to select 'Open Folder' to open a desired destination. 

### 5. Mapping the remote server locally

This is desirable when checking remote MRI results like local files. It won't do any remote computation. And it requires changing the security level in your local Mac computer. 

First, install MacFUSE and SSHFS
```bash
brew install macfuse
brew tap gromgit/fuse
brew install gromgit/fuse/sshfs
```

Second, create a local mount point

```bash
mkdir ~/dss
```

Third, mount the remote linux server folder:

```bash
sshfs username@server:/path/remote ~/dss
```

## Access Linux Cluster

It requires a Linux cluster LRZ account and 2FA. Details can be found in the LRZ website. 

```bash
# connect to linux cluster at LRZ

ssh -Y your_account@cool.hpc.lrz.de

```

### 1. Resource

Overview of cluster specifications and limits

https://doku.lrz.de/job-processing-on-the-linux-cluster-10745970.html

Status here:
https://doku.lrz.de/high-performance-computing-10613431.html

### 2. Install / load singularity on LRZ

On the LRZ Linux cluster the container runtime module is still named **`singularity`** (not `apptainer`). The CLI is identical — `apptainer` and `singularity` accept the same flags, so any recipe from [[2.3 fmriPrep with Docker]] works here by substituting the binary name.

**Discover the available version** before loading:
```bash
module spider singularity   # list all versions and how to load each
module avail singularity    # show what is loadable in the current module set
```

The recipe below has already been run for the lab account; you normally only need the *Usage* block. When logged in to the Linux cluster (on the login nodes):

```
module rm intel-mpi intel-mkl intel  
module load user_spack  
spack install squashfs squashfuse  
spack load squashfs squashfuse  
spack install singularity ~suid  
spack -c"modules:tcl:blacklist_implicits:False" module tcl refresh --upstream-modules squashfs singularity
```
**Usage** (new terminal or inside a SLURM job):
```bash
module use spack/modules/x86_avx2/linux-sles15-haswell
module load singularity squashfs
```

> [!tip]
> **Build vs pull.** Prefer `singularity build` (or `apptainer build` on workstations) over `singularity pull` — `build` is more reliable, especially for multi-layer Docker images:
> ```bash
> singularity build $CONTAINER_DIR/fmriprep-<VER>.sif docker://nipreps/fmriprep:<VER>
> ```
> On a workstation the equivalent is `apptainer build ... docker://...`.

> [!info]
> **LRZ storage tiers.**
> - `$HOME` — small quota, backed up. Fine for scripts, configs, `.sif` files if they fit.
> - **DSS** (`/dss/...`, refer to it as `$DSS_DIR` in your scripts; never hardcode `/dss/youraccount`) — large, project-shared. Best place for BIDS datasets, derivatives, and `.sif` containers shared across users.
> - **SCRATCH** — fastest, *not* backed up, periodically purged. Use for `$WORK_DIR` (fMRIPrep working dirs, temporary intermediates).
>
> Rule of thumb: containers in DSS or HOME; raw and derived data in DSS; transient working dirs in SCRATCH.

### 3. Slurm partition and job settings

The following information needs to be updated; please check lrz.

	job settings
	**cm2_std:** 
	--clusters=cm2  
	--partition=cm2_std  
	--qos=cm2_std
	
	**cm2_inter**:
	--clusters=inter  
	--partition=cm2_inter
	
	**cm2_inter_large_mem**
	56 cpu, memory 120 G
	--clusters=inter  
	--partition=cm2_inter_large_mem  
	--mem=<memory_per_node_MB>M

**Common Slurm commands for job management:**
56 cpu 56 G
```
squeue -M cm2_tiny -u $USER  
scancel -M cm2_tiny <JOB-ID>  
sacct -M cm2_tiny -X -u $USER --starttime=2021-01-01T00:00:01
```

```
squeue -M inter -u $USER  
scancel -M inter <JOB-ID>  
sacct -M inter -X -u $USER --starttime=2021-01-01T00:00:01
```


### 4. Interactive 

```
salloc -N 1 -M inter -p cm4_inter -t 8:00:00

```
#### show CPUs and Memory
```
lscpu | grep '^CPU(s):'
free -h
```

#### submit to slurm

```bash
sbatch sing_cluster.sh
```

A minimal `sing_cluster.sh` for a single-subject container run on LRZ — note `--cleanenv` and the use of env-var placeholders rather than hardcoded paths:

```bash
#!/bin/bash
#SBATCH --clusters=cm2
#SBATCH --partition=cm2_std
#SBATCH --qos=cm2_std
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=24:00:00

module use spack/modules/x86_avx2/linux-sles15-haswell
module load singularity squashfs

export DSS_DIR=/dss/<your-project>          # placeholder — set for your account
export BIDS_DIR=$DSS_DIR/bids
export DERIVS_DIR=$DSS_DIR/derivatives
export FMRIPREP_DIR=$DERIVS_DIR/fmriprep
export WORK_DIR=$SCRATCH/work               # SCRATCH for transient I/O
export FS_LICENSE=$HOME/license.txt

singularity run --cleanenv \
    -B $BIDS_DIR:/data:ro \
    -B $DERIVS_DIR:/out \
    -B $WORK_DIR:/work \
    -B $FS_LICENSE:/opt/freesurfer/license.txt \
    $DSS_DIR/containers/fmriprep-<VER>.sif \
    /data /out participant \
    --participant-label ${SLURM_ARRAY_TASK_ID} \
    -w /work --fs-license-file /opt/freesurfer/license.txt
```

For multi-subject processing, use a SLURM **array job** (`#SBATCH --array=1-30`) — see the canonical pattern in [[2.3 fmriPrep with Docker]] and adapt the `--participant-label` line accordingly.

