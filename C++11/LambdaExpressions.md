# C++11 - Lambda Expressions

Lambda, derived from the 11th letter of the Greek alphabet (λ), represents anonymous functions in computer science. These functions, often called lambda functions or lambda expressions, are essentially functions without names.

Following the footsteps of languages like Python, Java, C#, and PHP, C++11 finally introduces lambda expressions. This section provides a comprehensive guide to their usage.

## Defining Lambda Functions

Defining a lambda function is straightforward, following this syntax:

```c++
[capture-list] (parameters) mutable noexcept/throw() -> return-type {
    function-body;
};
```

Let's break down each part:

1. **[capture-list]**:
   - Square brackets `[]` signal a lambda expression and are mandatory.
   - Inside, you specify how external variables are captured (accessed) within the lambda's scope.

   > External variables are local variables within the same scope as the lambda.

2. **(parameters)**:
   - Similar to regular functions, lambdas can accept parameters.
   - If no parameters are needed, both parentheses `()` can be omitted.

3. **mutable**:
   - Optional keyword. If used, `()` is required (even with no parameters).
   - By default, external variables captured by value are treated as `const` inside the lambda.
   - `mutable` allows modification of captured copies (not the original variables).

4. **noexcept/throw()**:
   - Optional. If used, `()` is required (even with no parameters).
   - By default, lambdas can throw any exception.
   - `noexcept` indicates the lambda won't throw any exceptions.
   - `throw(type-list)` specifies the types of exceptions the lambda can throw.

   > Uncaught exceptions (violating `noexcept` or `throw()`) lead to program termination.

5. **-> return-type**:
   - Specifies the lambda's return type.
   - Can be omitted if the lambda has a single `return` statement or returns `void` (the compiler deduces the type).

6. **{ function-body; }**:
   - Contains the lambda's code.
   - Can access captured external variables, passed parameters, and global variables.

   > Captured variables are affected by pass-by-value/reference, while global variables are always accessible and modifiable.

   > **Red** indicates required parts, while **green** indicates optional parts.

Here's the simplest lambda function:

```c++
[]{}
```

It captures no external variables, takes no parameters, has no special specifiers, and has an empty body – essentially a no-op lambda.

## [Capture List] in Lambda Functions

The capture list often confuses beginners. Here's a breakdown of common capture patterns:

| Capture List | Description |
|---|---|
| `[]` | Captures no external variables. |
| `[=]` | Captures all external variables by value. |
| `[&]` | Captures all external variables by reference. |
| `[val1, val2, ...]` | Captures specific variables by value (order doesn't matter). |
| `[&val1, &val2, ...]` | Captures specific variables by reference (order doesn't matter). |
| `[val, &val2, ...]` | Mixes capture by value and reference (order doesn't matter). |
| `[=, &val1, ...]` | Captures all variables by value except `val1`, which is captured by reference. |
| `[this]` | Captures the current object's `this` pointer by value. |

> Note: Capturing the same variable multiple times with the same method is illegal (e.g., `[=, val1]`).

**Example 1: Defining and Using a Lambda**

```c++
#include <iostream>
#include <algorithm>
using namespace std;

int main() {
    int num[4] = {4, 2, 3, 1};

    // Sort elements in ascending order using a lambda
    sort(num, num + 4, [=](int x, int y) -> bool { return x < y; });

    for (int n : num) {
        cout << n << " ";
    }
    return 0;
}
```

Output:

```
1 2 3 4
```

This code sorts the `num` array using a lambda function for the comparison. Achieving the same with a regular function would be more verbose:

```c++
// ... (previous code)

// Custom comparison function for ascending order
bool sort_up(int x, int y) {
    return x < y;
}

int main() {
    // ... (previous code)

    // Sort using the custom comparison function
    sort(num, num + 4, sort_up);

    // ... (rest of the code)
}
```

Lambdas offer a concise way to define functionality directly where needed.

We can also assign a lambda to a variable for later use:

```c++
#include <iostream>
using namespace std;

int main() {
    // 'display' now refers to the lambda function
    auto display = [](int a, int b) -> void { cout << a << " " << b; };

    // Call the lambda function
    display(10, 20);
    return 0;
}
```

Output:

```
10 20
```

Here, `auto` deduces the type, allowing us to call the lambda through the `display` variable.

**Example 2: Capture by Value vs. Reference**

```c++
#include <iostream>
using namespace std;

// Global variable
int all_num = 0;

int main() {
    // Local variables
    int num_1 = 1;
    int num_2 = 2;
    int num_3 = 3;

    cout << "lambda1:\n";
    auto lambda1 = [=]() {
        // Access and modify global variable
        all_num = 10;

        // Can use but not modify captured local variables
        cout << num_1 << " " << num_2 << " " << num_3 << endl;
    };
    lambda1();
    cout << all_num << endl;

    cout << "lambda2:\n";
    auto lambda2 = [&]() {
        // Access and modify both global and captured local variables
        all_num = 100;
        num_1 = 10;
        num_2 = 20;
        num_3 = 30;

        cout << num_1 << " " << num_2 << " " << num_3 << endl;
    };
    lambda2();
    cout << all_num << endl;

    return 0;
}
```

Output:

```
lambda1:
1 2 3
10
lambda2:
10 20 30
100
```

`lambda1` captures local variables by value (`[=]`), so it can use but not modify them. It can, however, modify the global variable `all_num`.

`lambda2` captures by reference (`[&]`), allowing both access and modification of captured local variables and the global variable.

> Try modifying `num_1`, `num_2`, or `num_3` inside `lambda1` to see the compiler error.

To modify captured copies within a lambda using `[=]`, use the `mutable` keyword:

```c++
auto lambda1 = [=]() mutable {
    num_1 = 10;
    num_2 = 20;
    num_3 = 30;
    // ...
};
```

This modifies the captured copies, not the original variables outside the lambda.

**Example 3: Exception Handling**

```c++
#include <iostream>
using namespace std;

int main() {
    auto except = []() throw(int) {
        throw 10;
    };

    try {
        except();
    } catch (int) {
        cout << "Caught integer exception" << endl;
    }

    return 0;
}
```

Output:

```
Caught integer exception
```

Here, `except` specifies that it can throw an `int` exception, which is caught by the `try-catch` block.

Now, let's see what happens when the exception specification is violated:

```c++
// ... (previous code)

int main() {
    auto except1 = []() noexcept {
        throw 100; // Violates noexcept
    };

    auto except2 = []() throw(char) {
        throw 10; // Throws int, not char
    };

    try {
        except1();
        except2();
    } catch (int) {
        cout << "Caught integer exception" << endl;
    }

    return 0;
}
```

This code will crash because `except1` throws an exception despite being marked `noexcept`, and `except2` throws the wrong exception type. The `try-catch` block cannot handle these violations, leading to program termination.

> Omitting `noexcept` or `throw()` allows the lambda to throw any exception type.


