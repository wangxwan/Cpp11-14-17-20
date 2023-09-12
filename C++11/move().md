# C++11 - `move()` Function: Forcing Lvalue-to-Rvalue Conversion

From the [Move Constructors](MoveConstructors.md) section, we learned that C++11's rvalue references allow us to define move constructors for classes. When initializing an object using an rvalue (often a temporary object) of the same class, the compiler prioritizes the move constructor.

However, move constructors are specifically invoked when using rvalues for initialization. What if we want to use an lvalue (an object with a name and address) of the same class to initialize a new object, but still leverage the move constructor? C++11 provides a solution: the `move()` function.

Despite its name, `move()` doesn't actually move any data. Its sole purpose is to **cast an lvalue to an rvalue**.

> This unique functionality makes `move()` particularly useful for implementing move semantics.

Using `move()` is straightforward. The syntax is:

```c++
move(arg)
```

where `arg` is the lvalue object to be converted. The function returns an rvalue reference to `arg`.

**Example 1: Basic `move()` Usage**

```c++
#include <iostream>
using namespace std;

class movedemo {
public:
    movedemo() : num(new int(0)) {
        cout << "construct!" << endl;
    }
    // Copy constructor
    movedemo(const movedemo& d) : num(new int(*d.num)) {
        cout << "copy construct!" << endl;
    }
    // Move constructor
    movedemo(movedemo&& d) : num(d.num) {
        d.num = NULL;
        cout << "move construct!" << endl;
    }
public:     // Should be private, using public for demonstration
    int* num;
};

int main() {
    movedemo demo;
    cout << "demo2:\n";
    movedemo demo2 = demo; // Copy constructor called
    // cout << *demo2.num << endl;   // Works fine
    cout << "demo3:\n";
    movedemo demo3 = std::move(demo); // Move constructor called
    // demo.num is now NULL, accessing it here will cause a runtime error
    // cout << *demo.num << endl;
    return 0;
}
```

Output:

```
construct!
demo2:
copy construct!
demo3:
move construct!
```

The output and the initialization of `demo2` and `demo3` demonstrate the following:

- When `demo` (an lvalue) is used directly to initialize `demo2`, the copy constructor is called.
- When `move(demo)` is used to create an rvalue reference to `demo` and initialize `demo3`, the move constructor is called.

> Note: Calling the copy constructor doesn't affect the original `demo` object. However, calling the move constructor modifies `demo` by setting `demo.num` to `NULL`, which is why accessing it afterward (line 30) would lead to a runtime error.

**Example 2: Using `move()` with Nested Objects**

```c++
#include <iostream>
using namespace std;

class first {
public:
    first() : num(new int(0)) {
        cout << "construct!" << endl;
    }
    // Move constructor
    first(first&& d) : num(d.num) {
        d.num = NULL;
        cout << "first move construct!" << endl;
    }
public:    // Should be private, using public for demonstration
    int* num;
};

class second {
public:
    second() : fir() {}
    // Use first's move constructor to initialize fir
    second(second&& sec) : fir(move(sec.fir)) {
        cout << "second move construct" << endl;
    }
public:    // Should be private, using public for demonstration
    first fir;
};

int main() {
    second oth;
    second oth2 = move(oth);
    // cout << *oth.fir.num << endl;   // Runtime error
    return 0;
}
```

Output:

```
construct!
first move construct!
second move construct
```

This code defines two classes, `first` and `second`, with `second` containing an object of type `first`. Notice the two uses of `move()`:

- **Line 31:** `oth` is an lvalue. To initialize `oth2` using the move constructor, we first use `move(oth)` to obtain an rvalue reference to `oth`.

- **Line 22:** `oth` contains a `first` object (`oth.fir`), which is also an lvalue. Therefore, when initializing `fir` in `second`'s move constructor, we use `move(sec.fir)` to create an rvalue reference to `sec.fir`, allowing the use of `first`'s move constructor.
