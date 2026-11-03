# C/C++
### 1. [Show include paths used by the compiler](#show-include-paths)
```bash
$ <compiler> -v -E -x <lang> -
```
Example:
```bash
$ g++ -v -E -x c++ -
```

## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g