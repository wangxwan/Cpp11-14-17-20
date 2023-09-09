# C++11 - `constexpr`: Verifying Constant Expressions

Before diving into `constexpr`, a new keyword in C++11, let's understand what constant expressions are in C++.

A **constant expression** is an expression composed of one or more constants. In other words, if all components of an expression are constants, the expression itself is a constant expression. This implies that once a constant expression is evaluated, its value cannot be changed.

Constant expressions are frequently used in development. For instance, array sizes must be constant expressions:

```c++
// 1)
int url[10]; // Correct
// 2)
int url[6 + 4]; // Correct
// 3)
int length = 6;
int url[length]; // Error: 'length' is a variable
```

The first two examples use constant expressions (10 and 6+4) for the array size. The third example fails because `length` is a variable, not a constant.

> Constant expressions have other uses, such as in anonymous enums and `case` labels in `switch` statements.

C++ programs go through compilation, linking, and execution. Constant expressions are evaluated during compilation, while non-constant expressions are evaluated at runtime. This compile-time evaluation significantly improves performance, as the expression is calculated only once during compilation, saving time during each program execution.

Performance is a constant pursuit in C++ programming. So, how do we determine if an expression is a constant expression to leverage this compile-time evaluation? C++11 introduces the `constexpr` keyword for this purpose.

`constexpr` enables the specified constant expression to be evaluated at compile time. It can be applied to variables, functions (including template functions), and class constructors.

> Note: While `constexpr` allows compile-time evaluation, it doesn't guarantee it. The compiler ultimately decides the evaluation timing.

## `constexpr` with Variables

In C++11, you can use `constexpr` to declare variables, enabling their compile-time evaluation.

Importantly, `constexpr` variables must be initialized with a constant expression:

```c++
#include <iostream>
using namespace std;

int main() {
    constexpr int num = 1 + 2 + 3;
    int url[num] = {1, 2, 3, 4, 5, 6};
    cout << url[1] << endl;
    return 0;
}
```

Output:

```
2
```

> Removing `constexpr` would result in a compilation error: "expression must have a constant value."

Here, `constexpr` allows the compiler to evaluate `num` (1+2+3) at compile time, making it valid for use as an array size.

You might notice that replacing `constexpr` with `const` in this example still works. This is because `num` meets both conditions: it's declared `const` and initialized with a constant expression.

> Note: `const` and `constexpr` are not the same. We'll discuss their differences in the next section.

When constant expressions involve floating-point numbers, their precision might differ between compile time and runtime due to potential variations in system environments. C++11 mandates that the compile-time precision of floating-point constant expressions must be at least as high as the runtime precision.

## `constexpr` with Functions

`constexpr` can also be applied to function return values, creating **constexpr functions**.

However, not all functions can be constexpr functions. A function must meet these four conditions:

1. **Single Return Statement:** The function body can only contain a single `return` statement, along with `using` directives, `typedef` declarations, and `static_assert` assertions.

   Example:

   ```c++
   constexpr int display(int x) {
       // Can include using directives, typedef, and static_assert
       return 1 + 2 + x;
   }
   ```

   The following would be incorrect:

   ```c++
   constexpr int display(int x) {
       int ret = 1 + 2 + x; // Multiple statements
       return ret;
   }
   ```

2. **Non-Void Return Type:** The function must return a value; `void` return type is not allowed.

   ```c++
   constexpr void display() { // Incorrect: void return type
       // Function body
   }
   ```

3. **Definition Before Use:** Constexpr functions must be defined before they are used. Unlike regular functions, which can be declared and defined later, constexpr functions require their definition to be available at the point of use.

   Example:

   ```c++
   #include <iostream>
   using namespace std;

   // Regular function declaration
   int noconst_dis(int x);

   // Constexpr function declaration and definition
   constexpr int display(int x) {
       return 1 + 2 + x;
   }

   int main() {
       // Call constexpr function
       int a[display(3)] = {1, 2, 3, 4};
       cout << a[2] << endl;

       // Call regular function
       cout << noconst_dis(3) << endl;

       return 0;
   }

   // Regular function definition
   int noconst_dis(int x) {
       return 1 + 2 + x;
   }
   ```

   Output:

   ```
   3
   6
   ```

   > Moving `display()`'s definition after `main()` would result in a compilation error.

4. **Constant Expression Return Value:** The expression returned by `return` must be a constant expression.

   Example:

   ```c++
   #include <iostream>
   using namespace std;

   int num = 3; // Non-constant variable

   constexpr int display(int x) {
       return num + x; // Error: 'num' is not a constant expression
   }

   int main() {
       // Call constexpr function
       int a[display(3)] = {1, 2, 3, 4};
       return 0;
   }
   ```

   This code fails to compile because `num` is not a constant expression.

   The rationale is that to obtain a constant value at compile time, the `return` statement cannot involve variables whose values are determined at runtime.

   > Note: Assignment operations within the `return` statement are not allowed in constexpr functions (e.g., `return x = 1`). Constexpr functions can be recursive.

## `constexpr` with Class Constructors

You cannot directly apply `constexpr` to user-defined data types (structs or classes).

Example:

```c++
#include <iostream>
using namespace std;

// Custom type definition
constexpr struct myType { // Error: Cannot apply constexpr to a struct
    const char* name;
    int age;
    // Other members
};

int main() {
    constexpr struct myType mt {"zhangsan", 10}; // Error
    cout << mt.name << " " << mt.age << endl;
    return 0;
}
```

To create a type that can produce constants, define a **constexpr constructor** within the type:

```c++
#include <iostream>
using namespace std;

// Custom type definition
struct myType {
    constexpr myType(const char* name, int age) : name(name), age(age) {} // Constexpr constructor
    const char* name;
    int age;
    // Other members
};

int main() {
    constexpr struct myType mt {"wangxwan", 10};
    cout << mt.name << " " << mt.age << endl;
    return 0;
}
```

Output:

```
wangxwan 10
```

The `constexpr` constructor allows the creation of `constexpr` objects of type `myType`.

Note: Constexpr constructors must have an empty function body and use constant expressions in the initializer list.

Since member functions are essentially functions within the class's namespace, `constexpr` can also be applied to them, as long as they meet the four conditions mentioned earlier.

Example:

```c++
#include <iostream>
using namespace std;

// Custom type definition
class myType {
public:
    constexpr myType(const char* name, int age) : name(name), age(age) {}
    constexpr const char* getname() {
        return name;
    }
    constexpr int getage() {
        return age;
    }
private:
    const char* name;
    int age;
    // Other members
};

int main() {
    constexpr struct myType mt {"wangxwan", 10};
    constexpr const char* name = mt.getname();
    constexpr int age = mt.getage();
    cout << name << " " << age << endl;
    return 0;
}
```

Output:

```
wangxwan 10
```

> Note: C++11 doesn't allow `constexpr` for virtual member functions.

## `constexpr` with Template Functions

C++11 allows `constexpr` for template functions. However, due to the template's type uncertainty, the instantiated function might not always meet the constexpr function requirements.

In such cases, C++11 specifies that `constexpr` is ignored, and the function behaves like a regular function.

Example:

```c++
#include <iostream>
using namespace std;

// Custom type definition
struct myType {
    const char* name;
    int age;
    // Other members
};

// Template function
template <typename T>
constexpr T display(T t) {
    return t;
}

int main() {
    struct myType stu {"wangxwan", 10};

    // Regular function call
    struct myType ret = display(stu); // constexpr ignored
    cout << ret.name << " " << ret.age << endl;

    // Constexpr function call
    constexpr int ret1 = display(10);
    cout << ret1 << endl;

    return 0;
}
```

Output:

```
wangxwan 10
10
```

The `display()` template function's constexpr status depends on the instantiated type:

- When instantiated with `myType` (line 20), it's not a constexpr function because `myType` lacks a constexpr constructor. `constexpr` is ignored.

- When instantiated with `int` (line 23), it meets the constexpr function requirements, resulting in a constant expression return value.


