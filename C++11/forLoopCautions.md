# C++11 - Range-Based for Loop Considerations

The [Range-Based for Loops](forLoop.md) section covered the basic usage of this C++11 feature. This section delves into important considerations for using range-based `for` loops effectively and correctly.

1. **Iterating Over Elements, Not Iterators:**

   When using a range-based `for` loop, the loop variable represents each **element** in the sequence, regardless of whether it's an array, container, or initializer list.

   Example:

   ```c++
   #include <iostream>
   #include <vector>
   using namespace std;

   int main() {
       // Iterate over an initializer list
       for (int ch : {1, 2, 3, 4, 5}) {
           cout << ch;
       }
       cout << endl;

       // Iterate over an array
       char arc[] = "https://github.com/wangxwan/Cpp11-14-17-20";
       for (char ch : arc) {
           cout << ch;
       }
       cout << endl;

       // Iterate over a vector container
       vector<char> myvector(arc, arc + 28);
       for (auto ch : myvector) {
           cout << ch;
       }

       return 0;
   }
   ```

   Output:

   ```
   12345
   https://github.com/wangxwan/Cpp11-14-17-20
   https://github.com/wangxwan/
   ```

   While the first two cases are straightforward, when iterating over a container, it's crucial to understand that `ch` represents the **element itself**, not an iterator pointing to the element.

   Here's another example using a `map` container:

   ```c++
   #include <iostream>
   #include <map>
   #include <string>
   using namespace std;

   int main() {
       map<string, string> mymap {
           {"C++14", "https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B14"},
           {"C++17", "https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B17"},
           {"C++20", "https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B20"}
       };

       for (pair<string, string> ch : mymap) {
           cout << ch.first << " " << ch.second << endl;
       }

       return 0;
   }
   ```

   Output:

   ```
   C++14 https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B14
   C++17 https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B17
   C++20 https://github.com/wangxwan/Cpp11-14-17-20/tree/master/C%2B%2B20
   ```

   Since `map` stores `pair` objects, the loop variable `ch` is of type `pair<string, string>`.

   Range-based `for` loops can also directly iterate over string literals:

   ```c++
   for (char ch : "https://github.com/wangxwan/Cpp11-14-17-20") {
       cout << ch;
   }
   ```

   String literals are treated as arrays (`const char[]`) by the compiler, making them valid for range-based iteration.

   > You can also iterate over `std::string` objects using `char` as the loop variable type.

2. **Iterating Over Function Return Values:**

   Range-based `for` loops can iterate over sequences returned by functions, including `std::string` and container objects:

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   using namespace std;

   string str = "https://github.com/wangxwan/Cpp11-14-17-20";
   vector<int> myvector = {1, 2, 3, 4, 5};

   string retStr() {
       return str;
   }

   vector<int> retVector() {
       return myvector;
   }

   int main() {
       // Iterate over a string returned by a function
       for (char ch : retStr()) {
           cout << ch;
       }
       cout << endl;

       // Iterate over a vector returned by a function
       for (int num : retVector()) {
           cout << num << " ";
       }

       return 0;
   }
   ```

   Output:

   ```
   https://github.com/wangxwan/Cpp11-14-17-20
   1 2 3 4 5
   ```

   However, iterating over arrays returned as pointers is not supported:

   ```c++
   // Incorrect example
   #include <iostream>
   using namespace std;

   char str[] = "https://github.com/wangxwan/Cpp11-14-17-20";

   char* retStr() {
       return str;
   }

   int main() {
       for (char ch : retStr()) { // Error: Cannot iterate over a pointer
           cout << ch;
       }

       return 0;
   }
   ```

   Range-based `for` loops require a well-defined range, which a pointer doesn't provide.

3. **Single Function Call for Returned Sequences:**

   When iterating over a `std::string` or container returned by a function, the function is called only **once** at the beginning of the loop:

   ```c++
   #include <iostream>
   #include <string>
   using namespace std;

   string str = "https://github.com/wangxwan/Cpp11-14-17-20";

   string retStr() {
       cout << "retStr:" << endl;
       return str;
   }

   int main() {
       // Iterate over a string returned by a function
       for (char ch : retStr()) {
           cout << ch;
       }

       return 0;
   }
   ```

   Output:

   ```
   retStr:
   https://github.com/wangxwan/Cpp11-14-17-20
   ```

   `retStr()` is called only once before the loop starts iterating over the returned string.

4. **Modifying Containers During Iteration:**

   - **Associative Containers:** Modifying keys in associative containers (e.g., `map`, `unordered_map`) or elements in sets (e.g., `set`, `unordered_set`) is prohibited due to their underlying data structures. Attempting to do so within a range-based `for` loop will lead to undefined behavior.

   - **Invalidating Iterators:** Range-based `for` loops internally use iterators. Modifying the container's size (e.g., adding or removing elements) within the loop can invalidate these iterators, causing errors:

     ```c++
     #include <iostream>
     #include <vector>

     int main(void) {
         std::vector<int> arr = {1, 2, 3, 4, 5};

         for (auto val : arr) {
             std::cout << val << std::endl;
             arr.push_back(10); // Modifies container size, invalidating iterators
         }

         return 0;
     }
     ```

     This code might produce unexpected output due to iterator invalidation.

   Avoid modifying the container's size within a range-based `for` loop to prevent such issues.


