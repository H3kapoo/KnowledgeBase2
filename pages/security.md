# Security

### 1. [Generating SSH Key](#generating-ssh-key)
Make sure you have ```ssh-keygen``` cli tool installed. <br/>
Should be there by default on linux systems. <br/>
Standards other than ```rsa``` can be used but it's optional.
Press ENTER on all questions the program asks.
```cli
$ ssh-keygen -t rsa
```
Your public SSH key should be located at: ```~/.ssh/id_rsa.pub```.
To show the contents for later use:
```cli
$ cat ~/.ssh/id_rsa.pub
```