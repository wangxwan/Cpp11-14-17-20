# C++11 - Improved Handling of Consecutive Closing Angle Brackets (>>) in Template Instantiation

In C++98/03 generic programming, template instantiation had a cumbersome aspect: consecutive closing angle brackets (>>) were interpreted as the right shift operator instead of the end of a template argument list.

**Example:** C++98/03 code demonstrating the issue with consecutive closing angle brackets.

```c++
template <typename T>
struct Foo
{
      typedef T type;
};
template <typename T>
class A
{
    // ...
};
int main(void)
{
    Foo<A<int>>::type xx;  // Compilation error
    return 0;
}
```

Compiling this code with GCC would result in the following error:

```shell
error: ‘>>’ should be ‘>>’ within a nested template argument list Foo<A>::type xx;
```

This error indicates that the syntax `Foo<A<int>>` is not supported and should be written as `Foo<A<int> >` (note the space between the two closing angle brackets).

This restriction was unnecessary. Among all paired brackets in C++, only consecutive closing angle brackets exhibited this ambiguity. C++ standard conversion operators like `static_cast` and `reinterpret_cast` use `<>` to obtain the type-id, which could be inconvenient if the type-id itself was a template.

C++11 finally removed this limitation. The standard now requires compilers to handle template closing angle brackets separately, enabling them to correctly distinguish between `>>` as a right shift operator or the delimiter marking the end of a template argument list.

However, this automated handling might introduce incompatibilities with older code in certain cases. Consider the following example:

```c++
template <int N>
struct Foo
{
    // ...
};
int main(void)
{
    Foo<100 >> 2> xx;
    return 0;
}
```

This code would compile without issues in C++98/03 compilers. However, C++11 compilers would report an error:

```shell
error: expected unqualif?ied-id before ‘>’ token Foo<100 >> 2> xx;
```

The solution is to add parentheses:

```c++
Foo<(100 >> 2)> xx; // Note the parentheses
```

Adding parentheses is a good programming practice, promoting unambiguous code.

### Further Notes

Various C++98/03 compilers implemented extensions beyond the standard (ISO/IEC 14882:2003 and earlier). Some of these extensions were later refined and incorporated into C++11.

Therefore, certain C++11 features might be supported by older C++98/03 compilers. However, without standardization, compatibility across different platforms and compilers couldn't be guaranteed. For instance, Microsoft Visual C++ 2005, which doesn't fully support C++11, handles template closing angle brackets consistently with the C++11 standard.


