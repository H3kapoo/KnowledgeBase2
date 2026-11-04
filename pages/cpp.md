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
    // Note:: Values..., I actually concatenates I to Values
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

### 6. [Concepts and requries](#concepts-and-requires)
A simple custom concept that requires T to be integral and that the result of adding 2 of the same types returns the same type.
Nesting concepts requires the use of ```requires``` keyword otherwise it will not work as intended.
```cpp
// .hpp
template<typename T>
concept MyConcept = requires(T t)
{
    requires std::integral<T>;
    {t + t} -> std::same_as<T>;
};

template<MyConcept T>
void someFunc(T t) { /*...*/ }
```
Chaining concepts (useful when only the type is needed):
```cpp
// .hpp
template<typename T>
concept MyConcept = std::integral<T> && !std::floating_point<T>;

```

### 7. [Static/Extern/Inline keyword](#static-keyword)
Internal linkage means the variable/function is visible only in the TU is has been included, can have multiple copies. <br/>
External linkage means the variable/function can be shared by multiple TUs. <br/>
Static storage time means the variable exists for the entire program execution. <br/>
Static initialization of a variable means the program already knows the value of the static and doesn't need to execute code at runtime. <br/>
Dynamic initialization means the variable, even if it's a static, cannot have it's value determined before runtime. <br/>
Simple static usage:
```cpp
// .hpp
static int x = 3; // static storage, static initialization, internal linkage
```
Multiple copies per TU:
```cpp
// .hpp (static_hpp)
static int x; // this is a defintion at global namespace, static storage/init and internal linkage

// .cpp
#include "static_hpp"
static int x; // this is here behind the scenes after importing
x = 3; // this errors because it's a redefinition

// .cpp
#include "static_hpp"
void foo()
{
    x = 3; // this works, copy 1
}

// .cpp_2
#include "static_hpp"
void bar()
{
    x = 3; // this also works, but it's a 2nd independent from copy 1
}
```
One variable global for each TU:
```cpp
// .hpp (extern_hpp)
extern int x; // static storage (global), static init, external linkage

#include "extern_hpp"
void foo()
{
    x = 3; // this works, no copy
}

// .cpp_2
#include "extern_hpp"
void bar()
{
    x = 3; // this also works, no copy, modifies the same memory
}
```
Static variables with dynamic initialization:
```cpp
// .hpp
/* Some ordinary runtime function */
int foo()
{
    int sum = 0;
    for (auto v : someVector)
    {
        sum += v;
    }
    return sum;
}

static int x = foo(); // static storage, dynamic init, internal linkage, foo() needs runtime exec
```
Static variables with static initialization:
```cpp
// .hpp
/* Some ordinary constexpr/consteval function, assume it will be run at compile time */
constexpr int foo() { /* Same body as before*/ }
/* Or */
consteval int foo() { /* Same body as before*/ }

static int x = foo(); // static storage, STATIC init, internal linkage, foo() will run at compile time
```
Static keyword inside functions will allocate one variable no matter how many times the function is called:
```cpp
// .hpp
int func()
{
    static int x = 0;
    return ++x;
}

func(); // 1
func(); // 2
func(); // 3
```
This can be used with templates and one static variable will be created per unique template combination:
```cpp
// .hpp
template<typename T>
int func() {/* Same body as above */}

func<int>(); // 0
func<int>(); // 1
func<float>(); // 0
```
Static variables inside classes (only one instance will exist per class):
```cpp
// .hpp
struct Test
{
    static int x; // declaration
}

// .cpp
int Test::x = 0; // definition. static storage, static initialization, external linkage by default
```
Since C++17 ```inline``` can be used:
```cpp
// .hpp
struct Test
{
    inline static int x = 0; // declaration + init at the same time
}
```
Using ```inline``` at global namespace gives the variable external linkage:
```cpp
// .hpp (inline_hpp)
inline int x = 3;

// .cpp_1
#include "inline_hpp"
void foo()
{
    x = 1; // x overriden with 1
}

// .cpp_2
#include "inline_hpp"
void foo()
{
    x = 2; // same x overriden with 2
}
```

### 8. [constexpr/consteval keyword](#static-keyword)
On short, ```constexpr``` says that a variable/function can be evaluated at compile time, but not required to. <br/>
On the other hand, ```consteval``` says the variable/function must run/be determined at compile time.
```cpp
// .hpp
constexpr int foo()
{
    return 45;
}
constexpr int x = foo(); // most propably will be resolved at compile time

constexpr int bar()
{
    return /* Some complicated logic that cannot be resolved now */
}
constexpr int x = bar(); // most propably will NOT be resolved at compile time
```
```cpp
consteval int foo()
{
    return 45;
}
constexpr int x = foo(); // needs to be resolved at compile time

consteval int bar()
{
    return /* Some complicated logic that cannot be resolved now */
}
constexpr int x = bar(); // this will fail as bar() cannot be evaluated at compiletime

```
## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g
- https://en.cppreference.com/cpp/utility/integer_sequence