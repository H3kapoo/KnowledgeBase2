# Linux
### 1. [Change file permissions](#change-file-permissions)
```r - readable```
```w - writeable```
```x - executable```

```bash
$ chmod +x/+r+/+w/-x/-w/-r <file-path>
```
Most common use is to make a shell file executable:
```bash
$ chmod +x <file-path>
```