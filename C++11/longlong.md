# C++11 - `long long`: Extended Integer Type

C++11 provides various integer data types based on their size, as shown in the table below. The standard also specifies the minimum number of bits each type must occupy.

| Integer Type | Equivalent Type | Minimum Bits in C++11 |
|---|---|---|
| `short` | `short int` (signed short integer) | At least 16 bits (2 bytes) |
| `signed short` |  |  |
| `signed short int` |  |  |
| `unsigned short` | `unsigned short int` (unsigned short integer) |  |
| `unsigned short int` |  |  |
| `int` | `int` (signed integer) | At least 16 bits (2 bytes) |
| `signed` |  |  |
| `signed int` |  |  |
| `unsigned` | `unsigned int` (unsigned integer) |  |
| `unsigned int` |  |  |
| `long` | `long int` (signed long integer) | At least 32 bits (4 bytes) |
| `long int` |  |  |
| `signed long` |  |  |
| `signed long int` |  |  |
| `unsigned long` | `unsigned long int` (unsigned long integer) |  |
| `unsigned long int` |  |  |
| **`long long` (C++11)** | **`long long int` (signed long long integer)** | **At least 64 bits (8 bytes)** |
| **`long long int` (C++11)** |  |  |
| **`signed long long` (C++11)** |  |  |
| **`signed long long int` (C++11)** |  |  |
| **`unsigned long long` (C++11)** | **`unsigned long long int` (unsigned long long integer)** |  |
| **`unsigned long long int` (C++11)** |  |  |

> C++11 mandates that each integer type must have both signed and unsigned versions, occupying the same amount of storage (bits). The standard only specifies the minimum storage size; different platforms can use more bits.

Among these types, `long long` is a new addition in C++11. Let's explore it in detail.

The inclusion of `long long` in C++11 has a history. A proposal to add it to C++98 was rejected. However, after its adoption in C99 (a C language standard) and widespread compiler support, the C++ committee decided to include it in C++11.

Similar to `long` integers requiring an "L" or "l" suffix, `long long` integers also need specific suffixes:

- **Signed `long long`:** "LL" or "ll". For example, "10LL" represents the signed long long integer 10.
- **Unsigned `long long`:** "ULL", "ull", "Ull", or "uLL". For example, "10ULL" represents the unsigned long long integer 10.

> Without any suffix, integers default to `int` type.

Knowing the range of values a data type can hold is essential. For `long long`, the `<climits>` header file provides three macros:

1. `LLONG_MIN`: Minimum value of `long long`.
2. `LLONG_MAX`: Maximum value of `long long`.
3. `ULLONG_MAX`: Maximum value of `unsigned long long` (the minimum is 0).

Example:

```c++
#include <iostream>
#include <iomanip>
#include <climits>
using namespace std;

int main() {
    cout << "Minimum long long: " << LLONG_MIN << " " << hex << LLONG_MIN << "\n";
    cout << dec << "Maximum long long: " << LLONG_MAX << " " << hex << LLONG_MAX << "\n";
    cout << dec << "Maximum unsigned long long: " << ULLONG_MAX << " " << hex << ULLONG_MAX;
    return 0;
}
```

Output (may vary):

```
Minimum long long: -9223372036854775808 8000000000000000
Maximum long long: 9223372036854775807 7fffffffffffffff
Maximum unsigned long long: 18446744073709551615 ffffffffffffffff
```

This code outputs the minimum and maximum values in both decimal and hexadecimal. On this platform (Windows 10 64-bit), `long long` occupies 64 bits (8 bytes). You can run this code on your machine to determine the size of `long long` on your system.


