# C++11 - List Initialization (Unified Initialization)

C++98/03 offered various object initialization methods:

```c++
// Initializer list
int i_arr[3] = { 1, 2, 3 };  // Array
struct A {
    int x;
    struct B {
        int i;
        int j;
    } b;
} a = { 1, { 2, 3 } };  // POD type

// Copy initialization
int i = 0;
class Foo {
public:
    Foo(int) {}
} foo = 123;  // Requires copy constructor

// Direct initialization
int j(0);
Foo bar(123);
```

These methods had different scopes and purposes, and no single method worked universally.

C++11 introduces **list initialization** to unify initialization and ensure consistent behavior.

POD stands for "plain old data" and refers to types that can be copied directly using `memcpy`.

## Unified Initialization

C++98/03 allowed initializer lists for arrays and POD types:

```c++
int i_arr[3] = { 1, 2, 3 };
long l_arr[] = { 1, 3, 2, 4 };
struct A {
    int x;
    int y;
} a = { 1, 2 };
```

However, this was limited to these specific types.

C++11 expands initializer list applicability to any type:

```c++
class Foo {
public:
    Foo(int) {}
private:
    Foo(const Foo&);
};

int main(void) {
    Foo a1(123);
    Foo a2 = 123;  // error: 'Foo::Foo(const Foo &)' is private
    Foo a3 = { 123 };
    Foo a4 { 123 };
    int a5 = { 3 };
    int a6 { 3 };
    return 0;
}
```

Here, `a3` and `a4` use list initialization, achieving the same effect as `a1`'s direct initialization.

`a5` and `a6` demonstrate list initialization for fundamental types.

Note: Although `a3` uses an equal sign, it's still list initialization, bypassing the private copy constructor.

C++11 introduces the syntax of `a4` and `a6`, allowing direct initialization with an initializer list after the variable name.

This syntax also applies to arrays and POD types:

```c++
int i_arr[3] { 1, 2, 3 };  // Array
struct A {
    int x;
    struct B {
        int i;
        int j;
    } b;
} a { 1, { 2, 3 } };  // POD type
```

The presence or absence of the equal sign before `{}` doesn't affect the initialization behavior.

As expected, initializer lists can be used with `new` and other contexts where parentheses were used for initialization:

```c++
int* a = new int { 123 };
double b = double { 12.12 };
int* arr = new int[3] { 1, 2, 3 };
```

`a` points to memory initialized with `123`.

`b` copy-initializes from an anonymous object created using list initialization.

Notably, `arr` demonstrates list initialization for dynamically allocated arrays, a welcome addition.

Furthermore, list initialization can be used directly in function return statements:

```c++
struct Foo {
    Foo(int, double) {}
};

Foo func(void) {
    return { 123, 321.0 };
}
```

This `return` statement is equivalent to returning `Foo(123, 321.0)`.

These examples showcase the convenience of list initialization in C++11. It unifies object initialization and enhances code clarity and simplicity.
