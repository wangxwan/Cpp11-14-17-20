# C++11 - decltype Type Deduction

`decltype` is a keyword introduced in C++11 that, like `auto`, performs automatic type deduction at compile time. If you're unfamiliar with `auto`, refer to [auto Type Deduction](autoTypeDeduction.md).

`decltype` is short for "declare type."

Given the existence of `auto`, why introduce `decltype`?  `auto` doesn't cover all type deduction scenarios. In certain situations, using `auto` can be cumbersome or even impossible, hence the addition of `decltype` in C++11.

Both `auto` and `decltype` deduce variable types, but their usage differs:

```c++
auto varname = value;
decltype(exp) varname = value;
```

Here, `varname` represents the variable name, `value` is the assigned value, and `exp` is an expression.

`auto` deduces the variable's type based on the initializer `value` on the right side of `=`, while `decltype` deduces it based on the `exp` expression, independent of the `value`.

Furthermore, `auto` requires variable initialization, while `decltype` doesn't. This makes sense, as `auto` relies on the initial value for type deduction. `decltype` can be written as:

```c++
decltype(exp) varname;
```

## Considerations for `exp`

In principle, `exp` is a regular expression and can be arbitrarily complex. However, it's crucial to ensure that `exp` results in a typed value and not `void`. For instance, if `exp` calls a function returning `void`, `exp` itself will also be of type `void`, leading to a compilation error.

**C++ decltype Examples:**

```c++
int a = 0;
decltype(a) b = 1;  // b is deduced as int
decltype(10.8) x = 5.5;  // x is deduced as double
decltype(x + 100) y;  // y is deduced as double
```

As shown, `decltype` can deduce types from variables, literals, and expressions with operators. Note that in line 4, `y` is not initialized.

## decltype Deduction Rules

The examples above provide a basic understanding of `decltype`. However, its usage can be more intricate. When using `decltype(exp)` to obtain a type, the compiler follows these rules:

1. **Unparenthesized Expressions, Member Access, Single Variables:** If `exp` is an expression not enclosed in parentheses `( )`, a class member access expression, or a single variable, then `decltype(exp)` has the same type as `exp`. This is the most common case.

2. **Function Calls:** If `exp` is a function call, then `decltype(exp)` has the same type as the function's return type.

3. **Lvalues and Parenthesized Expressions:** If `exp` is an lvalue or enclosed in parentheses `( )`, then `decltype(exp)` is a reference to the type of `exp`. If `exp` has type `T`, then `decltype(exp)` has type `T&`.

Let's delve into practical examples to solidify our understanding of these rules.

**Example 1: `exp` is a Regular Expression**

```c++
#include <string>
using namespace std;

class Student {
public:
    static int total;
    string name;
    int age;
    float scores;
};

int Student::total = 0;

int main() {
    int n = 0;
    const int &r = n;
    Student stu;

    decltype(n) a = n;        // n is int, a is deduced as int
    decltype(r) b = n;        // r is const int&, b is deduced as const int&
    decltype(Student::total) c = 0; // total is int, c is deduced as int
    decltype(stu.name) url = "https://github.com/wangxwan/Cpp11-14-17-20"; // name is string, url is deduced as string

    return 0;
}
```

This code demonstrates rule 1. For general expressions, `decltype` deduces the same type as the expression.

**Example 2: `exp` is a Function Call**

```c++
// Function declarations
int& func_int_r(int, char);  // Returns int&
int&& func_int_rr();         // Returns int&&
int func_int(double);       // Returns int
const int& fun_cint_r(int, int, int);  // Returns const int&
const int&& func_cint_rr();           // Returns const int&&

// decltype type deduction
int n = 100;
decltype(func_int_r(100, 'A')) a = n;  // a is of type int&
decltype(func_int_rr()) b = 0;          // b is of type int&&
decltype(func_int(10.5)) c = 0;         // c is of type int
decltype(fun_cint_r(1, 2, 3)) x = n;    // x is of type const int&
decltype(func_cint_rr()) y = 0;          // y is of type const int&&
```

Note that while function calls within `exp` require parentheses and arguments, this is purely for form and doesn't execute the function code.

**Example 3: `exp` is an Lvalue or Parenthesized**

```c++
using namespace std;

class Base {
public:
    int x;
};

int main() {
    const Base obj;

    // Parenthesized expression
    decltype(obj.x) a = 0;  // obj.x is a member access expression (rule 1), a is int
    decltype((obj.x)) b = a; // obj.x is parenthesized (rule 3), b is int&

    // Addition expression
    int n = 0, m = 0;
    decltype(n + m) c = 0;  // n + m yields an rvalue (rule 1), c is int
    decltype(n = n + m) d = c; // n = n + m yields an lvalue (rule 3), d is int&

    return 0;
}
```

It's important to distinguish between lvalues and rvalues. Lvalues refer to persistent data that exists after the expression is evaluated, while rvalues are temporary data that doesn't persist. A simple way to differentiate them is by taking their address: if the compiler doesn't complain, it's an lvalue; otherwise, it's an rvalue.

## Practical Applications of decltype

While `auto` has a simpler syntax and is generally more convenient for type deduction, this section focuses on scenarios where only `decltype` suffices.

One such scenario is deducing the type of non-static class members, which `auto` cannot handle. Consider the following template definition:

```c++
#include <vector>
using namespace std;

template <typename T>
class Base {
public:
    void func(T& container) {
        m_it = container.begin();
    }
private:
    typename T::iterator m_it;  // Note this line
};

int main() {
    const vector<int> v;
    Base<const vector<int>> obj;
    obj.func(v);
    return 0;
}
```

The definition of member `m_it` in class `Base` might seem fine at first glance. However, when using `Base` with a `const` container, the compiler throws errors. This is because `T::iterator` doesn't encompass all iterator types. When `T` is a `const` container, `const_iterator` should be used.

In C++98/03, addressing this required template specialization for `const` containers, adding complexity and verbosity. However, C++11's `decltype` provides an elegant solution:

```c++
template <typename T>
class Base {
public:
    void func(T& container) {
        m_it = container.begin();
    }
private:
    decltype(T().begin()) m_it;  // Note this line
};
```

Much cleaner, isn't it?

Note that some older compilers might not support the `T().begin()` syntax. The code above was tested successfully in VS2019 but failed in VS2015.


