# C++11 - Default Template Arguments for Function Templates

In C++98/03, class templates could have default template arguments:

```c++
template <typename T, typename U = int, U N = 0>
struct Foo
{
    // ...
};
```

However, function templates did not support default template arguments:

```c++
template <typename T = int>  // error in C++98/03: default template arguments
void func()
{
    // ...
}
```

C++11 lifts this restriction. The `func` function above is now valid in C++11:

```c++
int main(void)
{
    func();   //T = int
    return 0;
}
```

In this case, the template argument `T` defaults to `int`. When all template parameters have defaults, calling a function template resembles calling an ordinary function. However, for class templates, even with default arguments, explicit instantiation with `<>` after the template name is still required.

Besides this change, default template arguments for function templates have some differences in usage compared to other default arguments. They are not restricted to appearing last in the parameter list. Moreover, the compiler can deduce some template argument types based on the context of the function template call.

This combination of default template arguments and automatic type deduction provides significant flexibility. We can specify default arguments for some template parameters while letting the compiler deduce others, as shown below:

```c++
template <typename R = int, typename U>
R func(U val)
{
    return val;
}
int main()
{
    func(97);               // R=int, U=int
    func<char>(97);         // R=char, U=int
    func<double, int>(97);  // R=double, U=int
    return 0;
}
```

In C++11, we can call the template function as `func(97)`. The compiler deduces `U` as `int` from the argument `97` and `R` as `int` from the return value `val`. In `func<char>(97)`, we explicitly specify `R` as `char` (overriding the default), and the compiler deduces `U` as `int`. Finally, in `func<double, int>(97)`, we explicitly provide both `R` and `U`, so no deduction is needed.

It's crucial to remember that when using both default and deduced template arguments, the compiler prioritizes deduction. If a type cannot be deduced, the default argument is used. If neither deduction nor a default argument is available, the compiler reports an error. For example:

```c++
template <typename T, typename U = double>
void func(T val1 = 0, U val2 = 0)
{
    //...
}
int main()
{
    func('c'); //T=char, U=double
    func();    //Compilation error
    return 0;
}
```

In `func('c')`, the compiler deduces `T` as `char` from the argument `'c'`. Since the second argument is missing, `U` defaults to `double`. However, `func()` fails to compile because even though `val1` has a default value, the compiler cannot deduce `T` from it. This demonstrates that the compiler's deduction capabilities are not limitless.

In conclusion, C++11 allows default arguments for function template parameters. We can choose to use default values, rely on compiler deduction, or explicitly specify all template argument types, providing flexibility and convenience in generic programming.
