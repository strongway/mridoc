---
title: Basic Linux Commands
draft: false
tags:
  - Command
---
## 1 Basic commands

Shells such as Bash, Zsh, and Tcsh provide powerful command-line environments for Unix-like systems, each with its own syntax and features. While they all allow users to execute multiple commands sequentially or in parallel, their scripting syntax and built-in functionalities differ. For example, Bash offers rich scripting capabilities and is widely adopted for its compatibility and flexibility, while Zsh includes advanced features like improved tab completion and a robust plugin system. Tcsh, known for its C-like syntax, is often preferred for interactive use due to its user-friendly command-line editing features. For example, AFNI uses the tsch shell. 

Print Working Directory (`pwd`) will show you the **current directory**. 
```
pwd
```

**List** the files in the directory (`ls`), with some parameters, such as long format with human readable. 
```
ls -lh  directory
```
If you want to list files Recursively, you can use `ls -R`. 

**Change the Current Directory** (`cd`)
```
cd to/sub/folder
```

**CoPy** files (`cp`)
```
cp original duplicate
```

**rsync** -- a fast, versatile, remote (and local) file-copying tool. You need to have an SSH connection setup. 

```bash
#copy from remote files to local
rsync -rv username@remote:/path/to/directory local/directory

#copy from local to remote
rsync -rv local/directory username@remote:/path/to/destination
```

You can **remove** (`rm`) files, rename (`rm`) files, or move (`mv`) files to elsewhere. 

For directories, you can make a new directory (`mkdir`) or remove directory (`rmdir`).

To quickly **show a text** file in the console, you can `cat` the file. 
```
cat readme.txt
```

If you only want to show the **head** (`head`) or the **tail** (`tail`), you can do 
```
# show the first 5 rows 
head -n 5 readme.txt
```

Similarly, you can **selective show columns** from a table text file using `cut`, but you need to specify the columns (fields, `-f`) and tell the delimiter (`-d`). 
```
cut -f 2-5,8 -d , exp1.csv
```
The above command will show the columns 2 to 5 and column 8 with the delimiter of `,` from the csv file `exp1.csv`. 

To store a command's output in a file `>`, which serves as a pipe line. 
```
head readme.txt > head.csv
```
If you want to append the output to the existing file, you replace `>` with `>>`. 

A general pipe symbol is `|`. For example, list the last 5 trials of the first block (48 trials) from data.csv file,
```
head -n 48 data.csv | tail -n 5
```


## 2 Regular expression 

`grep` takes a piece of text followed by one or more filenames and prints all of the lines in those files that contain specified text or patterns. 

For example, `grep memory readme.txt` prints lines from readme.txt that contain 'memory'. 

`grep` common parameters:

- `-c`: a count of matching lines rather than the lines themselves
- `-i`: ignore case (e.g., treat "Memory" and 'memory' as the same)
- `-l`: print the names of files that contain matches, not the matches
- `-n`: print line numbers for matching lines
- `-v`: invert selection, list not matched lines. 

The command `wc` (short for "word count") prints the number of **c**haracters, **w**ords, and **l**ines in a file. You can make it print only one of these using `-c`, `-w`, or `-l` respectively.

For example, let's suppose we have Exp1.csv file with the column 8 name `curDur`. You want to show how many trials with `curDur = 0.5`:

```
cut -f 8 -d , Exp1.csv | grep -v curDur | grep 0.5 | wc -l 
```

__wildcards__
The most common wildcard is `*`, which mean 'match everything'. There are others:
-   `?` matches a single character, so `sub-0?.txt` will match `sub-01.txt` or `sub-02.txt`, but not `sub-11.txt`.
-   `[...]` matches any one of the characters inside the square brackets, so `201[78].txt` matches `2017.txt` or `2018.txt`, but not `2016.txt`.
-   `{...}` matches any of the comma-separated patterns inside the curly brackets, so `{*.txt, *.csv}` matches any file whose name ends with `.txt` or `.csv`.
* arithmatic expansion. `$[]` or `$(())` will calculate the value inside. For example, `echo $[3 + 2*4]` will output 11. 

__Variable replacements__
You can construct a text string using variables. For example, 
```
a=sub
echo ${a}-01.txt
```
it will output `sub-01.txt`.  Note, if $a is separable from the rest, such as the above example, `{}` can be omitted. Otherwise, it may caused a confusion. For example, 
```
echo $ab.txt  #error
echo ${a}b.txt #output: subb.txt
```

__sorting__

`sort` puts data in order. By default, it in ascending order, but the flags -r can reverse the order (i.e., descending), `-n` to sort numerically, `-b` to ignore leading blanks, and `-f` tells it to fold case (i.e., be case-insensitive). 

__unique__
`uniq` is to remove duplicated lines that are adjacent. So to remove all duplicate lines, you need to sort the data first. With the flag `-c`, it also output the number of duplicates. 

For example, count the number of trials for each sample durations (column 8 `curDur`) from `Exp1.csv` file:
```
cut -d , -f 8 Exp1.csv | grep -v curDur | sort | uniq -c
```

## 3 String manipulations

Manipulating file names is a common task using command lines. For example, we want to substitute a file name with a specified pattern. **S**tream **ed**itor `sed` command can perform many functions on file like searching, finding and replacing, insertion, or deletion. 

**Replacing or substituting string**

```
sed 's/unix/linux' aText.txt
```
It will replace the word `unix` with `linux` in the file. Here, `s` specifies the substitution operation, and the "/" delimiters are used. 

Replacing the nth occurrence of a pattern in a line by adding a number at the end of the expression. For example, 
```
sed 's/unix/linux/2' aText.txt
```
Replacing all the occurrence of the pattern in a line with the flag `/g` (global replacement)
```
sed 's/unix/linux/g' aText.txt
```
Replacing from nth occurrence to all occurrence in a line. Use the combination of number and `g` flag together. 
```
sed 's/unix/linux/3g' aText.txt
```
The delimiter `/` could be `|`. 

>[!todo] Task
>Suppose you want to replace all filenames with '_run' to '_task-bisection_run'. Think about the pipe and commands. 

>[!solution]-
>```
>ls -R | grep _run | sed 's/_run/_task-bisection_run/'
>```

## 4 [[tmux]]
`tmux` is a command-line terminal multiplexer for Unix-like systems. You can control `tmux` using key combinations; you first type a prefix key combination (by default `ctrl + b`) followed by additional command keys.
```
# list existing tmux sessions
$ tmux ls
# create a new session
$ tmux new -s fmriAna
```

To make the connection and remember what you are doing; I recommend using `tmux.` 
Basic usage as follows:

```bash
# list existing tmux sessions
tmux ls

# create a new session
tmux new -s fmriAna

# attach to an existing tmux session
tmux a -t mysession

# terminate an existing tmux session
tmux kill-session -t mysession

# creat a new window (ctrl + b, c)
# navigate windows (ctrl + b, [0-9])
# flip to next window (ctrl + b, n)
# split horizontal (ctrl+b, ")
# split vertical (ctrl+b, %)
```


