# C++11 - `constexpr` vs. `const`: Distinguishing the Differences

In the [constexpr: Verifying Constant Expressions](constexpr.md) section, we explored the functionality and usage of the `constexpr` keyword. However, some learners often confuse `const` and `constexpr`, unsure when to use which. This section aims to clarify the distinctions between these two keywords.

As we know, `constexpr` is a new keyword introduced in C++11. Before that (C++98/03), only `const` existed, and it often exhibited two different meanings. Consider this example:

```c++
#include <iostream>
#include <array>
using namespace std;

void dis_1(const int x) {
    // Error: 'x' is a read-only variable
    array<int, x> myarr {1, 2, 3, 4, 5};
    cout << myarr[1] << endl;
}

void dis_2() {
    const int x = 5;
    array<int, x> myarr {1, 2, 3, 4, 5};
    cout << myarr[1] << endl;
}

int main() {
    dis_1(5);
    dis_2();
}
```

Both `dis_1()` and `dis_2()` contain `const int x`, but only `dis_2()` successfully initializes the `array` container.

This is because in `dis_1()`, `const int x` merely emphasizes that `x` is a read-only variable. It's still a variable at its core and cannot be used to initialize an `array` container, which requires a constant size.

In `dis_2()`, `const int x` implies both read-only and constant nature. `x` is a constant with a value of 5, making it suitable for initializing the `array`.

C++11 addresses this dual meaning of `const` by retaining its "read-only" semantics and assigning the "constant" semantics to the new `constexpr` keyword. Therefore, it's recommended to use `const` for "read-only" and `constexpr` for "constant" scenarios.

> In the example above, using `const int x` in `dis_2()` is not ideal; `constexpr` would be more appropriate.

You might wonder, doesn't "read-only" imply "cannot be modified"? Not necessarily. "Read-only" and "immutable" are not synonymous. Consider this example:

```c++
#include <iostream>
using namespace std;

int main() {
    int a = 10;
    const int& con_b = a;
    cout << con_b << endl;
    a = 20;
    cout << con_b << endl;
}
```

Output:

```
10
20
```

Here, `con_b` is declared `const`, making it read-only. You cannot directly modify its value through `con_b` itself. However, its value can change indirectly through modifications to `a`.

In most cases, `const` and `constexpr` can be used interchangeably:

```c++
const int a = 5 + 4;
constexpr int a = 5 + 4;
```

Both are equivalent and evaluated at compile time. However, certain scenarios require explicit use of `constexpr`:

```c++
#include <iostream>
#include <array>
using namespace std;

constexpr int sqr1(int arg) {
    return arg * arg;
}

const int sqr2(int arg) {
    return arg * arg;
}

int main() {
    array<int, sqr1(10)> mylist1; // Works: sqr1 is a constexpr function
    array<int, sqr2(10)> mylist2; // Error: sqr2 is not a constexpr function
    return 0;
}
```

`sqr2()`'s return value is only `const`, not `constexpr`, making it unsuitable for initializing the `array` container, which requires a constant size.

In summary, in C++11:

- `const`: Indicates that the variable is read-only.
- `constexpr`: Specifies a constant or constant expression, allowing compile-time evaluation for improved performance.


