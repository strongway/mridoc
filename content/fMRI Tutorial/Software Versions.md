---
title: Software Versions
draft: false
tags:
  - reference
  - reproducibility
---

This page pins the software versions used throughout the MSense fMRI tutorials. If your installed versions differ from the ones below, your numbers, figures, and confound files may not match what the tutorials show. When in doubt, match these pins first, then explore upgrades.

> [!note]
> Snapshot date: **early 2026**. Tools like AFNI follow a rolling release and have no fixed version number — pin the build date instead.

## Why pin versions at all?

fMRI pipelines are long chains of small numerical decisions. Each one is sensitive to defaults that change between releases:

- **Confounds change names.** fMRIPrep v23 → v24 renamed and reorganised several columns in `*_desc-confounds_timeseries.tsv`. Scripts that hard-code column names break silently.
- **Normalisation templates differ.** `MNI152NLin2009cAsym` (fMRIPrep default) and `MNI152NLin6Asym` (FSL default) are not the same space. Atlases, ROIs, and group-level results must use a consistent template.
- **Default smoothing kernels** vary across FEAT, fMRIPrep workflows, and nilearn examples. Always set FWHM explicitly.
- **Container digests** are the only truly reproducible identifier. A tag like `:latest` will give a different image next month.
- **Stats models** (BIDS Stats Models v1.0.0) and FitLins now require explicit model JSONs — older notebooks built around `fitlins` 0.10 will not run unchanged.

A practical rule for the course: **record the exact versions in your project README** before you generate any figure for a thesis or report.

## Recommended pins

| Tool | Pinned version | Notes |
| --- | --- | --- |
| Python | 3.11 | 3.10 minimum; 3.12 works but is less tested with Nipype |
| fMRIPrep | 24.1.1 | Confound column names changed from v23 |
| MRIQC | 24.0.x | Pair with the fMRIPrep major version where possible |
| FSL | 6.0.7.x | Tutorials [[4.1 First-level Analysis with FSL]] and [[4.2 High-level Analysis with FSL]] assume this |
| AFNI | latest stable build | Rolling release — record the build date with `afni -ver` |
| SPM12 | not required | Mentioned only for comparison; install only if you follow the optional Nipype SPM track |
| dcm2bids | 3.x | Uses dcm2niix under the hood |
| dcm2niix | latest stable | Updated alongside dcm2bids |
| BIDS specification | 1.10.x | See [[2.2 BIDS Structure]] |
| bids-validator | ≥ 2.0 | The Deno-based v2 validator replaces the old Node CLI |
| BIDS Stats Models | 1.0.0 | Required by recent FitLins |
| nilearn | 0.10.x | See flag below |
| nibabel | 5.x | |
| numpy | 1.26 or 2.x | Both supported; pin one per project |
| pandas | 2.x | |
| matplotlib | ≥ 3.8 | |
| Nipype | 1.8.x | Optional drafts (Nipype Basics, Nipype for 1st-level fMRI analysis); expect minor breakage |
| FitLins | 0.11.x | Pin explicitly; consumes BIDS Stats Models v1.0. Run inside a container for reproducibility |
| templateflow | ≥ 24.2.x | fMRIPrep and MRIQC delegate template downloads to templateflow; an older templateflow can fetch a different *default* template than the version pinned in fMRIPrep, silently changing your normalisation target |
| FreeSurfer | ≥ 7.4.x | fMRIPrep delegates surface reconstruction to FreeSurfer's `recon-all`. The FS license file must be available (see `$FS_LICENSE` in [[2.3 fmriPrep with Docker]]) |
| Apptainer / Singularity | ≥ 1.3 | Preferred on HPC over Docker |
| Docker | ≥ 24 | For local runs on laptops |
| Node.js | ≥ 22 | Only needed to build this Quartz site, not for analyses |

> [!warning]
> Tutorial [[5.1 NiLearn fMRI Basic Analysis]] currently lists `nilearn 0.12+`. That version did not exist at the time of writing. Use **nilearn 0.10.x** until 0.11 / 0.12 ship and the tutorial is updated. The maintainer should confirm whether to upgrade the tutorial or downgrade the pin.

> [!note]
> AFNI's "version" is its build date. Two installations with the same major release can still differ. Always log the output of `afni -ver`.

## Installation pointers

These are one-line entry points. Detailed install steps live in each tool's tutorial.

### Containers (preferred for fMRIPrep, MRIQC, FitLins)

```bash
# Apptainer / Singularity on HPC
apptainer pull docker://nipreps/fmriprep:24.1.1
apptainer pull docker://nipreps/mriqc:24.0.0

# Docker on a laptop
docker pull nipreps/fmriprep:24.1.1
docker pull nipreps/mriqc:24.0.0
```

See [[2.3 fmriPrep with Docker]] and [[2.4 Quality Control with MRIQC]].

### Python stack

```bash
# Inside a fresh venv (python -m venv .venv && source .venv/bin/activate)
pip install \
  "nilearn==0.10.*" \
  "nibabel>=5,<6" \
  "numpy>=1.26" \
  "pandas>=2" \
  "matplotlib>=3.8" \
  "nipype==1.8.*" \
  "dcm2bids>=3,<4"
```

### FSL

```bash
# Use the official installer from fmrib.ox.ac.uk
python fslinstaller.py -V 6.0.7.16
```

### AFNI

```bash
# Linux example; on macOS use the official installer script
@update.afni.binaries -package linux_ubuntu_24_64 -do_extras
```

### BIDS validator (v2, Deno)

```bash
deno run -A jsr:@bids/validator path/to/bids_dataset
```

### SPM12 (optional)

Only needed if you follow the optional SPM branch of [[Nipype for 1st-level fMRI analysis]]. Install via the SPM website and point `MATLABCMD` at your MATLAB or Octave binary.

## Recording versions in your project

Reproducibility is cheap if you do it from day one. Every project folder should contain a record of what produced its results.

### Python dependencies

```bash
# Pip-based projects
pip freeze > requirements.txt

# Conda-based projects
conda env export --no-builds > environment.yml
```

### Neuroimaging tools

```bash
fmriprep --version          # 24.1.1
mriqc --version             # 24.0.0
flirt -version              # FLIRT version 6.0
afni -ver                   # AFNI_25.x.x 'Build-date ...'
```

Append these to a `versions.txt` in your project root and commit it.

### Container digests (the gold standard)

A tag can be moved. A digest cannot.

```bash
apptainer inspect --json fmriprep_24.1.1.sif | grep -i digest
docker inspect --format '{{index .RepoDigests 0}}' nipreps/fmriprep:24.1.1
```

Record the resulting `sha256:...` in your methods section.

## Version-variable convention

All tutorials use the same pattern: a shell variable per tool, set once at the top of the script. This makes upgrading deliberate rather than accidental.

```bash
export FMRIPREP_VERSION=24.1.1
export MRIQC_VERSION=24.0.0
export FSL_VERSION=6.0.7.16
export PYTHON_VERSION=3.11

# Then use them everywhere
apptainer pull docker://nipreps/fmriprep:${FMRIPREP_VERSION}
apptainer pull docker://nipreps/mriqc:${MRIQC_VERSION}
```

For Python projects, mirror the same idea in `pyproject.toml` or `environment.yml`:

```yaml
# environment.yml
name: fmri-course
channels:
  - conda-forge
dependencies:
  - python=3.11
  - pip
  - pip:
      - nilearn==0.10.*
      - nibabel>=5,<6
      - nipype==1.8.*
      - pandas>=2
      - numpy>=1.26
```

## When versions drift

If your output disagrees with the tutorial:

1. Check `fmriprep --version` and `mriqc --version` against the table above.
2. Open `*_desc-confounds_timeseries.tsv` and confirm column names match what your script expects (this is the v23 → v24 trap).
3. Verify the output space — most tutorials assume `MNI152NLin2009cAsym`.
4. Compare your container digest against the one recorded for the original run.

> [!tip]
> When you upgrade a tool, re-run a single subject end-to-end and diff the headline statistics before trusting the new version across the dataset.

## Related pages

- [[2.2 BIDS Structure]]
- [[2.3 fmriPrep with Docker]]
- [[2.4 Quality Control with MRIQC]]
- [[2.5 fMRIPrep and FitLins]]
- [[4.1 First-level Analysis with FSL]]
- [[5.1 NiLearn fMRI Basic Analysis]]
- [[Nipype Basics]]
