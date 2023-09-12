# C++11 - Move Constructors

In the [Rvalue References](RvalueReferences.md) section, we discussed the meaning and usage of rvalue references in C++, mentioning that they are primarily used for implementing move semantics and perfect forwarding. We'll cover perfect forwarding in a later section. This section focuses on understanding move semantics and how to implement them.

## What is Move Semantics in C++11?

Before C++11 (in C++98/03), initializing a new object of a class with another object of the same class relied solely on the class's copy constructor. The copy constructor simply creates a new object with an identical copy of the data from the other object.

> Note: When a class has pointer member variables, the copy constructor must perform a deep copy (not a shallow copy) of these pointers.

Example:

```c++
#include <iostream>
using namespace std;

class demo {
public:
    demo() : num(new int(0)) {
        cout << "construct!" << endl;
    }
    // Copy constructor
    demo(const demo& d) : num(new int(*d.num)) {
        cout << "copy construct!" << endl;
    }
    ~demo() {
        cout << "class destruct!" << endl;
    }
private:
    int* num;
};

demo get_demo() {
    return demo();
}

int main() {
    demo a = get_demo();
    return 0;
}
```

> In this example, we define a custom copy constructor for the `demo` class. When copying the `d.num` pointer member, it must perform a deep copy, copying both the pointer itself and the memory it points to. Otherwise, if multiple objects' pointer members point to the same heap memory, their destructors would attempt to free that memory multiple times, leading to errors.

This code defines a `get_demo()` function that returns a `demo` object, used to initialize the `a` object in `main()`. The initialization process involves these steps:

1. `demo()` inside `get_demo()` is executed, calling the default constructor to create an anonymous object.

2. `return demo()` calls the copy constructor to create a copy of the anonymous object and return it as the function's result (the anonymous object is destroyed before the function ends).

3. `a = get_demo()` calls the copy constructor again to copy the temporary object returned by `get_demo()` to `a` (the temporary object is destroyed after this line).

4. Before the program ends, `demo`'s destructor is called to destroy `a`.

Modern compilers often optimize copy operations. Running this code with compilers like VS 2017 or CodeBlocks might show an optimized output:

```
construct!
class destruct!
```

However, compiling and running this code on Linux with `g++ demo.cpp -fno-elide-constructors` (assuming the file is named `demo.cpp`) reveals the full process:

```
construct! <-- demo()
copy construct! <-- return demo()
class destruct! <-- Anonymous object from demo() destroyed
copy construct! <-- a = get_demo()
class destruct! <-- Temporary object from get_demo() destroyed
class destruct! <-- a destroyed
```

As shown, initializing `a` using the copy constructor involves two deep copy operations. While this might be acceptable for small temporary objects, deep copying large amounts of heap memory can significantly impact performance.

> This issue existed in C++ programs written with the C++98/03 standard. However, it often went unnoticed because the creation, destruction, and copying of temporary objects were hidden by compiler optimizations and didn't affect program correctness.

So, how can we avoid the performance overhead of deep copying when initializing objects with pointer members? C++11 introduces move semantics to address this.

## C++ Move Constructors (Implementing Move Semantics)

**Move semantics** refers to initializing class objects with pointer members by moving (transferring ownership of) the memory resources instead of deep copying. Essentially, it "steals" the resources from another object (usually a temporary object).

Consider the `demo` class from the previous example. Its member `num` points to heap memory holding an integer. When initializing `a` with the temporary object returned by `get_demo()`, we can simply shallow copy the temporary object's `num` pointer to `a.num` and then modify the temporary object's `num` to point to `NULL`. This effectively initializes `a.num` without deep copying.

> Temporary objects often serve only to transfer data and are quickly destroyed. Therefore, when initializing a new object with a temporary object, we can directly transfer ownership of the memory resources pointed to by its pointer members to the new object, eliminating the need for deep copying and improving performance.

Here's the modified `demo` class with a move constructor:

```c++
#include <iostream>
using namespace std;

class demo {
public:
    demo() : num(new int(0)) {
        cout << "construct!" << endl;
    }
    demo(const demo& d) : num(new int(*d.num)) {
        cout << "copy construct!" << endl;
    }
    // Move constructor
    demo(demo&& d) : num(d.num) {
        d.num = NULL;
        cout << "move construct!" << endl;
    }
    ~demo() {
        cout << "class destruct!" << endl;
    }
private:
    int* num;
};

demo get_demo() {
    return demo();
}

int main() {
    demo a = get_demo();
    return 0;
}
```

We've added a new constructor that takes an rvalue reference parameter (`demo&& d`). This is the **move constructor**. It performs a shallow copy of `num` and sets `d.num` to `NULL` to prevent double-free errors.

Compiling and running this code on Linux with `g++ demo.cpp -o demo.exe -std=c++0x -fno-elide-constructors` produces the following output:

```
construct!
move construct!
class destruct!
move construct!
class destruct!
class destruct!
```

The output shows that both copy operations are now handled by the move constructor.

Non-const rvalue references can only bind to rvalues. Temporary objects (function return values, lambda expressions, etc.) are rvalues because they have no names and their addresses cannot be taken. When a class has both copy and move constructors, the compiler prioritizes the move constructor when initializing an object with a temporary object. The copy constructor is used only if no suitable move constructor is available.

> In practice, it's common to define both a move constructor and a copy constructor. This allows the compiler to choose the appropriate constructor based on whether the initialization is done with an rvalue or an lvalue.

You might wonder if it's possible to use the move constructor even when initializing an object with an lvalue.

By default, initializing an object with an lvalue uses the copy constructor. To force the use of the move constructor, you can use the `std::move()` function, which converts an lvalue to an rvalue.

> We'll discuss `std::move()` in detail in a later section.


