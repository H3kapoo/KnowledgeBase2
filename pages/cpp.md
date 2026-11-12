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

### 6. [Concepts and requires](#concepts-and-requires)
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

### 9. [Unique set of template parameters](#unique-set-of-template-parameters)
```cpp
// .hpp
template<typename... Ts>
struct allUnique; // base template

template<typename FirstType, typename... OtherTypes>
struct allUnique<FirstType, OtherTypes...>
{
    // This uses fold expressions, it copies what's inside the ( ) for each OtherType
    // Ex for <int, float, double> :
    // FirstType = int, OtherTypes = float, double
    // (!std::same_as<int, float> && ((!std::same_as<int, double>)))
    static constexpr bool value = (!std::same_as<FirstType, OtherTypes> && ...)
                                    && allUnique<OtherTypes...>::value;
};

template <>
struct allUnique<> // base case when no types are left
{
    static constexpr bool value = true;
};

template<typename... Ts>
concept UniqueConcept = allUnique<Ts...>::value; // define a custom concept for this

template<UniqueConcept Ts> // use it either like this
struct myStruct{};

template<typename... Ts> requires allUnique<Ts...>::value // or like this
struct myStruct{};

int main()
{
    myStruct<int, float> // ok
    myStruct<int, int> // doesn't compile
}
```

### 10. [X Macros](#x-macros)
Those are macros that take other macros as parameters basically.
```cpp
#define MY_PROPERTIES(X_MACRO)\
    X_MACRO(int, myInt)\
    X_MACRO(float, myFloat)\
    X_MACRO(std::string, myString)\

#define POPULATE_PREDICATE(type, name) type name;

struct MyStruct
{
    // this..
    MY_PROPERTIES(POPULATE_PREDICATE)

    // will expand to this..
    int myInt;
    float myFloat;
    std::string myString;
}
```

### 11. [Memory Management Objects](#memory-management-objects)
Particularly ```unique_ptr```, ```shared_ptr``` and ```weak_ptr```. <br/>
These help us automatically manage the memory, more specifically free-ing the allocated memory automatically instead of
having to call ```delete``` each time and potentially forgetting to do so and thus creating memory leaks.

1. ```unique_ptr``` is used when the ownership is strict, only one object can reference the managed memory:
```cpp
#include <memory>

std::unique_ptr<int> ptr = std::make_unique<int>();
//^ 'ptr' now manages the memory of the allocated object of type 'int'

std::unique_ptr<int> ptr_copy = ptr;
// ^ this is illegal, the memory is managed by 'ptr' already and it cannot be managed by 'ptr_copy'

std::unique_ptr<int> ptr_copy = std::move(ptr);
// ^ this is OK because ptr now transfers it's ownership to 'ptr_copy'. 'ptr' remains in an empty but
//  defined state (will point to nothing)

```

2. ```shared_ptr``` is used when ownership isn't strict, aka more objects can reference the same memory:
```cpp
#include <memory>

std::shared_ptr<int> ptr = std::shared_ptr<int>();
//^ 'ptr' now manages the memory of the allocated object of type 'int'

std::shared_ptr<int> ptr_copy = ptr;
// ^ this is OK, the memory is managed by 'ptr' and by 'ptr_copy'

```
<br/>

```shared_ptr``` uses reference counting, imagine a small structure keeping track of how many ```shared_ptr```s there are
pointing to a particular memory address. <br/>
Reference count increases when the ```shared_ptr``` is copied and it decreases when it goes out of scope. <br/>
A reference count of ```zero``` means the underlying object's destructor will be called and the underlaying object will be deleted. <br/>
Suppose:
```cpp
#include <memory>

int main()
{
    std::shared_ptr<int> ptr = std::shared_ptr<int>(); // 1 reference to int address
    {
        std::shared_ptr<int> ptr_copy = ptr; // 2 references to int address
        {
            std::shared_ptr<int> ptr_copy_2 = ptr_copy; // 3 references to int address
        } // reference count drops to 2
    } // reference count drops to 1
} // reference count drops to 0, nobody points to that `int` anymore, call delete

```

```unique_ptr``` cannot be used with ```shared_ptr``` as that will defeat the purpose of the first.

3. ```weak_ptr``` goes together with ```shared_ptr``` and solves the problem of cyclic references. It is a non-owning
view of the managed object (doesn't increase reference count):
```cpp
#include <memory>

std::shared_ptr<int> ptr = std::shared_ptr<int>();
//^ 'ptr' now manages the memory of the allocated object of type 'int'; reference count is 1

std::weak_ptr<int> weak_ptr_copy = ptr;
// ^ this is OK, the memory is managed by 'ptr' and `weak_ptr_copy` 
// just "remembers this", it doesn't increase ref count

std::shared_ptr<int> concrete_ptr = weak_ptr_copy.expired() ? nullptr : weak_ptr_copy.lock();
// ^ later we can lock the weak ptr to get the shared_ptr out of it (if it's still alive, ref > 0).
// doing this increases ref count by 1 since now `concrete_ptr` also points to `ptr` managed memory.

```

```weak_ptr``` can solve cyclic references because in those cases ```shared_ptr``` has it's reference count in such
a way that it never reaches zero. <br/>
Suppose:

```cpp
struct Tree
{
    std::shared_ptr<Tree> parent;
    std::shared_ptr<Tree> child;
};

int main()
{
    // RC = ref count
    std::shared_ptr<Tree> parent = std::make_shared<Tree>(); // parent_RC = 1
    std::shared_ptr<Tree> child = std::make_shared<Tree>(); // child_RC = 1

    parent.parent = nullptr; // parent has no parent
    parent.child = child; // child_RC + 1 => 2
    child.parent = parent; // parent_RC +1 => 2

} // now we go out of scope with everything

/*
1. 'child' goes out of scope, RC decreases by 1, 2-1 = 1, it doesn't get deallocated (not zero)
     > 'parent' keeps a reference to it in parent.child
2. 'parent' goes out of scope, RC decreases by 1, 2-1 = 1, it doesn't get deallocated (not zero)
     > 'child' keeps a reference to it in child.parent

No object got deallocated since no RC reached zero.
*/
```
Now suppose the correct version which uses ```weak_ptr```:
```cpp
struct Tree
{
    std::weak_ptr<Tree> parent;
    // ^ keep a non-owning reference to the parent
    std::shared_ptr<Tree> child;
};

int main()
{
    // RC = ref count
    std::shared_ptr<Tree> parent = std::make_shared<Tree>(); // parent_RC = 1
    std::shared_ptr<Tree> child = std::make_shared<Tree>(); // child_RC = 1

    parent.parent = nullptr; // parent has no parent
    parent.child = child; // child_RC + 1 => 2
    child.parent = parent; // parent_RC REMAINS 1 (it's a weak_ptr to a shared_ptr)

    if (std::shared_ptr<Tree> locked = child.parent.lock(); locked) // parent_RC + 1 => 2
    {
        /* ... */
    } // parent_RC - 1 => 1

} // now we go out of scope with everything

/*
1. 'child' goes out of scope, RC decreases by 1, 2-1 = 1, it doesn't get deallocated (not zero)
     > 'parent' keeps a reference to it in parent.child
2. 'parent' goes out of scope, RC decreases by 1, 1-1 = 0, it gets deallocated
     > 'child' gets deallocated as well since 'parent' no longer exists to keep a reference to it

All objects reached RC zero and got deallocated.
*/
```

## References Used
- https://stackoverflow.com/questions/11946294/dump-include-paths-from-g
- https://en.cppreference.com/cpp/utility/integer_sequence
- https://www.youtube.com/watch?v=6i95i_X2u64