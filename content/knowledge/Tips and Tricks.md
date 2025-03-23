---
title: Tips and Tricks
draft: false
tags:
---
## Neurocommand in Linux Cluster

### Install Container
1. Search and install Specific Containers
```bash
bash containers.sh fmriprep
```
The above will give you options of which specific containers you want to install. For example:

```
| fmriprep | 24.1.0 | 20241003 | functional imaging,workflows | Run:

--------------------------------------------------------------------

./local/fetch_containers.sh fmriprep 24.1.0 20241003
```

## Install Singularity containers

Find docker images, and then build it with `singularity`. For example, install `FitLins`:

```bash
singularity build /path/to/fitlins-<VERSION>.simg \
  docker://poldracklab/fitlins:<VERSION>
```
