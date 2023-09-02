## C++11 - Summary of Differences Between auto and decltype

Having learned about [auto](autoTypeDeduction.md) and [decltype](decltypeTypeDeduction.md), you should be familiar with their syntax and usage scenarios. This section compares and contrasts auto and decltype, highlighting their differences and guiding you on when to choose which.

## Syntax Differences

Both auto and decltype are keywords introduced in C++11 for automatic type deduction, but their syntax differs:

```c++
auto varname = value; // auto syntax
decltype(exp) varname [= value]; // decltype syntax
```

Here, `varname` represents the variable name, `value` represents the value assigned to the variable, `exp` represents an expression, and square brackets `[ ]` indicate optionality.

Both auto and decltype automatically deduce the type of variable `varname`:

- `auto` deduces the type based on the initial value `value` on the right side of `=`.

- `decltype` deduces the type based on the expression `exp`, independent of the value on the right side of `=`.

Furthermore, `auto` requires variable initialization, meaning you must assign a value when defining the variable. `decltype` doesn't have this requirement; initialization doesn't affect the deduced type. This is because `auto` relies on the initial value to deduce the type, making deduction impossible without initialization.

`auto` binds the variable type and initial value, while `decltype` separates them. Although `auto` offers more concise syntax, `decltype` provides greater flexibility.

Consider the following examples:

```c++
auto n1 = 10;
decltype(10) n2 = 99;
auto url1 = "https://github.com/wangxwan/Cpp11-14-17-20/blob/master/C++11/autoTypeDeduction.md";
decltype(url1) url2 = "https://github.com/wangxwan/Cpp11-14-17-20/blob/master/C++11/decltypeTypeDeduction.md";
auto f1 = 2.5;
decltype(n1*6.7) f2;
```

These usages were analyzed in previous sections and won't be elaborated on here.

## Handling of cv-Qualifiers

"cv-qualifiers" refer to the `const` and `volatile` keywords:

- `const` indicates read-only data, meaning it cannot be modified.

- `volatile`, contrary to `const`, indicates mutable and volatile data, preventing the CPU from caching the data in registers and forcing it to be read from the original memory location.

`auto` and `decltype` handle cv-qualifiers differently during type deduction. `decltype` preserves cv-qualifiers, while `auto` may discard them.

Here are the rules for `auto`'s deduction of cv-qualifiers:

- If the expression's type is not a pointer or reference, `auto` discards cv-qualifiers, deducing a non-const or non-volatile type.

- If the expression's type is a pointer or reference, `auto` preserves cv-qualifiers.

The following example demonstrates deduction with the `const` qualifier:

```c++
// Non-pointer, non-reference type
const int n1 = 0;
auto n2 = 10;
n2 = 99;  // Assignment successful
decltype(n1) n3 = 20;
n3 = 5;  // Assignment error
// Pointer type
const int *p1 = &n1;
auto p2 = p1;
*p2 = 66;  // Assignment error
decltype(p1) p3 = p1;
*p3 = 19;  // Assignment error
```

In C++, directly printing the complete type of a variable is not possible. We can infer whether a variable is const-qualified by attempting assignment. If the assignment fails, it implies the variable is const-qualified; otherwise, it's not. While not the most intuitive approach, it serves the purpose.

`n2`'s successful assignment indicates it's not const-qualified, meaning `auto` discarded the `const` qualifier, verifying the first deduction rule. `p2`'s assignment failure indicates it's const-qualified, meaning `auto` preserved the `const` qualifier, verifying the second deduction rule.

Both `n3` and `p3`'s assignment failures indicate that `decltype` preserves the `const` qualifier of the expression.

## Handling of References

When the expression's type is a reference, `auto` and `decltype` also differ in their deduction rules. `decltype` preserves the reference type, while `auto` discards it, deducing the original type directly. Consider the following example:

```c++
#include <iostream>
using namespace std;
int main() {
    int n = 10;
    int &r1 = n;
    // auto deduction
    auto r2 = r1;
    r2 = 20;
    cout << n << ", " << r1 << ", " << r2 << endl;
    // decltype deduction
    decltype(r1) r3 = n;
    r3 = 99;
    cout << n << ", " << r1 << ", " << r3 << endl;
    return 0;
}
```

Output:

```shell
10, 10, 20
99, 99, 99
```

The output shows that assigning to `r2` doesn't change the value of `n`, indicating that `r2` doesn't point to `n` but has its own separate memory location. This proves that `r2` is no longer a reference type; its reference type was discarded by `auto`.

Assigning to `r3` also changes the value of `n`, indicating that `r3` still points to `n`, and its reference type was preserved by `decltype`.

## Conclusion

While `auto` offers simpler syntax than `decltype`, its deduction rules are more complex and may sometimes alter the original type of the expression. `decltype` is more straightforward, generally preserving the original expression's type, making the deduction more faithful.

From a code robustness perspective, `decltype` is recommended as it avoids potential pitfalls. However, `decltype` can be cumbersome, especially with complex expressions, for example:

```c++
vector<int> nums;
decltype(nums.begin()) it = nums.begin();
```

Using `auto` in such cases is much cleaner:

```c++
vector<int> nums;
auto it = nums.begin();
```

In practice, developers often prefer `auto` (myself included) due to its simplicity and intuitiveness, aligning better with aesthetic preferences. If your expression type is not complex, `auto` is recommended for its elegance and readability.
