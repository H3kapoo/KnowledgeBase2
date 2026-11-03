# Security

### 1. [Generating SSH Key](#generating-ssh-key)
Make sure you have ```ssh-keygen``` bash tool installed. <br/>
Should be there by default on linux systems. <br/>
Standards other than ```rsa``` can be used but it's optional.
Press ENTER on all questions the program asks.
```bash
$ ssh-keygen -t rsa
```
Your public SSH key should be located at: ```~/.ssh/id_rsa.pub```.
To show the contents for later use:
```bash
$ cat ~/.ssh/id_rsa.pub
```

#### References Used
- https://stackoverflow.com/questions/75272888/adding-an-ssh-key-to-github-from-git