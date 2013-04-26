git
================

Basic Usage
------------------

    man gittutorial

To create a new local repository:

    git init      // Initialize a new git repository
    git remote add origin https://github.com/jijoel/project.git

To copy a remote repository to the local machine:

    git clone https://github.com/jijoel/project.git local_project_dir

To copy a specific branch of a remote repository to the local machine in git 1.7.10 or later:

    git clone -b develop --single-branch https://github.com/laravel/laravel.git laravel-develop

To copy a specific branch of a remote repository to the local machine in git 1.7.9 or prior:

    mkdir repo
    git init
    git fetch url-to-repo branchname:refs/remotes/origin/branchname
    

To use git to send files to a remote repository:

    git add *     // Add all files in the path on the local machine to the track list
    git commit    // Commits changes to the local repository
    git push      // Pushes changes to the remote repository
    
To pull working files from the repository (eg, to get updates from other people or other places that have been pushed to the remote repository)

    git pull      // Pull (and merge) changes from the remote repository

or

    git fetch
    git merge

To see what has changed in the working tree

    git status


Advanced Usage
--------------------------------------------------------------
Pull data from a remote repository, wiping out any local changes:

    git reset --hard HEAD                   (resets to the current HEAD)
    git pull

If you have local commits that you'd also like to wipe out:

    git reset --hard origin/master          (resets to the current origin/master commit)
    git pull



git add just adds changes (including in new files)

    git add .             (adds all files in directory)
    git commit -a         (stages all tracked files with modifications)
    git commit -m '<msg>' (commit everything with a comment; don't open editor)

Use a .gitignore file in a directory to ignore files and folders. List each type of file to be ignored. If you need to track specific files among ignored files, use a ! to disregard the ignore, and track those files:

    temp/*.txt              (ignore every .txt file in temp)
    !temp/important.txt     (track important.txt)

Parts of git:

    working directory       uncompressed files
    staging area (index)    lets you choose what to commit; good encapsulated messages
    git repository          

git diff is used to see changes that have been made over time.

    git diff              difference between working directory and staging area
    git diff --staged     shows difference between committed and staged
    git diff HEAD         difference between working dir and HEAD (repository)
    git diff <hash> <file>  changes to <file> since <commit hash>
    
git log shows a history of commits over time

    git log               basic history of commits over time
    git log --stat        includes statistics  (files changed, insertions/deletions)
    git log --oneline     Shows each commit on one line  (hash, title)
    git log --graph       Shows graph of the current branch
    git log --all         Shows changes to all branches
    git log --decorate    Shows which branch the commit points to
    git log --oneline --graph --all --decorate   shows one-line graph of all branches
    git log --pretty='x'  Shows log with formatted output (which can be used elsewhere)
                          (see http://git-scm.com/docs/git-log for params) Examples:
                            %h   abbreviated hash
                            %t   abbreviated tree hash
                            %an  author name
                            %ad  author date
                            %ar  author date (relative)
                            %cn  committer name
                            %s   subject (commit message)
    git log --grep='x'    Limit output to those with log message matching regex

    alias gl='git log --oneline --graph --all --decorate'
    
    
Branches
---------

Master branch is there by default. Work on new, experimental features in new branches. You can switch branches at any time. When you commit, it will commit to the active branch.

    git branch              shows list of branches
    git branch <name>       create a new branch
    git branch -d <name>    delete the named branch
    git checkout <name>     swich to the named branch
    git checkout -b <name>  create and switch to a new branch
    git merge <name>        merge the named branch with the active branch
    git rebase <name>       rewind HEAD to branch point, apply branch changes, 
                            then replay additional changes to HEAD
                            (makes the log linear, so it looks like there was no branching)

    git push -u origin <name>   Push latest commit from branch <name> to origin repository (github)
                                (create the origin repository if it doesn't already exist)
    


Stashes
---------

You can stash changes to pause what you're working on, switch to something else, then resume when you're ready.

    git add .           adds changes to the index
    git stash           Write those changes to a stash, return to original working state
    git stash apply     Bring your stashed work back
    git shash pop       Bring your stashed work back (and delete the stash)
    git stash list      List all stashes
    git shash drop <id> Delete a stash (use after applying it)
    git stash clear     Delete all stashes





Configuration
--------------------------------------------------------------
Use the nano editor for commits

    git config --global core.editor "nano"
    
Syntax highlighting in nano
http://milianw.de/code-snippets/git-commit-message-highlighting-in-nano

Color output:

    git config --global color.ui true

Configuration information is stored in these files:

    ~/.gitconfig          (global for user)
    project/.git/config   (local to project)