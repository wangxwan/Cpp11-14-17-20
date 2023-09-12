# C++11 - Rvalue References

As mentioned in the [README](../README.md), C++11 introduces approximately 140 new features to the language, building upon the C++98/03 standard. Rvalue references are one of these additions and a crucial one at that.

Many beginners find rvalue references confusing, but they are simply a new C++ syntax. The real complexity lies in two programming techniques they enable: move semantics and perfect forwarding. This section focuses on understanding rvalue references and their basic usage, leaving move semantics and perfect forwarding for later discussions.

## Lvalues and Rvalues in C++

The term "rvalue reference" is self-explanatory: it refers to using C++ rvalues by reference (instead of by value).

In C++, an expression (literal, variable, object, function return value, etc.) can be categorized as either an lvalue expression or an rvalue expression based on its usage context. This concept of lvalues and rvalues is inherited from C.

> Note: "lvalue" stands for "locator value," indicating data stored in memory with a specific address (addressable). "rvalue" originally meant "register value" but now broadly refers to data that can provide a value (not necessarily addressable, like data in registers).

Here are two common ways to distinguish lvalues from rvalues:

1. **Assignment:** An expression that can appear on the left side of an assignment operator (`=`) is an lvalue. Conversely, an expression that can only appear on the right side is an rvalue. Example:

   ```c++
   int a = 5;
   5 = a; // Error: 5 cannot be an lvalue
   ```

   Here, `a` is an lvalue, while the literal `5` is an rvalue. Note that lvalues can be used as rvalues:

   ```c++
   int b = 10; // b is an lvalue
   a = b; // Both a and b are lvalues, but b is used as an rvalue here
   ```

2. **Name and Address:** An expression with a name and a retrievable memory address is an lvalue. Otherwise, it's an rvalue.

   In the previous example, `a` and `b` are variable names, and we can obtain their addresses using `&a` and `&b`. Therefore, they are lvalues. Literals like `5` and `10` have no names and no retrievable addresses (they are typically stored in registers or with the code), making them rvalues.

> Note: These methods are not exhaustive. This section focuses on rvalue references, so we won't delve deeper into lvalues and rvalues.

## Rvalue References in C++

C++98/03 already had references, denoted by `&`. However, these references, now called **lvalue references**, could only operate on lvalues under normal circumstances. Example:

```c++
int num = 10;
int& b = num; // Correct
int& c = 10; // Error
```

We can create a reference to the lvalue `num`, but not to the rvalue `10`.

Note: While C++98/03 didn't support non-const lvalue references to rvalues, it allowed **const lvalue references** to bind to rvalues. Const lvalue references can operate on both lvalues and rvalues:

```c++
int num = 10;
const int& b = num; // OK
const int& c = 10; // OK
```

Rvalues are often temporary and unnamed, so we can only use them through references. However, we might need to modify rvalues (e.g., in move semantics), which const lvalue references don't allow.

C++11 introduces **rvalue references**, denoted by `&&`, to address this.

> The choice of `&&` for rvalue references was driven by the need to use existing C++ symbols while avoiding conflicts with C++98/03.

Like lvalue references, rvalue references must be initialized immediately, and they can only be initialized with rvalues:

```c++
int num = 10;
int&& a = num; // Error: Cannot initialize rvalue reference with an lvalue
int&& b = 10; // Correct
```

Unlike const lvalue references, rvalue references allow modification of the rvalue:

```c++
int&& a = 10;
a = 100;
cout << a << endl; // Output: 100
```

C++ allows declaring **const rvalue references**:

```c++
const int&& a = 10;
```

However, these have limited practical use. Rvalue references are primarily for move semantics and perfect forwarding, which require modification. Const rvalue references, which refer to unmodifiable rvalues, are essentially replaceable by const lvalue references.

Here's a table summarizing lvalue and rvalue reference behavior:

| Reference Type | Non-Const Lvalue | Const Lvalue | Non-Const Rvalue | Const Rvalue | Usage |
|---|---|---|---|---|---|
| Non-Const Lvalue Reference | Y | N | N | N | None |
| Const Lvalue Reference | Y | Y | Y | Y | Common in copy constructors |
| Non-Const Rvalue Reference | N | N | Y | N | Move semantics, perfect forwarding |
| Const Rvalue Reference | N | N | Y | Y | No practical use |

(Y: Supported, N: Not Supported)

> C++11 further categorizes rvalues into **prvalues** (pure rvalues) and **xvalues** (expiring values). Prvalues are the traditional rvalues discussed in this section. Xvalues are expressions related to rvalue references (e.g., a function returning `T&&`). Both prvalues and xvalues are considered rvalues.


