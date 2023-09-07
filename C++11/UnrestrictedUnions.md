# C++11 - Unrestricted Unions

In C/C++, a union is a user-defined data type where all members share the same memory location. Older C++ versions, for compatibility with C, imposed significant restrictions on the types of members allowed in a union. C++11 removes these restrictions, deeming them unnecessary.

C++11 allows any non-reference type as a union member, leading to the term **unrestricted unions**. For example:

```c++
class Student {
public:
    Student(bool g, int a) : gender(g), age(a) {}
private:
    bool gender;
    int age;
};

union T {
    Student s;  // Member with non-POD type, would cause errors in older compilers
    char name[10];
};

int main() {
    return 0;
}
```

In earlier versions, this code would fail because `Student` has a user-defined constructor, making it a non-POD type. This restriction, imposed for C compatibility, proved unnecessary in practice.

We'll discuss POD types in detail shortly.

Let's explore the specific improvements C++11 brings to unions.

## 1. Allowing Non-POD Types

C++98 prohibited non-POD types as union members, a restriction lifted in C++11.

POD (Plain Old Data) is a crucial concept in C++. Here's a brief overview:

A POD type (including `class`, `union`, and `struct`) typically exhibits these characteristics:

1. No user-defined constructors, destructors, copy constructors, or move constructors.

2. No virtual functions or virtual base classes.

3. Non-static members must be declared `public`.

4. The first non-static member's type must differ from its base class, e.g.:

   ```c++
   class B1 {};
   class B2 : B1 {
       B1 b; // First non-static member is of base class type
   };
   ```

   `B2` is not a POD type.

5. In inheritance:

   - If the derived class has non-static members, it can have only one base class, and that base class must have only static members.
   - If the base class has non-static members, the derived class must not have non-static members.

   ```c++
   class B1 {
       static int n;
   };
   class B2 : B1 {
       int n1; // B2 is POD: non-static member, base has only static
   };
   class B3 : B2 {
       static int n2; // B3 is POD: base has non-static, derived doesn't
   };
   ```

6. All non-static data members must also be POD (recursive definition).

7. All C-compatible data types are POD (as long as `struct` and `union` definitions adhere to these rules).

## 2. Allowing Static Members

C++11 removes the restriction on static members in unions:

```c++
union U {
    static int func() {
        int n = 3;
        return n;
    }
};
```

However, static member variables can only be defined within the union but not used outside, limiting their practical use.

## Assignment Considerations for Unrestricted Unions

C++11 states that if an unrestricted union contains a non-POD member with a user-defined constructor, the union's default constructor is deleted. Other special member functions, like the copy constructor, copy assignment operator, and destructor, are also deleted.

This can lead to object construction failures:

```c++
#include <string>
using namespace std;

union U {
    string s;
    int n;
};

int main() {
    U u;   // Construction fails: U's constructor is deleted
    return 0;
}
```

Here, `string` has a custom constructor, so `U`'s constructor is deleted, causing the definition of `u` to fail.

A common solution involves placement new:

```c++
#include <string>
using namespace std;

union U {
    string s;
    int n;
public:
    U() { new(&s) string; }
    ~U() { s.~string(); }
};

int main() {
    U u; // Now works
    return 0;
}
```

We use placement new to construct `s` at its address (`&s`), essentially calling `string`'s constructor. Importantly, we also call `string`'s destructor in `U`'s destructor.

## What is Placement New?

Placement new is an advanced usage of `new`, allowing object creation at a specific memory address, either on the stack or the heap. In contrast, regular `new` (operator new) allocates memory on the heap.

Placement new syntax:

```c++
new (address) ClassConstruct(...);
```

- `address`: Address of pre-allocated memory (stack or heap).
- `ClassConstruct(...)`: Call to the class's constructor (parentheses optional if no arguments).

Placement new constructs the object at the provided address without allocating new memory. In our example, it utilizes `s`'s memory space.

## Anonymous Unions and "Enum-Style Classes"

An anonymous union is a union without a name:

```c++
union {
    int x;
}; // Anonymous union
```

Unrestricted unions can also be anonymous. When used within a class definition, they enable a pattern resembling "enum-style classes":

```c++
#include <cstring>
using namespace std;

class Student {
public:
    Student(bool g, int a) : gender(g), age(a) {}
    bool gender;
    int age;
};

class Singer {
public:
    enum Type { STUDENT, NATIVE, FOREIGNER };

    Singer(bool g, int a) : s(g, a) { t = STUDENT; }
    Singer(int i) : id(i) { t = NATIVE; }
    Singer(const char* n, int size) {
        int copySize = (size > 9) ? 9 : size;
        memcpy(name, n, copySize);
        name[copySize] = '\0';
        t = FOREIGNER;
    }

    ~Singer() {}

private:
    Type t;
    union {
        Student s;
        int id;
        char name[10];
    }; // Anonymous unrestricted union
};

int main() {
    Singer student(true, 13);
    Singer native(310217);
    Singer foreigner("J Michael", 9);
    return 0;
}
```

Here, the anonymous unrestricted union acts as a "variant member" of `Singer`, providing flexibility not possible in C++98 (which would raise an error for the `Student s` member). This pattern allows the class to hold different data types based on its state, indicated by the `Type` enum.
