# Git / Github

### 1. [Status of a local repo:](#status-of-a-local-repo)
```cli
$ git status
```

### 2. [Committing the changes to the tree (not remotely):](#committing-changes-locally)
```cli
$ git commit -m "for inline messages"
OR
$ git commit -F message_file_path_for_complex_message
```

### 3. [Push to remote repo (github usually):](#push-to-remote-repo)
```cli
$ git push -u <remote_name> <branch_name>
```
Example:
```cli
$ git push -u origin master
```

### 4. [Configuring local git details globally:](#configuring-local-git-globally)
```cli
$ git config --global user.email "email_here"
$ git config --global user.name "github_name"
```

### 5. [Adding a new remote (or modifying one):](#adding-new-remote)
Add:
```cli
$ git remote add <remote_name> <remote_path>
```
Modify:
```cli
$ git remote set-url <remote_name> <remote_path>
```
Example:
```cli
$ git remote set-url origin https://github.com/<github_name>/<reponame>.git
```
The url can a be ```https``` or ```ssh``` path. <br/>
SSH is preferrabe as it doesn't require to input the password for each push. <br/>
Setting SSH keys is addressed in [Security](/pages/security.md#generating-ssh-key).

### 6. [Adding SSH Key to github:](#adding-ssh-key-to-github)
Go to ```Profile -> Settings -> SSH and CPG Keys -> New SSH Key``` and paste the contents of the generated .pub SSH Key.

#### Test connection:
```cli
$ ssh -T git@github.com
```
#### Expected output:
```cli
$ The authenticity of host 'github.com (140.82.121.3)' can't be established.
ED25519 key fingerprint is: SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU
This key is not known by any other names.
```
When asked answer ```yes``` and all should be good.
Now you can push directly via SSH.

> **_NOTE_**: Your remote url needs to be the SSH one for this to work, **not** the HTTPS one.

### [Merge vs Rebase](#merge-vs-rebase)
Merge will just create a new commit on top of the destination branch and will contain the changes from the commits of the source branch. <br/>
The destination branch knows about the source branch and where the merge occured. <br/>
Merging doesn't keep the history linear.

```cli
(1) A -> B -> C -> F1
                   ^
                   | merge
                   |
(2)      D -> E -> F

Suppose (1) is main branch and (2) a feature branch.
Merging (2) into (1) will create a new commit on top of (1) aka (F1) that contains all the changes of
that feature branch.
```

Rebase will try to replay all the commits from the source branch on top of the destination branch and new commit hashes will be generated for all commits. <br/>
The destination branch doesn't know anymore about the source branch, nothing to link it back to it. <br/>
Rebasing keeps the history linear.
```cli
(1) A -> B -> C
(2)      D -> E -> F
(1+2) A -> B -> C -> D1 -> E1 -> F1

Suppose (1) is main branch and (2) a feature branch.
Rebasing (2) into (1) will replay the changes on top of (1) and create a new commit hashes (D1, E1, F1)
for each commit in (2) resulting in (1+2).
```