---
title: Basic Linux Commands
draft: false
tags:
  - linux
  - command-line
  - lab
  - server
---

Most lab work — fMRI preprocessing, container runs, dataset transfer — happens on a remote Linux box (workstation or the LRZ cluster). This page is a survival kit: enough shell to navigate, edit configs, move data, manage permissions, and watch processes. Companion pages: [[tmux]] for persistent sessions, [[LRZ Cloud Computing]] for the cluster, [[Tips and Tricks]] for assorted gotchas.

Shells such as Bash, Zsh, and Tcsh provide command-line environments for Unix-like systems, each with slightly different syntax. Bash is the default on most servers; Zsh adds better completion and plugins; Tcsh shows up because AFNI uses it. Stick with Bash unless a tool tells you otherwise.

## 1 Files, directories, and navigation

Print Working Directory (`pwd`) shows the **current directory**.

```bash
pwd
```

**List** the files in a directory (`ls`); `-l` is long format, `-h` is human-readable sizes, `-a` includes hidden dotfiles.

```bash
ls -lah directory
```

To list files recursively, use `ls -R`.

**Change directory** (`cd`):

```bash
cd to/sub/folder
cd ~          # home
cd -          # previous directory
cd ..         # up one level
```

**Copy** (`cp`), **move/rename** (`mv`), **remove** (`rm`):

```bash
cp original duplicate
cp -r folder1 folder2     # recursive copy for directories
mv old_name new_name
rm file.txt
rm -r folder/             # recursive; be careful
```

Make a new directory (`mkdir -p`) or remove an empty one (`rmdir`):

```bash
mkdir -p $BIDS_DIR/derivatives/fmriprep
rmdir empty_folder
```

**Show a text file** in the console with `cat`, or only the **head** / **tail**:

```bash
cat readme.txt
head -n 5 readme.txt      # first 5 lines
tail -n 20 fmriprep.log   # last 20 lines
tail -f fmriprep.log      # follow live (e.g., watch a running job)
```

**Selectively show columns** from a delimited file using `cut`:

```bash
cut -f 2-5,8 -d , exp1.csv
```

Shows columns 2–5 and column 8 from `exp1.csv` (delimiter `,`).

**Symbolic links** (`ln -s`) — useful for pointing into shared datasets without copying:

```bash
ln -s /data/shared/MyStudy $HOME/MyStudy
```

### Inspecting BIDS layouts

`tree -L 2` (if installed) shows a directory two levels deep — handy for sanity-checking a BIDS dataset:

```bash
tree -L 2 $BIDS_DIR
```

If `tree` isn't available, use:

```bash
find $BIDS_DIR -maxdepth 2 -type d
```

Count all BOLD files across subjects:

```bash
find $BIDS_DIR -name "sub-*_bold.nii.gz" | wc -l
```

Size each subject folder (BIDS structure):

```bash
du -sh $BIDS_DIR/sub-*
```

## 2 Pipes, redirection, and wildcards

Store a command's output in a file with `>` (overwrite) or `>>` (append):

```bash
head readme.txt > head.csv
echo "done" >> log.txt
```

Chain commands with the pipe symbol `|`. For example, show the last 5 trials of the first block (48 trials) from `data.csv`:

```bash
head -n 48 data.csv | tail -n 5
```

**Wildcards** in filenames:

- `*` matches any string (e.g. `sub-*_bold.nii.gz`)
- `?` matches a single character (`sub-0?.txt` matches `sub-01.txt` but not `sub-11.txt`)
- `[...]` matches any character inside the brackets (`201[78].txt` matches `2017.txt` or `2018.txt`)
- `{...}` matches comma-separated patterns (`{*.txt,*.csv}`)

**Arithmetic expansion**: `$[]` or `$(())` evaluates the expression. `echo $[3 + 2*4]` outputs `11`.

**Variable expansion**: build strings from variables.

```bash
a=sub
echo ${a}-01.txt    # outputs: sub-01.txt
```

Braces `{}` disambiguate when the variable name sits next to other characters: `echo $ab.txt` looks for the variable `$ab`, whereas `echo ${a}b.txt` outputs `subb.txt`.

## 3 Searching and text processing

### grep — find lines

`grep PATTERN file` prints matching lines. Common flags:

- `-c`: count matches instead of printing
- `-i`: ignore case
- `-l`: list filenames with matches
- `-n`: include line numbers
- `-v`: invert (lines that do **not** match)
- `-r`: recursive into subdirectories

```bash
grep -rni "task-bisection" $BIDS_DIR
```

### wc — counting

`wc` (word count) prints **c**haracters, **w**ords, and **l**ines. `-c`, `-w`, `-l` select just one.

For example, count trials with `curDur = 0.5` from `Exp1.csv` (column 8 is `curDur`):

```bash
cut -f 8 -d , Exp1.csv | grep -v curDur | grep 0.5 | wc -l
```

### sort, uniq

`sort` orders lines; `-r` reverses, `-n` sorts numerically, `-f` is case-insensitive. `uniq` collapses *adjacent* duplicate lines, so always `sort` first. `uniq -c` prefixes each line with its count.

Count trials per sample duration:

```bash
cut -d , -f 8 Exp1.csv | grep -v curDur | sort | uniq -c
```

### sed — find/replace

`sed` (stream editor) does substitutions:

```bash
sed 's/unix/linux/' aText.txt      # first match per line
sed 's/unix/linux/2' aText.txt     # 2nd match per line
sed 's/unix/linux/g' aText.txt     # all matches
sed 's/unix/linux/3g' aText.txt    # from 3rd match to end of line
```

The delimiter `/` can be `|` (handy when replacing paths).

### awk — column-aware tool

`awk` treats each line as columns. One-liner: print column 2 where column 3 equals `0.5`:

```bash
awk -F, '$3 == 0.5 {print $2}' Exp1.csv
```

> [!todo] Task
> Suppose you want to rename all filenames containing `_run` to `_task-bisection_run`. Think about pipes.

> [!solution]-
> ```bash
> ls -R | grep _run | sed 's/_run/_task-bisection_run/'
> ```

## 4 SSH, file transfer, and remote work

### SSH basics

```bash
ssh user@host.lrz.de
```

Generate a key pair (Ed25519 is the modern default):

```bash
ssh-keygen -t ed25519 -C "your.email@lmu.de"
```

This creates `~/.ssh/id_ed25519` (private — never share) and `~/.ssh/id_ed25519.pub` (public — copy to the server). Add the public key to the server's `~/.ssh/authorized_keys`, or use:

```bash
ssh-copy-id user@host
```

### ~/.ssh/config — your friend

Set up short aliases in `~/.ssh/config`:

```
Host lrz
    HostName login.lrz.de
    User di12abc
    IdentityFile ~/.ssh/id_ed25519

Host workstation
    HostName 10.0.1.42
    User mystudent
    IdentityFile ~/.ssh/id_ed25519
```

Now `ssh lrz` is enough. See also [[LRZ Cloud Computing]] for cluster-specific setup.

### scp vs rsync

`scp` is fine for a single file. For datasets — and especially anything that might be interrupted — **use `rsync`**: it resumes, skips unchanged files, and shows progress.

```bash
# single file
scp results.csv user@host:/path/to/

# directory, resumable, with progress
rsync -avz --progress local/directory user@host:/path/to/destination

# pull from remote
rsync -avz --progress user@host:/path/to/directory local/directory
```

Flags: `-a` archive (preserves permissions, timestamps), `-v` verbose, `-z` compress in transit. Add `--dry-run` first to preview.

### wget vs curl

Both download files. `wget` is simpler for "grab this URL into a file"; `curl` is more flexible (REST APIs, custom headers, uploads).

```bash
wget https://example.org/dataset.tar.gz
curl -L -o dataset.tar.gz https://example.org/dataset.tar.gz
```

### screen and tmux

Long jobs over SSH die when your connection drops — unless you run them inside a terminal multiplexer. `screen` is the old standby; **the lab uses `tmux`** (see [[tmux]]).

## 5 Permissions and ownership

Linux permissions cover **read** (`r`), **write** (`w`), **execute** (`x`) for three classes: **u**ser, **g**roup, **o**ther. `ls -l` shows them as `-rwxr-xr--`.

```bash
chmod 600 ~/.ssh/id_ed25519           # private key: only you can read/write
chmod 755 my_script.sh                # owner rwx, group/others rx
chmod -R u+rwX,g+rX,o-rwx /data/private
```

Change ownership / group:

```bash
chown newuser file.txt
chgrp -R labgroup /path/to/shared    # recursively assign group
```

Group ownership matters on shared servers: set the group of a shared dataset to your lab group so collaborators can read it.

## 6 Disk space and process monitoring

### Disk usage

```bash
df -h                          # free space per mount, human-readable
du -sh $BIDS_DIR               # total size of a folder
du -sh */ | sort -hr           # subdirs of cwd, largest first
```

### Processes

```bash
top                            # live process list (everywhere)
htop                           # nicer interactive top (if installed)
ps aux | grep fmriprep         # find a specific process
kill <PID>                     # ask process to quit
kill -9 <PID>                  # force kill (last resort)
```

### GPU

```bash
nvidia-smi                     # GPU usage, memory, processes
watch -n 2 nvidia-smi          # refresh every 2 s
```

## 7 Archiving and compression

Create a gzipped tarball, and extract one:

```bash
tar czvf archive.tar.gz folder/      # c=create, z=gzip, v=verbose, f=file
tar xzvf archive.tar.gz              # x=extract
tar tzvf archive.tar.gz              # t=list contents
```

For single files: `gzip file` / `gunzip file.gz`.

## 8 Editors — minimum survival

You'll edit `.bashrc`, `~/.ssh/config`, job scripts. Two options on every server:

### nano (easy)

```bash
nano ~/.bashrc
```

Bottom of the screen shows the shortcuts. `Ctrl-O` save, `Ctrl-X` exit, `Ctrl-W` search.

### vim (powerful)

```bash
vim ~/.bashrc
```

Modal: starts in **normal mode**. Press `i` to enter **insert mode**, type, then `Esc` back to normal. Save and quit with `:wq`, quit without saving `:q!`. That's enough to survive — invest more time later if you stick with it.

## 9 Where to go next

- [[tmux]] — keep long jobs alive across disconnects
- [[LRZ Cloud Computing]] — accessing and using the LRZ cluster
- [[Tips and Tricks]] — miscellaneous lab knowledge
- [[GNode and Datalad]] — versioned dataset workflows
- [[Neurocommand]] — running neuroimaging containers
