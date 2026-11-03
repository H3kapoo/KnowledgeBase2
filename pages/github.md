# Git

1. Status of a local repo:
```cli
$ git status
```

2. Commiting the changes to the tree (not remotely):
```cli
$ git commit -m "for inline messages"
OR
$ git commit -F message_file_path_for_complex_message
```

3. Push to remote repo (github usually):
```cli
$ git push -u <remote_name> <branch_name>
```
Example:
```cli
$ git push -u origin master
```

[bla bla](#foo)
4. Configuring local git details globally:
```cli
$ git config --global user.email "email_here"
$ git config --global user.name "github_name"
```

5. Adding a new remote (or modifying one):
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
The url can a be ```https``` or ```ssh``` path. SSH is preferrabe as it doesn't require to input the password for each push. Setting SSH keys is addressed in [Security](/pages/github.md/foo).



# Github