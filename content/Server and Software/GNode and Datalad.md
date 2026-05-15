---
title: GNode and Datalad
draft: false
tags:
  - GNode
  - Datalad
  - git-annex
  - data-management
---

> [!info] datalad vs git-annex vs git
> **git** tracks code and other small text files. **git-annex** tracks the *content addresses* of large binary files, so the git repository stays small while you still version huge data. **DataLad** wraps git + git-annex into a user-friendly tool for neuroimaging — datasets are git repositories whose data is fetched on demand. **GNode** (via GIN) is an LMU-hosted service that speaks this protocol. See [[Git Annex]] for the underlying mechanics.

## 1. G-Node 

Focusing on the development and free distribution of tools for handling and analyzing neurophysiological data, G-Node aims to address these aspects as part of the International Neuroinformatics Coordinating Facility ([INCF](http://www.incf.org/)) and the German Bernstein Network for Computational Neuroscience ([NNCN](http://www.nncn.de/)). G-Node also serves as an international forum for Computational Neuroscientists interested in sharing experimental data and tools for data analysis and modeling. G-Node is funded through the German Federal Ministry of Education and Research and hosted by [Ludwig-Maximilians-Universität München](https://lmu.de).

Benefits of G-Node over Github:
- Hosting large binary files
- Assigning unique DOI. It is good to open data and get citations. 
- Using [[Git Annex]] as infrastructure helps manage versions. 

### gin 

Management of scientific data, including consistent organization, annotation, and storage of data, is a challenging task. Accessing and managing data from multiple workplaces while keeping it in sync, backed up, and easily accessible from within or outside the lab is even more demanding. The GIN (G-Node Infrastructure) service is a free and open data management system designed for comprehensive and reproducible management of neuroscientific data.

### gin client installation

Please see the [official website]( https://gin.g-node.org/G-Node/Info/wiki/GIN+CLI+Setup) for installations. For Mac OS users, the easiest way to install the client on macOS is via [homebrew](https://brew.sh/). G-Node homebrew formulae are maintained in the [G-Node tap](https://github.com/g-node/homebrew-pkg). Install the client, including any dependencies, with:

```bash
brew tap g-node/pkg
brew install g-node/pkg/gin-cli
```

Alternatively, if you already have git and git-annex installed on your system, or you want to install them manually or via homebrew, the recommended and simplest way to install git-annex is via [Homebrew](https://git-annex.branchable.com/install/OSX/Homebrew/) using `brew install git-annex`. Alternatively, download git-annex from the [git-annex website](https://git-annex.branchable.com/install/OSX/).

Once you've installed git-annex, simply download the [gin client for macOS](https://gin.g-node.org/G-Node/gin-cli-releases/raw/master/gin-cli-latest-macos.tar.gz), extract the archive, and put the file named `gin` in a location that's included in your `$PATH`.

### Basic usage of gin

1. register g-node.org website and sign into the [GIN Server](https://gin.g-node.org/user/login).
2.  Create a new repository using the "+" on the top right. Alterantively, you can create locally:
```bash
gin create <repository name>
```
3. Copy new files into the newly created directory via Drag & Drop, Copy & Paste etc.
4.  In the GIN client (terminal) window, navigate into the newly created local workspace by typing `cd <repository name>`. 
5. Upload the new files using

```bash
 gin upload .
```

Note the period at the end of the command. This command will _commit_ your changes. In other words, it will detect the new files in the directory, add them to the repository, and start uploading to the GIN server. Every time you perform a `gin upload .` the changes are saved and uploaded and a _checkpoint_ is made of your data.
You can instead upload individual files or directories by listing them on the command line. For example:    
```bash
    gin upload file1.data recordings/recording1.h5
```
 
 This will upload changes made to two files: `file1.data` and `recording1.h5`, where the latter is in the `recordings` directory.

 Note that _upload_ here doesn't only mean sending new files and changes to the server. This command sends _all_ changes made in the directory to the server, including deletions, renames, etc. Therefore, if you delete files from the directory on your computer and perform a `gin upload`, the deletion will also be sent and the file will be removed from the server as well. Such changes can be synchronized without uploading any new files by not specifying any files or directories.

```bash
    gin upload
```

**Fetch any repository updates from the server**

If changes are made to your data elsewhere, for example on another computer (assuming they were uploaded to the server), or from another user that you share your data with, you can download these changes by typing the download command from within the repository.

```bash
gin download
```

This command will only download changes made to the repository (file deletions, renames, etc.) but any new files are downloaded as placeholders. Placeholder files are empty files that represent files uploaded to the repository but do not hold any of the data. This is useful for downloading the contents of larger files on demand without downloading the entire repository.

If you would like to download _all_ the data contained in a repository, you can do so using the `--content` flag.

```bash
gin download --content
```

This will synchronize the local directory with all changes made on the server and download the content of all files.

**Selective download**

When new data has become available or existing files have been changed on the GIN server, a selected subset of the changes can be downloaded to the local workspace.

1.  Download a summary of the changes on the GIN server using `gin download`. IMPORTANT: This does not download any data. New files and files changed on the GIN server are considered to be "unsynced".
2.  Use `gin ls` to check the sync status of the files in the repository.
3.  Use `gin get-content <file name>` to download the data of a specific file.

**Selective upload**

When new data has been added to or existing files changed in the local workspace, a selected subset of the changes can be uploaded to the GIN server.

1.  Use `gin ls` to check the sync status of the files in the repository.
2.  Use `gin upload <file name>` to only upload the specified new file or changes to the specified existing file.

## 2 DataLad

A good start of [Datalad Handbook](https://handbook.datalad.org/en/latest/index.html).

It assists with the combination of all things necessary in the digital workflow of data and science. 

DataLad only cares (knows) about two things: **Datasets** and **files**. A dataset is a Git repository. A DataLad dataset can take care of managing and version controlling arbitrarily large data. 

DataLad can manage Gnode repository, see [Gin and DataLad](https://handbook.datalad.org/en/latest/basics/101-139-gin.html). 

Basic usage:

```bash
datalad save -m 'save something'
datalad update
datalad push
```

This mirrors the traditional git commands

```bash
git add
git commit
git pull
git push
```

> [!warning] `datalad save` vs `git commit`
> A common pitfall: students try to commit changes to annexed data with plain `git commit -am ...` and end up with an inconsistent state — git tracks the small symlink/pointer files, but git-annex tracks the actual content. **Always use `datalad save -m "msg"`** inside a DataLad dataset. It calls both `git add` and `git annex add` correctly depending on file type, then commits. Use `git commit` only when you're absolutely sure all changes are pure text (e.g. editing a README).

### A typical fMRI workflow with DataLad

The story: clone a study dataset hosted on GNode, fetch only the subjects you need, run a container, save the derivatives, push them back.

```bash
# 1. Clone the dataset skeleton (small — only the git history and pointers)
datalad clone https://gin.g-node.org/<lab>/<study>.git $BIDS_DIR
cd $BIDS_DIR

# 2. Inspect what's there; nothing big has been downloaded yet
ls sub-*

# 3. Fetch only the subjects you'll analyze
datalad get sub-01 sub-02 sub-03

# 4. Run preprocessing in a container; --cleanenv keeps the host env from leaking in
apptainer run --cleanenv \
    -B $BIDS_DIR:/data \
    -B $FMRIPREP_DIR:/out \
    fmriprep.sif /data /out participant --participant-label 01

# 5. Record the derivatives in the dataset
datalad save -m "fMRIPrep derivatives for sub-01" $FMRIPREP_DIR

# 6. Drop large raw files you no longer need locally (they stay on the remote)
datalad drop sub-01/func/*_bold.nii.gz

# 7. Push your changes (git history + annexed content) back to GNode
datalad push --to origin
```

> [!tip] Provenance
> Wrapping commands in `datalad run -m "msg" -- <command>` records the exact call into the dataset history, so anyone who later runs `datalad rerun` reproduces your step. Worth the small extra typing.

### Cluster considerations

On the LRZ cluster (see [[LRZ Cloud Computing]]) and inside containers, two things bite:

- **Annexed files are symlinks** into `.git/annex/objects/`. If you bind-mount only the subject directory into a container, the symlink targets disappear. Bind the dataset root, not a subdirectory:

  ```bash
  apptainer run --cleanenv -B $BIDS_DIR:$BIDS_DIR fmriprep.sif ...
  ```

  This keeps the relative symlinks valid inside the container.

- **Object store on shared storage**: `.git/annex/objects/` can get large. If your home quota is tight, configure git-annex to keep the store on scratch and symlink it in, or use `git annex move` to relocate.

See [[Git Annex]] for the underlying commands these wrappers call.

