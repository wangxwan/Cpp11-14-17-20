# C++11 - Using `using` for Alias Declarations (Replacing `typedef`)

In C++, we can redefine a type using `typedef`:

```c++
typedef unsigned int uint_t;
```

This doesn't create a new type; it merely provides a new name for the existing type. Therefore, the following code would result in a redefinition error:

```c++
void func(unsigned int);
void func(uint_t); // error: redefinition
```

While `typedef` is convenient, it has limitations, such as the inability to redefine a template.

Consider the following scenario:

```c++
typedef std::map<std::string, int> map_int_t;
// …
typedef std::map<std::string, std::string> map_str_t;
// …
```

We essentially want a map with `std::string` keys that can map to either `int` or another `std::string`. However, achieving this with `typedef` alone is cumbersome.

In C++98/03, we often resorted to workarounds like this:

```c++
template <typename Val>
struct str_map
{
    typedef std::map<std::string, Val> type;
};
// ...
str_map<int>::type map1;
// ...
```

This required a wrapper template `str_map`, which, while functional, added unnecessary verbosity and complexity, especially when reusing generic code.

C++11 introduced a new syntax for redefining templates using the `using` keyword. Here's the previous example rewritten using `using`:

```c++
template <typename Val>
using str_map_t = std::map<std::string, Val>;
// ...
str_map_t<int> map1;
```

This code defines `str_map_t` as an alias template for `std::map` with a fixed `std::string` key type. Compared to the previous wrapper template approach, this syntax is much cleaner and resembles defining a new map class template.

In fact, the `using` alias declaration syntax encompasses all functionalities of `typedef`. Let's compare both syntaxes for redefining ordinary types:

```c++
// Redefining unsigned int
typedef unsigned int uint_t;
using uint_t = unsigned int;
// Redefining std::map
typedef std::map<std::string, int> map_int_t;
using map_int_t = std::map<std::string, int>;
```

Both methods achieve the same result, with the only difference being the syntax.

`typedef`'s syntax resembles variable declaration: declare the redefined type like a variable and prepend it with `typedef`. This maintains consistency with C/C++ syntax but can sometimes hinder readability, especially when redefining function pointers:

```c++
typedef void (*func_t)(int, int);
```

In contrast, `using` is followed by the new identifier, and an assignment-like syntax assigns the existing type-id to the new alias:

```c++
using func_t = void (*)(int, int);
```

This comparison highlights the clarity of C++11's `using` alias declaration syntax. `typedef`'s syntax resembles solving an equation, while `using` defines aliases through assignment, aligning better with our natural thought process.

Let's examine another example comparing how both syntaxes define template aliases:

```c++
/* C++98/03 */
template <typename T>
struct func_t
{
    typedef void (*type)(T, T);
};
// Using the func_t template
func_t<int>::type xx_1;
/* C++11 */
template <typename T>
using func_t = void (*)(T, T);
// Using the func_t template
func_t<int> xx_2;
```

The `using` syntax for defining template aliases simply adds the `template` parameter list to the ordinary type alias syntax. This allows for easy creation of new template aliases without the need for wrapper templates like in C++98/03.

It's important to note that `using`, like `typedef`, doesn't create new types. In the example above, the C++11 `using` syntax is equivalent to the `typedef` version. Although `func_t` is a template, `xx_2` is not an instance of a class template but an alias for `void(*)(int, int)`.

Therefore, the following code would still result in a redefinition error:

```c++
void foo(void (*func_call)(int, int));
void foo(func_t func_call); // error: redefinition
```

`func_t` is merely an alias for the `void(*)(int, int)` type.

Astute readers might notice that `func_t` defined with `using` is a template, but it's neither a class template nor a function template (which, when instantiated, results in a function). It represents a new template form: an **alias template**.

In fact, `using` can define template expressions for any type:

```c++
template <typename T>
using type_t = T;
// …
type_t<int> i;
```

Here, `type_t` is an alias template that, when instantiated, results in a type equivalent to its template argument. In this case, `type_t<int>` would be equivalent to `int`.


