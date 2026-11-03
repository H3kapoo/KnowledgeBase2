# C/C++
### 1. [Show include paths used by the compiler](#show-include-paths)
```cli
$ <compiler> -v -E -x <lang> -
```
Example:
```cli
$ g++ -v -E -x c++ -
```

## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g