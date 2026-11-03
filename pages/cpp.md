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
};

// .cpp
template<typename T>
void myFunc() {/* Actual definition */ }

template A::myFunc<int>();
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

template<>
struct B<int, float>{} // specific case when base case T = int and U = float 
```

### 4. [Variadic templates](#variadic-templates)
Templates can take an arbitray number of types/NTTP (non-template type parameter) using the spread operator.
```cpp
// .hpp
template<typename... Ts>
struct A
{
    std::tuple<Ts...> allTypes;
};
```
The number of arguments passed can be queried via the ```sizeof``` operator:
```cpp
// .hpp
template<typename... Ts>
struct B
{
    static const std::size_t size = sizeof...(Ts);
};
```

### 5. [NTTP sequence generator](#nttp-sequence-generator)
Suppose you want to generate at compile time a template like ```Type<0, 1, 2, 3, 4>``` which can be used with the ```comma``` operator for fold expressions.
This could be one of the many possible implementations:
```cpp
// .hpp
template<std::size_t... Values>
struct Seq{};

template<std::size_t I, std::size_t N, std::size_t... Values>
struct GenerateSeqHelper
{
    using type = typename GenerateSeqHelper<I+1, N, Values..., I>::type;
};

template<std::size_t N, std::size_t... Values>
struct GenerateSeqHelper<N, N, Values...>
{
    using type = Seq<Values...>;
};

template<std::size_t N>
struct GenerateSeq
{
    using type = typename GenerateSeqHelper<0, N>::type;
};

template<std::size_t... Is>
void show(Seq<Is...>)
{
    (...,(std::cout << "value " << Is << "\n"));
}

// for printing only
template<std::size_t... Is>
void show(Seq<Is...>)
{
    (...,(std::cout << "value " << Is << "\n"));
}

template<std::size_t N>
using GenSeq = typename GenerateSeq<N>::type;

int main()
{
    Seq s = GenSeq<4>{}
    show(s);
}
```
Or simply use the already existing functions:
```cpp
// .hpp
#include <utility>
auto seq = std::make_index_sequence<N>{};
auto seq = std::make_integer_sequence<int, N>{};
auto seq = std::integer_sequence<3,4,1,2>{};
auto seq = std::index_sequence_for<Type_1, Type_2, Type_3>{};
```


## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g
- https://en.cppreference.com/cpp/utility/integer_sequence