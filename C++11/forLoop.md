# C++11 - Range-Based for Loops

Before C++11 (C++98/03), iterating over arrays or containers with a `for` loop required the following structure:

```c++
for (expression1; expression2; expression3) {
    // Loop body
}
```

Here's an example demonstrating this structure for iterating over an array and a container (Example 1):

```c++
#include <iostream>
#include <vector>
#include <string.h>
using namespace std;

int main() {
    char arc[] = "https://github.com/wangxwan/Cpp11-14-17-20";
    int i;

    // Iterate over an array
    for (i = 0; i < strlen(arc); i++) {
        cout << arc[i];
    }
    cout << endl;

    vector<char> myvector(arc, arc + 28);
    vector<char>::iterator iter;

    // Iterate over a vector container
    for (iter = myvector.begin(); iter != myvector.end(); ++iter) {
        cout << *iter;
    }

    return 0;
}
```

Output:

```
https://github.com/wangxwan/Cpp11-14-17-20
https://github.com/wangxwan/
```

> In this example, `vector` is a sequential container from the STL.

C++11 introduces a new syntax for `for` loops, offering a more concise way to iterate over ranges:

```c++
for (declaration : expression) {
    // Loop body
}
```

- **declaration**: Defines a variable whose type matches the element type of the sequence being iterated. C++11 allows using `auto` here for automatic type deduction.

- **expression**: Represents the sequence to iterate over, such as an array, container, or an initializer list enclosed in `{}`.

The key difference from the traditional `for` loop is the absence of explicit loop bounds. The new range-based `for` loop automatically iterates over each element in the specified sequence.

Here's how to iterate over the `arc` array and `myvector` container from Example 1 using the range-based `for` loop:

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    char arc[] = "https://github.com/wangxwan/Cpp11-14-17-20";

    // Iterate over an array
    for (char ch : arc) {
        cout << ch;
    }
    cout << '!' << endl;

    vector<char> myvector(arc, arc + 28);

    // Iterate over a vector container
    for (auto ch : myvector) {
        cout << ch;
    }
    cout << '!';

    return 0;
}
```

Output:

```
https://github.com/wangxwan/Cpp11-14-17-20!
https://github.com/wangxwan/!
```

Two points to note:

1. When iterating over `myvector`, `auto` deduces `ch` as `char`. `ch` represents each element in the container, not an iterator.

2. The first line includes a space before "!" because the range-based `for` loop iterates over the null terminator (`\0`) at the end of the string. The second line doesn't have this space because `myvector` doesn't store the null terminator.

Range-based `for` loops can also iterate over initializer lists:

```c++
#include <iostream>
using namespace std;

int main() {
    for (int num : {1, 2, 3, 4, 5}) {
        cout << num << " ";
    }
    return 0;
}
```

Output:

```
1 2 3 4 5
```

To modify elements during iteration, declare the loop variable as a reference:

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    char arc[] = "abcde";
    vector<char> myvector(arc, arc + 5);

    // Iterate and modify elements
    for (auto& ch : myvector) {
        ch++;
    }

    // Iterate and print modified elements
    for (auto ch : myvector) {
        cout << ch;
    }

    return 0;
}
```

Output:

```
bcdef
```

The first loop modifies elements using a reference (`auto& ch`), while the second loop prints the modified values.

When choosing between a regular variable and a reference in the `declaration`, consider whether you need to modify elements. If so, use a reference. Otherwise, a `const&` (constant reference) is recommended for efficiency (avoids copying), or a regular variable can be used.


