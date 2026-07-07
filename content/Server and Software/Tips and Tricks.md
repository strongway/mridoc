---
title: Tips and Tricks
draft: false
tags:
  - apptainer
  - slurm
  - linux
  - lab
---

A catch-all of practical pointers for working on the lab servers and the LRZ cluster. Most items are one-liners that have saved time at some point. See [[Basic Linux Commands]] for the absolute basics, and [[LRZ Cloud Computing]] for cluster-specific details.

## Long-running jobs

- **Interactive sessions** — never run a multi-hour command in a plain SSH session; if the connection drops, the job dies. Wrap it in `tmux` (or `screen`). See [[tmux]] for the basics.
- **Batch jobs** — on LRZ, use SLURM (`sbatch`) and array jobs for multi-subject pipelines. The canonical pattern is in [[2.3 fmriPrep with Docker]] (search for `--array=`). On lab workstations, a `tmux` session is usually enough.

## Neurocommand in the Linux Cluster

### Install a container

```bash
bash containers.sh fmriprep
```

This prompts for the version you want. Example offer:

```
| fmriprep | 24.1.0 | 20241003 | functional imaging,workflows | Run:
--------------------------------------------------------------------
./local/fetch_containers.sh fmriprep 24.1.0 20241003
```

Pin versions deliberately and record them in [[Software Versions]] so analyses stay reproducible.

## Build your own Apptainer / Singularity image

Prefer `build` over `pull` — it is more reliable on slow or interrupted networks:

```bash
# Workstation (Apptainer)
apptainer build $CONTAINER_DIR/fitlins-<VER>.sif docker://poldracklab/fitlins:<VER>

# LRZ cluster (Singularity — same flags)
singularity build $CONTAINER_DIR/fitlins-<VER>.sif docker://poldracklab/fitlins:<VER>
```

Always pass `--cleanenv` when you `run` or `exec` the resulting `.sif`; otherwise host env vars (locale, `PYTHONPATH`, etc.) leak in and cause confusing failures.

## Apptainer tips

- **Cache location.** Pulled and built layers live in `~/.apptainer/cache` (or `~/.singularity/cache`). On LRZ, `$HOME` is small — override with `export APPTAINER_CACHEDIR=$DSS_DIR/apptainer-cache` before building.
- **Clean the cache** when it grows: `apptainer cache list` then `apptainer cache clean`.
- **Bind mounts.** Use `-B host:container[:ro]`:
  ```bash
  apptainer run --cleanenv \
      -B $BIDS_DIR:/data:ro \
      -B $DERIVS_DIR:/out \
      -B $FS_LICENSE:/opt/freesurfer/license.txt \
      $CONTAINER_DIR/fmriprep-<VER>.sif /data /out participant
  ```
- **Why `--cleanenv`.** Containers should be reproducible. `--cleanenv` strips host environment variables so the container sees only what you explicitly export via `APPTAINERENV_*` (or `SINGULARITYENV_*` on LRZ).

## Disk hygiene

Cluster quotas are real. Audit before deleting.

```bash
du -sh */ | sort -hr | head -20    # top-20 largest direct subdirs
ncdu .                              # interactive disk usage browser
df -h /dss /scratch ~              # see quota / free space per filesystem
```

> [!warning]
> Never run `rm -rf` against a `/dss/` path without first doing a dry run. A safe pattern: `find . -name '<pattern>' -print` first, then re-run with `-delete` once you have confirmed the list.

## Permissions and ACLs for shared lab folders

```bash
chgrp -R <lab-group> /dss/<project>/shared        # set group ownership
chmod -R g+rwX,o-rwx /dss/<project>/shared        # group can read/write; others none
chmod g+s /dss/<project>/shared                   # new files inherit the group
```

The capital `X` in `g+rwX` adds execute only to directories — exactly what you want for a mixed file tree.

## Symlinks for shared datasets

Avoid copying large BIDS datasets into every user's home. Link instead:

```bash
ln -s /dss/<project>/bids ~/bids
ln -s /dss/<project>/derivatives ~/derivatives
```

Scripts that read `$HOME/bids` then work for everyone without bloating quotas.

## TemplateFlow cache (share across users)

Many tools (fMRIPrep, MRIQC, FitLins) pull templates from TemplateFlow on first run. Point everyone at the same cache so it is fetched once:

```bash
export TEMPLATEFLOW_HOME=/dss/<project>/shared/templateflow
# Inside containers (workstation)
export APPTAINERENV_TEMPLATEFLOW_HOME=$TEMPLATEFLOW_HOME
# Inside containers (LRZ / Singularity)
export SINGULARITYENV_TEMPLATEFLOW_HOME=$TEMPLATEFLOW_HOME
```

## FreeSurfer license

fMRIPrep and several other tools require a FreeSurfer `license.txt`. Lab convention:

```bash
export FS_LICENSE=$HOME/license.txt
# bind into the container
apptainer run --cleanenv -B $FS_LICENSE:/opt/freesurfer/license.txt ...
```

> [!warning]
> Never commit `license.txt` to git. Add it to `.gitignore` in any analysis repo and keep the file mode at `0600`.

## Common gotchas

- **Windows line endings** in scripts you edited on a laptop cause cryptic `bad interpreter` errors. Fix with `dos2unix script.sh`.
- **SSH key permissions.** `~/.ssh/id_*` must be `chmod 600`; otherwise `ssh` refuses to use the key.
- **Locale issues** in older containers (`perl: warning: Setting locale failed`). The cheap workaround is `export LC_ALL=C` (or `APPTAINERENV_LC_ALL=C`) before the run.
- **Confound selection** — see [[5.2 Confound Selection]] when copy-pasting columns between fMRIPrep versions; column names occasionally change.

## Where to ask for help

- Lab Slack `#fmri-help` channel — first stop for "is this normal?" questions.
- Tool-specific bugs go to GitHub issues on the relevant project (fMRIPrep, MRIQC, FitLins, …). Include the exact version from [[Software Versions]].
- LRZ servicedesk for cluster-side problems (quota, broken modules, partition queues).
