# C++11 - The `tuple` Class Template

C++11 introduces a new class template called `tuple`. The most significant feature of `tuple` is its ability to store an arbitrary number of elements with varying types.

`tuple` has various applications. For instance, you can use it to store multiple elements of different types or as a return value for functions that need to return multiple values.

This section provides a detailed explanation of how to use `tuple`.

## Creating `tuple` Objects

`tuple` is essentially a class template defined with variadic template parameters. It's defined in the `<tuple>` header file within the `std` namespace. To use it, include the following:

```c++
#include <tuple>
using std::tuple;
```

There are two common ways to instantiate `tuple` objects: using its constructors or the `make_tuple()` function.

### 1) Constructors

The `tuple` class template provides several constructors, including:

```c++
1) Default constructor
constexpr tuple();
2) Copy constructor
tuple (const tuple& tpl);
3) Move constructor
tuple (tuple&& tpl);
4) Implicit type conversion constructors
template <class... UTypes>
    tuple (const tuple<UTypes...>& tpl); // lvalue reference
template <class... UTypes>
    tuple (tuple<UTypes...>&& tpl);      // rvalue reference
5) Constructor supporting initializer lists
explicit tuple (const Types&... elems);  // lvalue references
template <class... UTypes>
    explicit tuple (UTypes&&... elems);  // rvalue references
6) Constructor converting from `pair`
template <class U1, class U2>
    tuple (const pair<U1,U2>& pr);       // lvalue reference
template <class U1, class U2>
    tuple (pair<U1,U2>&& pr);            // rvalue reference
```

Here's an example:

```c++
#include <iostream>     // std::cout
#include <tuple>        // std::tuple
using std::tuple;

int main() {
    std::tuple<int, char> first;                             // 1)   first{}
    std::tuple<int, char> second(first);                     // 2)   second{}
    std::tuple<int, char> third(std::make_tuple(20, 'b'));   // 3)   third{20,'b'}
    std::tuple<long, char> fourth(third);                    // 4) lvalue, fourth{20,'b'}
    std::tuple<int, char> fifth(10, 'a');                    // 5) rvalue, fifth{10.'a'}
    std::tuple<int, char> sixth(std::make_pair(30, 'c'));    // 6) rvalue, sixth{30,''c}
    return 0;
}
```

### 2) `make_tuple()` Function

We used the `make_tuple()` function in the previous example. It's a function template defined in the `<tuple>` header file. It creates a `tuple` rvalue object (or temporary object).

We can use the `tuple` object created by `make_tuple()` as an argument to the move constructor, or like this:

```c++
auto first = std::make_tuple(10, 'a');   // tuple<int, char>
const int a = 0; int b[3];
auto second = std::make_tuple(a, b);     // tuple<int, int*>
```

This code creates two `tuple` objects, `first` and `second`, whose types are automatically deduced using `auto`.

## Common `tuple` Functions

The `tuple` class template provides a useful member function, and the `<tuple>` header file offers several function templates and class templates for working with `tuple` objects, as summarized in Table 1.

| Function/Class Template | Description |
|---|---|
| `tup1.swap(tup2)`<br>`swap(tup1, tup2)` | Swaps the contents of two `tuple` objects (`tup1` and `tup2`) of the same type. `tuple` has a `swap()` member function, and `<tuple>` provides a global `swap()` function. |
| `get(tup)` | Returns a reference to the element at the specified index (`num`) in the `tuple` object (`tup`). This is a global function in `<tuple>`. |
| `tuple_size::value` | `tuple_size` is a class template in `<tuple>` with a single member variable `value`. It retrieves the number of elements in a `tuple` object of the specified type. |
| `tuple_element<I, type>::type` | `tuple_element` is a class template in `<tuple>` with a single member variable `type`. It retrieves the type of the element at the specified index (`I`) in a `tuple` object of the specified type. |
| `forward_as_tuple<args...>` | Creates a `tuple` object containing the elements in `args...` as rvalue references. |
| `tie(args...) = tup` | Unpacks the elements of the `tuple` object (`tup`) and assigns them to the variables in `args...`. `tie()` is provided in `<tuple>`. |
| `tuple_cat(args...)` | Concatenates multiple `tuple` objects in `args...` into a single `tuple` object. This function is provided in `<tuple>`. |

> The `tuple` class template overloads the assignment operator `=` for direct assignment between `tuple` objects of the same type. It also overloads comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) for comparing `tuple` objects element-wise.

The following program demonstrates some of the functions and class templates listed in Table 1:

```c++
#include <iostream>
#include <tuple>

int main() {
    int size;

    // Create a tuple object storing 10 and 'x'
    std::tuple<int, char> mytuple(10, 'x');

    // Calculate the number of elements in mytuple
    size = std::tuple_size<decltype(mytuple)>::value;

    // Output the elements stored in mytuple
    std::cout << std::get<0>(mytuple) << " " << std::get<1>(mytuple) << std::endl;

    // Modify a specific element
    std::get<0>(mytuple) = 100;
    std::cout << std::get<0>(mytuple) << std::endl;

    // Create a tuple object using make_tuple()
    auto bar = std::make_tuple("test", 3.1, 14);

    // Unpack the bar object into mystr, mydou, and myint
    const char* mystr = nullptr;
    double mydou;
    int myint;

    // Use tie() to unpack elements. Use std::ignore to discard unwanted values.
    std::tie(mystr, mydou, myint) = bar;
    // std::tie(std::ignore, std::ignore, myint) = bar;  // Only receive the 3rd integer value

    // Concatenate elements from mytuple and bar into a new tuple object
    auto mycat = std::tuple_cat(mytuple, bar);
    size = std::tuple_size<decltype(mycat)>::value;
    std::cout << size << std::endl;

    return 0;
}
```

Program output:

```
10 x
100
5
```


