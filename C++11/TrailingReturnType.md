# C++11 - Trailing Return Types (Deduced Return Types)

In generic programming, you might need to determine the return type of a function based on operations performed on its parameters. Consider the following scenario:

```c++
template <typename R, typename T, typename U>
R add(T t, U u)
{
    return t+u;
}
int a = 1; float b = 2.0;
auto c = add<decltype(a + b)>(a, b);
```

We don't necessarily care about the exact type of `a + b`. We just want to directly obtain the return type using `decltype(a + b)`. However, this approach is inconvenient because the caller doesn't know how the parameters should be combined. Only the `add` function itself knows how to deduce the return type.

So, can we directly use `decltype` within the `add` function definition to get the return type?

```c++
template <typename T, typename U>
decltype(t + u) add(T t, U u)  // error: t and u are not yet defined
{
    return t + u;
}
```

This code won't compile because `t` and `u` are part of the parameter list, and C++ uses a forward declaration syntax for return types. At the point of defining the return type, the parameter variables don't exist yet.

A workaround in C++98/03 is:

```c++
template <typename T, typename U>
decltype((*(T*)0) + (*(U*)0)) add(T t, U u)
{
    return t + u;
}
```

While this successfully deduces the return type using `decltype`, the syntax is obscure, making it difficult to use `decltype` for return type deduction and reducing code readability.

Therefore, C++11 introduced **trailing return types** (also known as **deduced return types**) to address this issue by combining `decltype` and `auto` for return type deduction.

Trailing return types are used in conjunction with `auto` and `decltype`. The `add` function can be rewritten using this new syntax:

```c++
template <typename T, typename U>
auto add(T t, U u) -> decltype(t + u)
{
    return t + u;
}
```

Let's look at another example to further illustrate this syntax:

```c++
int& foo(int& i);
float foo(float& f);
template <typename T>
auto func(T& val) -> decltype(foo(val))
{
    return foo(val);
}
```

While the previous `add` example could be achieved with some effort in C++98/03, this example would be impossible without trailing return types.

In this example, using `decltype` with a trailing return type easily deduces the possible return types of `foo(val)` and applies it to `func`.

Trailing return types solve the problem of determining function return types that depend on parameters. With this syntax, return type deduction can be expressed clearly (directly using parameter operations) without resorting to obscure workarounds required in C++98/03.
