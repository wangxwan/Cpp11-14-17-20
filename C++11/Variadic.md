# C++11 - Variadic Templates and Functions

Variadic parameters refer to parameters whose number and types can vary. While we often associate parameters with functions, C++ templates (both function and class templates) also utilize parameters.

C++ has always supported variadic function parameters. The classic example is the `printf()` function:

```c++
int printf ( const char * format, ... );
```

The `...` represents variadic parameters, meaning `printf()` can accept any number of arguments with potentially different types:

```c++
printf("%d", 10);
printf("%d %c", 10, 'A');
printf("%d %c %f", 10, 'A', 1.23);
```

We often refer to this collection of arguments as a **parameter pack**. The `format` string helps `printf()` determine the number and types of arguments within the pack.

Here's a simple example of a custom variadic function:

```c++
#include <iostream>
#include <cstdarg> // for variadic functions

void vair_fun(int count, ...) {
    va_list args;
    va_start(args, count);
    for (int i = 0; i < count; ++i) {
        int arg = va_arg(args, int);
        std::cout << arg << " ";
    }
    va_end(args);
}

int main() {
    // Variadic arguments: 10, 20, 30, 40
    vair_fun(4, 10, 20, 30, 40);
    return 0;
}
```

The `vair_fun()` function takes two parameters: `count` and the variadic `...`. We can easily use `count` inside the function, but accessing the parameter pack requires three macros from `<cstdarg>`: `va_start`, `va_arg`, and `va_end`:

- `va_start(args, count)`: Initializes `args` (a `va_list` variable, essentially a `char*`) to point to the beginning of the variadic arguments based on the last named parameter, `count`.

- `va_arg(args, int)`: Retrieves the next argument from the parameter pack, interpreting it as an `int`.

- `va_end(args)`: Cleans up the `args` variable after processing the parameter pack.

> Note: `va_arg` doesn't inherently know when to stop. We use `count` to control the loop and retrieve all arguments. Other methods, like stopping at a sentinel value, are also possible.

Here are some points to remember when using `...` for variadic parameters:

1. `...` must be the last parameter, and a function can have only one.

2. At least one named parameter must precede `...`.

3. When the parameter pack includes `char`, `va_arg` should read it as `int`. Similarly, for `short`, use `double` with `va_arg`.

Importantly, `...` for variadic parameters only applies to function parameters, not template parameters. C++11 introduces a way to achieve variadic template parameters.

## Variadic Templates

Before C++11, function and class templates could only have a fixed number of template parameters. C++11 extends templates to accept any number of arguments, known as **variadic templates**.

### 1) Variadic Function Templates

Here's how to define a variadic function template:

```c++
template<typename... T>
void vair_fun(T... args) {
    // Function body
}
```

`typename... T` (or `class... T`) declares `T` as a variadic template parameter, accepting multiple types. This is called a **template parameter pack**. Inside `vair_fun()`, `T... args` declares `args` as a **function parameter pack**, capable of holding any number of arguments.

This allows us to instantiate `vair_fun()` with any number of arguments of any type:

```c++
vair_fun();
vair_fun(1, "abc");
vair_fun(1, "abc", 1.23);
```

The challenge lies in "unpacking" the parameter pack within the template function. Here are two common approaches:

**Recursive Unpacking**

```c++
#include <iostream>
using namespace std;

// Base case for recursion
void vir_fun() {}

template <typename T, typename... Args>
void vir_fun(T argc, Args... argv) {
    cout << argc << endl;
    // Recursively call with remaining arguments
    vir_fun(argv...);
}

int main() {
    vir_fun(1, "https://github.com/wangxwan", 2.34);
    return 0;
}
```

Output:

```
1
https://github.com/wangxwan
2.34
```

Let's analyze the execution flow:

- `main()` calls `vir_fun()`. The compiler deduces `T` as `int`, `argc` as `1`, and the remaining arguments go into `argv`.

- Inside `vir_fun()`, we print `argc` (value: 1) and recursively call `vir_fun()` with the remaining arguments from `argv`.

- This continues until `vir_fun()` is called with an empty `argv`, hitting the base case and ending the recursion.

> Always provide a base case to terminate recursion.

**Non-Recursive Unpacking**

We can unpack the parameter pack using comma expressions and initializer lists:

```c++
#include <iostream>
using namespace std;

template <typename T>
void display(T t) {
    cout << t << endl;
}

template <typename... Args>
void vir_fun(Args... argv) {
    // Comma expression + initializer list
    int arr[] = { (display(argv), 0)... };
}

int main() {
    vir_fun(1, "https://github.com/wangxwan", 2.34);
    return 0;
}
```

The key is line 13. The initializer list `{(display(argv), 0)...}` expands to `{(display(1), 0), (display("https://github.com/wangxwan"), 0), (display(2.34), 0)}`. Each element is a comma expression that calls `display()` with the argument and discards the result, ultimately initializing the `arr` array with zeros. The array's purpose is solely to facilitate unpacking.

### 2) Variadic Class Templates

C++11 also allows variadic template parameters in class templates. The standard library's `tuple` is a prime example:

```c++
template <typename... Types>
class tuple;
```

Unlike classes with fixed template parameters, `tuple` can be instantiated with any number and types of arguments:

```c++
std::tuple<> tp0;
std::tuple<int> tp1 = std::make_tuple(1);
std::tuple<int, double> tp2 = std::make_tuple(1, 2.34);
std::tuple<int, double, string> tp3 = std::make_tuple(1, 2.34, "https://github.com/wangxwan");
```

Here's an example of a custom variadic class template:

```c++
#include <iostream>

// Declare the template class 'demo'
template<typename... Values> class demo;

// Base case for inheritance-based recursion
template<> class demo<> {};

// Unpack using inheritance
template<typename Head, typename... Tail>
class demo<Head, Tail...> : private demo<Tail...> {
public:
    demo(Head v, Tail... vtail) : m_head(v), demo<Tail...>(vtail...) {
        dis_head();
    }
    void dis_head() { std::cout << m_head << std::endl; }
protected:
    Head m_head;
};

int main() {
    demo<int, float, std::string> t(1, 2.34, "https://github.com/wangxwan");
    return 0;
}
```

The `Tail` parameter pack in the `demo` template is unpacked using "recursion + inheritance." When `demo<Head, Tail...>` is instantiated, it inherits from `demo<Tail...>`, which in turn inherits from `demo<Tail...>` with one less type, and so on, until the base case `demo<>` is reached.

Output:

```
https://github.com/wangxwan
2.34
1
```

Variadic class templates offer other unpacking techniques as well. This example provides a starting point for further exploration.
