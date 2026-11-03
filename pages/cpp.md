# C/C++
### 1. [Show include paths used by the compiler](#show-include-paths)
```bash
$ <compiler> -v -E -x <lang> -
```
Example:
```bash
$ g++ -v -E -x c++ -
```

### 2. [Template explicit instantiation](#template-explicit-instantiation)
In C++ a template can only be defined in a ```.cpp``` file and not in the header **unless** there's an explicit instantiation of that template in the corresponding TU (translation unit).
```cpp
// .hpp
struct A
{
    template<typename T>
    void myFunc();
}

// .cpp
template A::myFunc<int> {/* Actual definition */ }
```

### 3. [Template specialization](#template-specialization)
Templates can be fully specialized or partially specialized and the compiler will try to deduct what fits best.
```cpp
// .hpp
template<typename T>
struct A{}; // base case

struct A<int>{}; // specific for ints

template<typename T, typename U>
struct B{} // base case

template<typename X>
struct B<int, X>{} // specific case when base case T = int and U = X 

```

## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g