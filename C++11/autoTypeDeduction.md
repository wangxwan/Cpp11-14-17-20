## C++11 - auto Type Deduction

In versions of C++ prior to C++11 (C++98 and C++03), it was mandatory to specify the type of a variable, such as `int`, `char`, etc., before defining or declaring it. However, in more flexible languages like C#, JavaScript, PHP, and Python, programmers can define variables without explicitly stating their types, allowing the compiler (or interpreter) to infer them. This makes coding more convenient.

C++11 introduced support for automatic type deduction to align with this trend, using the `auto` keyword.

**Syntax and Rules of auto Type Deduction**

In earlier C++ versions, the `auto` keyword was used to indicate a variable's storage duration. It contrasted with the `static` keyword, with `auto` implying automatic storage, which was the compiler's default behavior. This made the `auto` keyword somewhat redundant, as it was rarely used explicitly.

C++11 gave the `auto` keyword a new meaning: automatic type deduction. By using `auto`, the compiler determines the variable's type during compilation, eliminating the need for manual type specification.

The basic syntax for `auto` type deduction is:

```c++
auto name = value;
```

Where `name` is the variable name and `value` is its initial value.

Note: `auto` is merely a placeholder that the compiler replaces with the actual type during compilation. In other words, variables in C++ must have a definite type, but this type can be deduced by the compiler.

**Simple Examples of auto Type Deduction:**

```c++
auto n = 10;
auto f = 12.8;
auto p = &n;
auto url = "https://github.com/wangxwan/Cpp11-14-17-20";
```

Let's break these down:

- In line 1, `10` is an integer literal, defaulting to the `int` type. Therefore, the compiler deduces the type of variable `n` as `int`.

- In line 2, `12.8` is a floating-point literal, defaulting to the `double` type. Hence, the type of variable `f` is deduced as `double`.

- In line 3, `&n` results in a pointer of type `int*`. Consequently, the type of variable `p` is deduced as `int*`.

- In line 4, the string literal enclosed in double quotes (`""`) is of type `const char*`. Therefore, the type of variable `url` is deduced as `const char*`, which is a constant pointer.

We can also define multiple variables consecutively:

```c++
int n = 20;
auto *p = &n, m = 99;
```

Looking at the first sub-expression, the type of `&n` is `int*`. The compiler, encountering `auto *p`, deduces `auto` to be `int`. Naturally, the subsequent variable `m` is also of type `int`, making the assignment of `99` to it valid.

It's crucial to avoid ambiguity during deduction. In this example, once the compiler deduces `auto` as `int` from the first sub-expression, `m` can only be of type `int`. Writing `m = 12.5` would be an error because `12.5` is of type `double`, conflicting with `int`.

Another important point: variables declared with `auto` must be initialized immediately. This is because `auto` in C++11 acts as a "placeholder" and not a true type declaration like `int`.

**Advanced Usage of auto**

Besides standalone use, `auto` can be combined with specific types, representing a "partial" type instead of a complete one. Consider the following code:

```c++
int  x = 0;
auto *p1 = &x;   // p1 is of type int*, auto is deduced as int
auto p2 = &x;    // p2 is of type int*, auto is deduced as int*
auto &r1 = x;    // r1 is of type int&, auto is deduced as int
auto r2 = r1;    // r2 is of type int, auto is deduced as int
```

Let's analyze each line:

- In line 2, `p1` is of type `int*`, meaning `auto *` is `int *`. Therefore, `auto` is deduced as `int`.

- In line 3, `auto` is deduced as `int*`, as demonstrated in previous examples.

- In line 4, `r1` is of type `int&`, and `auto` is deduced as `int`.

- Line 5 requires special attention. While `r1` is of type `int&`, `auto` is deduced as `int`. This demonstrates that when the expression on the right side of `=` is a reference type, `auto` discards the reference and deduces the original type.

Next, let's examine the combination of `auto` and `const`:

```c++
int x = 0;
const auto n = x;    // n is of type const int, auto is deduced as int
auto f = n;          // f is of type const int, auto is deduced as int (const is discarded)
const auto &r1 = x;  // r1 is of type const int&, auto is deduced as int
auto &r2 = r1;       // r2 is of type const int&, auto is deduced as const int
```

Explanation:

- In line 2, `n` is of type `const int`, and `auto` is deduced as `int`.

- In line 3, despite `n` being of type `const int`, `auto` is deduced as `int`. This illustrates that when the expression on the right side of `=` has the `const` qualifier, `auto` ignores it and deduces the non-const type.

- In line 4, `auto` is deduced as `int`, which is straightforward.

- In line 5, `r1` is of type `const int&`, and `auto` is also deduced as `const int`. This shows that when `const` and reference are combined, `auto` preserves the `const` qualifier in its deduction.

In summary, for `auto` combined with `const`:

- When the type is not a reference, `auto` discards the `const` qualifier.

- When the type is a reference, `auto` retains the `const` qualifier.

**Limitations of auto**

We mentioned earlier that variables declared with `auto` must be initialized. This is one limitation. Are there others?

1. **`auto` cannot be used in function parameters.**

This is because function parameters are declared with their types but not assigned values during function definition. Assignment happens during function calls. Since `auto` requires immediate initialization, this creates a conflict.

2. **`auto` cannot be used for non-static class member variables (i.e., member variables without the `static` keyword).**

3. **`auto` cannot define arrays.** For example, the following code is incorrect:

```c++
char url[] = "https://github.com/wangxwan";
auto str[] = url;    // Incorrect: auto cannot define arrays
```

4. **`auto` cannot be used for template parameters.** Consider the following example:

```c++
template <typename T>
class A {
    // ...
};

int main() {
    A<int> C1;
    A<auto> C2 = C1;  // Incorrect: auto cannot be a template parameter
    return 0;
}
```

**Applications of auto**

Having discussed the deduction rules and caveats of `auto`, let's explore its practical applications.

1. **Defining Iterators**

A common use case for `auto` is defining STL iterators.

When working with STL containers, we use iterators to traverse their elements. Different containers have different iterator types, which must be specified during definition. However, iterator types can be complex and cumbersome to write, as shown below:

```c++
#include <vector>
using namespace std;

int main() {
    vector<vector<int>> v;
    vector<vector<int>>::iterator i = v.begin();
    return 0;
}
```

Defining iterator `i` involves a lengthy and error-prone type declaration. With `auto` type deduction, we can simplify this:

```c++
#include <vector>
using namespace std;

int main() {
    vector<vector<int>> v;
    auto i = v.begin();  // Using auto instead of the specific type
    return 0;
}
```

`auto` infers the type of variable `i` based on the type of the expression `v.begin()` (the return type of the `begin()` function).

**2. Generic Programming**

Another application of `auto` is when the variable's type is unknown or we want to avoid specifying it explicitly, such as in generic programming. Let's look at an example:

```c++
#include <iostream>
using namespace std;

class A {
public:
    static int get() {
        return 100;
    }
};

class B {
public:
    static const char* get() {
        return "https://github.com/wangxwan/Cpp11-14-17-20";
    }
};

template <typename T>
void func() {
    auto val = T::get();
    cout << val << endl;
}

int main() {
    func<A>();
    func<B>();
    return 0;
}
```

Output:

```
100
https://github.com/wangxwan/Cpp11-14-17-20
```

The template function `func()` in this example calls the static function `get()` of all classes and processes its return value uniformly. However, the return type of `get()` varies and cannot be automatically converted. Implementing this in earlier C++ versions was cumbersome, requiring an additional template parameter and manually specifying the type of variable `val` during each call.

With `auto` type deduction, the compiler automatically deduces the type of `val` based on the return value of `get()`, eliminating the need for an extra template parameter.

The following code demonstrates the solution without using `auto`:

```c++
#include <iostream>
using namespace std;

class A {
public:
    static int get() {
        return 100;
    }
};

class B {
public:
    static const char* get() {
        return "https://github.com/wangxwan/Cpp11-14-17-20";
    }
};

template <typename T1, typename T2>  // Additional template parameter T2
void func() {
    T2 val = T1::get();
    cout << val << endl;
}

int main() {
    // Manually specifying the template parameter during each call
    func<A, int>();
    func<B, const char*>();
    return 0;
}
```

As you can see, `auto` significantly simplifies code in scenarios like this, making C++ more expressive and easier to use.
