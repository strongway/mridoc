---
title: Git Annex
draft: false
tags:
  - git-annex
  - data-management
  - version-control
---

`git-annex` is the layer underneath [[GNode and Datalad|DataLad]]. Most of the time you'll interact with it through DataLad — but knowing the underlying commands helps when something goes sideways. This page is a pragmatic primer, not a full manual.

## 1 Why git-annex for neuroimaging

A plain git repository chokes on large binary files: NIfTIs, DICOMs, derivatives. Cloning a 500 GB study would copy every byte of every revision. git-annex sidesteps this by storing **only a content-addressed pointer** in git — typically a symlink whose name encodes a hash of the file's contents. The actual bytes live in `.git/annex/objects/` and can be fetched on demand from one or more **remotes** (another machine, a server, S3, GIN).

The trade-off:

- **git** is good for small text (code, README, JSON sidecars). Every clone gets everything.
- **git-annex** is good for large binary blobs. Every clone gets the *history* but only the *content* you ask for.

## 2 Setting up an annex

After a normal `git init` (or `git clone`), initialize the annex once:

```bash
git init
git annex init "my-laptop"        # an optional human-readable name for this clone
```

You'll see a new branch called `git-annex` — it stores metadata (which content is where), separate from your working branch.

## 3 Adding files: `git add` vs `git annex add`

The difference is where the **content** lives:

```bash
git add notes.md                  # content goes into git history (good for text)
git annex add sub-01_bold.nii.gz  # content goes into .git/annex/objects; git tracks a symlink
```

After either, you still commit normally:

```bash
git commit -m "Add subject 01"
```

Rule of thumb: small text files → `git add`; everything large or binary → `git annex add`. DataLad's `datalad save` makes this choice for you based on file type and size.

## 4 The unlock / lock lifecycle

Annexed files are stored read-only by default — the working-tree entry is a symlink to a content-addressed object. To edit one:

```bash
git annex unlock data.csv         # replaces the symlink with a writable copy
# ...edit...
git annex add data.csv            # re-locks and ingests the new content
git commit -m "update data.csv"
```

> [!note] v7/v8 repositories
> Modern git-annex repos (v7+) use "adjusted unlocked" mode where files appear as regular writable files (no visible symlinks). The lock/unlock concept still applies under the hood, but day-to-day you'll usually edit files directly and let `git annex add` or `datalad save` re-ingest them.

## 5 Fetching and dropping content

The defining git-annex commands. After cloning, you have pointers but no content:

```bash
git annex get sub-01/             # download content from a remote
git annex get .                   # everything
git annex drop sub-01/func/*_bold.nii.gz   # delete local content; pointer stays
```

`drop` is safe by default — it refuses to remove the last copy of a file. git-annex keeps a count of how many remotes have each blob (`numcopies`, default 1).

To push content to another remote:

```bash
git annex copy --to=origin sub-01/
git annex move --to=archive sub-01/    # copy + drop locally
```

## 6 Remotes and special remotes

A git-annex repo can have multiple **remotes**. Beyond ordinary git remotes (another git-annex-aware repo, e.g. GIN), git-annex adds **special remotes** that store content in non-git backends:

- `datalad` — fetch content from any URL DataLad knows about
- `S3` / `glacier` — Amazon storage
- `rsync` — a plain rsync target
- `directory` — a local or mounted folder (great for an external drive)

Add one:

```bash
git annex initremote backup type=directory directory=/mnt/external/annex encryption=none
git annex copy --to=backup .
```

## 7 Syncing metadata

Pushing content moves bytes; **syncing** moves the metadata (which file lives where) across remotes:

```bash
git annex sync                    # exchange metadata with all remotes
git annex sync --content          # also push/pull content as needed
```

After `sync`, each clone knows the global state — including which remotes hold which files — even if you haven't downloaded the content locally.

## 8 Useful inspection commands

```bash
git annex whereis sub-01/func/*_bold.nii.gz    # which remotes hold this file
git annex info                                  # repo summary, sizes, remotes
git annex find --not --in=here                  # files whose content is missing locally
git annex unused                                # candidate orphans for cleanup
```

## 9 Moving files out of the annex

Occasionally you'll annex something by accident — e.g. a CSV that should live in git history. Reverse it with `unannex`:

```bash
git annex unannex *.csv
git commit -am "Move CSV files from git-annex to git"
git push
```

After this, the CSVs are tracked as ordinary git files (full content in history).

## 10 When to use bare git-annex vs DataLad

- Use **DataLad** for almost everything in this lab: dataset creation, subject-level fetches, provenance tracking, GIN/GNode sync. See [[GNode and Datalad]].
- Drop down to **bare git-annex** when you need: a fine-grained `copy --to`, a special remote DataLad doesn't wrap, troubleshooting a stuck dataset, or simply when you want to see what's actually happening underneath.

See also: [[Basic Linux Commands]] for the shell basics, [[GNode and Datalad]] for the lab's main data-versioning workflow.
