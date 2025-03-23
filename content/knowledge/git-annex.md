---
title: git-annex
draft: false
tags:
---
## git annex tips

To ensure that your CSV files are managed by Git and not by `git-annex`, you can use the `git annex unannex` command to convert files that are currently managed by `git-annex` back to being managed by Git. After using `unannex`, you will commit the change to record the fact that these files are no longer managed by `git-annex`.

Here's how you can do it:

1. **Navigate to your repository's directory** in the terminal.
    
2. **Unannex the CSV files** you want to be managed by Git. You can do this for individual files or for all CSV files in a directory. Here's how you do it for all CSV files in the current directory:

    
    `git annex unannex *.csv`
    
    If your CSV files are spread out in different directories or you want to be more selective, you'll need to adjust the command accordingly.
    
3. **Commit the change** to record that these files are no longer managed by `git-annex`:
    
    `git commit -am "Move CSV files from git-annex to Git management"`
    
4. **Push the changes** to your remote repository:
        
    `git push`
