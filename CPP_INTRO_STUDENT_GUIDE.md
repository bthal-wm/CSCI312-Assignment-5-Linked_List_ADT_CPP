# C++ Introduction: From C Structs to Classes and RAII
## Student Guide for CSCI 312

**Prerequisites:** Completion of Assignment 4 (C Linked List ADT)

**Learning Objectives:**
- Understand the differences between C and C++ programming
- Convert C structs and functions to C++ classes with methods
- Implement proper memory management using constructors and destructors
- Apply the Rule of Three for safe resource management
- Understand RAII (Resource Acquisition Is Initialization)

---

## Table of Contents
1. [Part 1: From C to C++ Classes](#part1)
2. [Part 2: Constructors and Member Functions](#part2)
3. [Part 3: Destructors and RAII](#part3)
4. [Part 4: The Rule of Three](#part4)
5. [Complete Code Examples](#code-examples)
6. [Practice Exercises](#exercises)

---

<a name="part1"></a>
## Part 1: From C to C++ Classes

### Why C++?

In your Assignment 4 (C Linked List), you likely encountered these problems:

```c
// C Version - Easy to mess up
LinkedList *list = list_create();
list_insert_head(list, 10);
list_insert_tail(20);
// ... code ...
list_destroy(list);  // ❌ Easy to forget = memory leak!
```

**Common C problems:**
1. **Manual Memory Management** - Easy to forget `free()`, causing memory leaks
2. **No Encapsulation** - Anyone can break your data: `list->head = NULL;`
3. **Verbose Syntax** - Every function needs explicit list parameter
4. **Type Casts** - `malloc` returns `void*`, requiring casts

**C++ Solutions:**
- Automatic memory management (destructors)
- Data protection (private members)
- Cleaner syntax (methods)
- Type-safe allocation (`new`/`delete`)

---

### C++ Structs: First Improvements

C++ is mostly backward-compatible with C, but with enhancements:

**C Code:**
```c
typedef struct Node {
    int data;
    struct Node *next;
} Node;

typedef struct LinkedList {
    Node *head;
    size_t size;
} LinkedList;
```

**C++ Code:**
```cpp
#include <iostream>
using namespace std;

// No typedef needed!
struct Node {
    int data;
    Node* next;  // Can use Node directly
};

struct LinkedList {
    Node* head;
    size_t size;
};

int main() {
    LinkedList list;  // No * needed for stack allocation
    list.head = nullptr;  // nullptr instead of NULL
    list.size = 0;
    
    cout << "List size: " << list.size << endl;
    return 0;
}
```

**Key Differences:**
- ✅ No `typedef` needed - struct name is automatically a type
- ✅ `nullptr` instead of `NULL` (type-safe)
- ✅ Can create on stack without pointers
- ✅ `cout` instead of `printf`

---

### From `struct` to `class`: Access Control

The main difference between `struct` and `class` is default access:
- `struct` members are **public** by default
- `class` members are **private** by default

```cpp
#include <iostream>
using namespace std;

class LinkedList {
private:  // Hidden from outside code
    struct Node {
        int data;
        Node* next;
        Node(int val) : data(val), next(nullptr) {}
    };
    
    Node* head;
    size_t list_size;

public:  // Accessible from outside
    LinkedList() {  // Constructor
        head = nullptr;
        list_size = 0;
    }
    
    size_t size() const {  // Member function
        return list_size;
    }
    
    bool isEmpty() const {
        return head == nullptr;
    }
};

int main() {
    LinkedList list;  // Constructor automatically called!
    
    cout << "List size: " << list.size() << endl;
    cout << "Is empty? " << (list.isEmpty() ? "yes" : "no") << endl;
    
    // This would cause a compile error:
    // list.head = nullptr;  // ERROR: 'head' is private
    
    return 0;
}
```

**Encapsulation - The Vending Machine Analogy:**
```
┌──────────────────────────┐
│   LINKED LIST CLASS      │
├──────────────────────────┤
│ PRIVATE (locked inside): │
│   • head pointer         │ ← Can't reach in!
│   • node structure       │
│   • size counter         │
├──────────────────────────┤
│ PUBLIC (buttons):        │
│   [Insert Head]          │ ← Can only use these
│   [Insert Tail]          │
│   [Get Size]             │
└──────────────────────────┘
```

---

<a name="part2"></a>
## Part 2: Constructors and Member Functions

### Constructors: Guaranteed Initialization

A **constructor** is a special function that runs when an object is created:

```cpp
class LinkedList {
public:
    // Constructor - same name as class, no return type
    LinkedList() : head(nullptr), list_size(0) {
        cout << "LinkedList created!" << endl;
    }
};

int main() {
    LinkedList list;  // Constructor called automatically!
    // head and list_size are guaranteed to be initialized
}
```

**Member Initializer List:**
```cpp
LinkedList() : head(nullptr), list_size(0) {}
//           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//           Initializes members BEFORE constructor body runs
```

This is preferred because:
- More efficient than assignment
- Required for const members and references
- Idiomatic C++ style

---

### Member Functions: Methods

Instead of passing the object as a parameter, make functions part of the class:

**C Version:**
```c
void list_insert_head(LinkedList *list, int value) {
    Node *new_node = malloc(sizeof(Node));
    new_node->data = value;
    new_node->next = list->head;
    list->head = new_node;
    list->size++;
}

// Usage:
list_insert_head(mylist, 42);
```

**C++ Version:**
```cpp
class LinkedList {
public:
    void insertHead(int value) {  // No list parameter!
        Node* new_node = new Node(value);
        new_node->next = head;  // Uses this object's head
        head = new_node;
        list_size++;
    }
};

// Usage:
list.insertHead(42);  // Much cleaner!
```

**Key Concepts:**

1. **No explicit object parameter** - Member functions automatically work on the current object
2. **Direct member access** - Just use `head`, not `list->head`
3. **`new` instead of `malloc`** - Type-safe, calls constructor automatically
4. **`const` correctness** - Mark functions that don't modify state

```cpp
void print() const {  // const = won't modify object
    Node* current = head;
    while (current != nullptr) {
        cout << current->data << " -> ";
        current = current->next;
    }
    cout << "nullptr" << endl;
}
```

---

### Complete Example: Basic LinkedList

```cpp
#include <iostream>
#include <cstddef>
using namespace std;

class LinkedList {
private:
    struct Node {
        int data;
        Node* next;
        Node(int val) : data(val), next(nullptr) {}
    };
    
    Node* head;
    size_t list_size;

public:
    LinkedList() : head(nullptr), list_size(0) {}
    
    void insertHead(int value) {
        Node* new_node = new Node(value);
        new_node->next = head;
        head = new_node;
        list_size++;
    }
    
    void insertTail(int value) {
        Node* new_node = new Node(value);
        
        if (head == nullptr) {
            head = new_node;
        } else {
            Node* current = head;
            while (current->next != nullptr) {
                current = current->next;
            }
            current->next = new_node;
        }
        list_size++;
    }
    
    void print() const {
        Node* current = head;
        cout << "List: ";
        while (current != nullptr) {
            cout << current->data << " -> ";
            current = current->next;
        }
        cout << "nullptr" << endl;
    }
    
    size_t size() const {
        return list_size;
    }
    
    bool isEmpty() const {
        return head == nullptr;
    }
};

int main() {
    LinkedList list;
    
    list.insertHead(10);
    list.insertHead(20);
    list.insertTail(5);
    
    list.print();  // Output: List: 20 -> 10 -> 5 -> nullptr
    cout << "Size: " << list.size() << endl;  // Output: Size: 3
    
    return 0;
}
```

**Compile and run:**
```bash
g++ -o list_basic list_basic.cpp
./list_basic
```

---

<a name="part3"></a>
## Part 3: Destructors and RAII

### The Memory Leak Problem

With the code above, we have a **memory leak**:

```cpp
void leaky() {
    LinkedList list;
    list.insertHead(10);
    list.insertHead(20);
}  // Nodes never freed - MEMORY LEAK!
```

The dynamically allocated nodes are never deleted!

---

### Destructors: Automatic Cleanup

A **destructor** is called automatically when an object is destroyed:

```cpp
class LinkedList {
public:
    // Constructor
    LinkedList() : head(nullptr), list_size(0) {
        cout << "LinkedList created" << endl;
    }
    
    // Destructor - note the tilde (~)
    ~LinkedList() {
        cout << "LinkedList destroyed - freeing memory" << endl;
        
        Node* current = head;
        while (current != nullptr) {
            Node* next = current->next;  // ✅ Save next BEFORE delete
            delete current;
            current = next;
        }
    }
};

int main() {
    LinkedList list;
    list.insertHead(10);
    list.insertHead(20);
}  // Destructor called automatically here!
```

**Destructor Properties:**
- Name is `~ClassName()`
- No parameters, no return type
- Only one destructor per class
- Called automatically when object goes out of scope

**When are destructors called?**

```cpp
int main() {
    LinkedList list1;  // Constructor
    
    {
        LinkedList list2;  // Constructor
    }  // list2 destructor called here
    
}  // list1 destructor called here
```

---

### Critical Pattern: Save Next Before Delete

**❌ WRONG - Undefined Behavior:**
```cpp
while (current != nullptr) {
    delete current;
    current = current->next;  // ❌ Accessing freed memory!
}
```

**✅ CORRECT:**
```cpp
while (current != nullptr) {
    Node* next = current->next;  // Save while valid
    delete current;              // Now safe to delete
    current = next;              // Use saved value
}
```

---

### RAII: Resource Acquisition Is Initialization

**RAII** is a fundamental C++ idiom:

```
Constructor:  Acquire resources (allocate memory)
Destructor:   Release resources (free memory)

Result: Automatic cleanup, exception-safe
```

**Example:**
```cpp
void automatic_cleanup() {
    LinkedList list;     // Constructor allocates
    list.insertHead(10);
    // ... use list ...
    
    if (some_condition) {
        return;  // Early return
    }
    
    // ... more code ...
}  // Destructor frees - AUTOMATIC in ALL code paths!
```

**Compare to C:**
```c
void manual_cleanup() {
    LinkedList* list = list_create();
    list_insert_head(list, 10);
    
    if (some_condition) {
        list_destroy(list);  // ❌ Must remember!
        return;
    }
    
    list_destroy(list);  // ❌ Must remember!
}
```

**RAII Benefits:**
- ✅ Can't forget to clean up
- ✅ Works even if exception thrown
- ✅ Works in all code paths (return, break, continue)
- ✅ Composable (objects containing objects all clean up automatically)

---

<a name="part4"></a>
## Part 4: The Rule of Three

### The Copy Disaster

Consider this code:

```cpp
int main() {
    LinkedList list1;
    list1.insertHead(10);
    list1.insertHead(20);
    
    LinkedList list2 = list1;  // What happens?
    
    return 0;
}  // CRASH!
```

**Why does it crash?**

The compiler-generated copy constructor does a **shallow copy**:

```
After list1.insertHead(20):
┌────────┐
│ list1  │
│ head ──┼──→ [20]──→[10]──→nullptr
└────────┘

After list2 = list1 (shallow copy):
┌────────┐
│ list1  │
│ head ──┼──╮
└────────┘  │
            ├──→ [20]──→[10]──→nullptr  ← SAME NODES!
┌────────┐  │
│ list2  │  │
│ head ──┼──╯
└────────┘

When destructors run:
1. list2 destructor frees [20] and [10]
2. list1 destructor tries to free ALREADY FREED memory
   → DOUBLE FREE → CRASH!
```

---

### The Rule of Three

**If your class manages resources, you need all three:**

1. **Destructor** - `~ClassName()`
2. **Copy Constructor** - `ClassName(const ClassName& other)`
3. **Copy Assignment Operator** - `operator=(const ClassName& other)`

---

### 1. Destructor (Already Covered)

```cpp
~LinkedList() {
    Node* current = head;
    while (current != nullptr) {
        Node* next = current->next;
        delete current;
        current = next;
    }
}
```

---

### 2. Copy Constructor: Deep Copy

Create **new nodes** with the same values:

```cpp
class LinkedList {
private:
    // Helper function for deep copy
    void copyFrom(const LinkedList& other) {
        Node* current = other.head;
        Node* last = nullptr;
        
        while (current != nullptr) {
            Node* new_node = new Node(current->data);
            
            if (head == nullptr) {
                head = new_node;
            } else {
                last->next = new_node;
            }
            
            last = new_node;
            current = current->next;
        }
    }

public:
    // Copy constructor
    LinkedList(const LinkedList& other) : head(nullptr), list_size(0) {
        copyFrom(other);
        list_size = other.list_size;
    }
};
```

**Result - Independent copies:**
```
After deep copy:

list1.head → [20] → [10] → nullptr

list2.head → [20] → [10] → nullptr
             (NEW)  (NEW)

When destructors run:
1. list2 destructor frees its [20], [10] ✅
2. list1 destructor frees its [20], [10] ✅
No crash!
```

**When is copy constructor called?**

```cpp
LinkedList list1;
list1.insertHead(10);

// 1. Direct initialization
LinkedList list2 = list1;  // Copy constructor

// 2. Passing by value
void func(LinkedList list) {  // Copy constructor
    // list is a copy
}
func(list1);

// 3. Returning by value
LinkedList createList() {
    LinkedList temp;
    return temp;  // Copy constructor
}
```

---

### 3. Copy Assignment Operator

**Difference from copy constructor:**
- Copy constructor: creates a **new** object
- Copy assignment: overwrites an **existing** object

```cpp
LinkedList list1;
list1.insertHead(10);

// Copy constructor - list2 doesn't exist yet
LinkedList list2 = list1;

// Copy assignment - list3 already exists
LinkedList list3;
list3 = list1;  // operator= called
```

**Implementation:**

```cpp
LinkedList& operator=(const LinkedList& other) {
    // 1. Check for self-assignment
    if (this == &other) {
        return *this;
    }
    
    // 2. Free existing nodes
    Node* current = head;
    while (current != nullptr) {
        Node* next = current->next;
        delete current;
        current = next;
    }
    
    // 3. Copy from other
    head = nullptr;
    list_size = 0;
    copyFrom(other);
    list_size = other.list_size;
    
    // 4. Return *this
    return *this;
}
```

**Four critical steps:**

1. **Self-assignment check**
   ```cpp
   if (this == &other) return *this;
   ```
   Prevents: `list = list;` from breaking

2. **Free existing resources**
   ```cpp
   // Delete old nodes first
   ```

3. **Copy new resources**
   ```cpp
   copyFrom(other);
   ```

4. **Return *this**
   ```cpp
   return *this;  // Enables: a = b = c;
   ```

---

<a name="code-examples"></a>
## Complete Code Examples

### Full LinkedList with Rule of Three

```cpp
#include <iostream>
#include <cstddef>
using namespace std;

class LinkedList {
private:
    struct Node {
        int data;
        Node* next;
        Node(int val) : data(val), next(nullptr) {
            cout << "  Node(" << val << ") created" << endl;
        }
        ~Node() {
            cout << "  Node(" << data << ") destroyed" << endl;
        }
    };
    
    Node* head;
    size_t list_size;
    
    // Helper for deep copy
    void copyFrom(const LinkedList& other) {
        Node* current = other.head;
        Node* last = nullptr;
        
        while (current != nullptr) {
            Node* new_node = new Node(current->data);
            
            if (head == nullptr) {
                head = new_node;
            } else {
                last->next = new_node;
            }
            
            last = new_node;
            current = current->next;
        }
    }
    
    // Helper for cleanup
    void clear() {
        Node* current = head;
        while (current != nullptr) {
            Node* next = current->next;
            delete current;
            current = next;
        }
        head = nullptr;
        list_size = 0;
    }

public:
    // Constructor
    LinkedList() : head(nullptr), list_size(0) {
        cout << "LinkedList() constructor" << endl;
    }
    
    // Destructor
    ~LinkedList() {
        cout << "~LinkedList() destructor" << endl;
        clear();
    }
    
    // Copy constructor
    LinkedList(const LinkedList& other) : head(nullptr), list_size(0) {
        cout << "LinkedList(const LinkedList&) copy constructor" << endl;
        copyFrom(other);
        list_size = other.list_size;
    }
    
    // Copy assignment operator
    LinkedList& operator=(const LinkedList& other) {
        cout << "LinkedList::operator= copy assignment" << endl;
        
        if (this == &other) {
            return *this;
        }
        
        clear();
        copyFrom(other);
        list_size = other.list_size;
        
        return *this;
    }
    
    // Member functions
    void insertHead(int value) {
        Node* new_node = new Node(value);
        new_node->next = head;
        head = new_node;
        list_size++;
    }
    
    void insertTail(int value) {
        Node* new_node = new Node(value);
        
        if (head == nullptr) {
            head = new_node;
        } else {
            Node* current = head;
            while (current->next != nullptr) {
                current = current->next;
            }
            current->next = new_node;
        }
        list_size++;
    }
    
    void print() const {
        cout << "List: ";
        Node* current = head;
        while (current != nullptr) {
            cout << current->data << " -> ";
            current = current->next;
        }
        cout << "nullptr" << endl;
    }
    
    size_t size() const {
        return list_size;
    }
    
    bool isEmpty() const {
        return head == nullptr;
    }
};

int main() {
    cout << "=== Creating list1 ===" << endl;
    LinkedList list1;
    list1.insertHead(10);
    list1.insertHead(20);
    list1.insertHead(30);
    list1.print();
    
    cout << "\n=== Copy constructor (list2 = list1) ===" << endl;
    LinkedList list2 = list1;
    list2.print();
    
    cout << "\n=== Copy assignment (list3 = list1) ===" << endl;
    LinkedList list3;
    list3 = list1;
    list3.print();
    
    cout << "\n=== Modifying list1 ===" << endl;
    list1.insertHead(40);
    cout << "list1: ";
    list1.print();
    cout << "list2: ";
    list2.print();
    cout << "list3: ";
    list3.print();
    
    cout << "\n=== Exiting main (watch destructors) ===" << endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++11 -o linkedlist linkedlist.cpp
./linkedlist
```

**Expected output:**
```
=== Creating list1 ===
LinkedList() constructor
  Node(10) created
  Node(20) created
  Node(30) created
List: 30 -> 20 -> 10 -> nullptr

=== Copy constructor (list2 = list1) ===
LinkedList(const LinkedList&) copy constructor
  Node(30) created
  Node(20) created
  Node(10) created
List: 30 -> 20 -> 10 -> nullptr

=== Copy assignment (list3 = list1) ===
LinkedList() constructor
LinkedList::operator= copy assignment
  Node(30) created
  Node(20) created
  Node(10) created
List: 30 -> 20 -> 10 -> nullptr

=== Modifying list1 ===
  Node(40) created
list1: List: 40 -> 30 -> 20 -> 10 -> nullptr
list2: List: 30 -> 20 -> 10 -> nullptr
list3: List: 30 -> 20 -> 10 -> nullptr

=== Exiting main (watch destructors) ===
~LinkedList() destructor
  Node(30) destroyed
  Node(20) destroyed
  Node(10) destroyed
~LinkedList() destructor
  Node(30) destroyed
  Node(20) destroyed
  Node(10) destroyed
~LinkedList() destructor
  Node(40) destroyed
  Node(30) destroyed
  Node(20) destroyed
  Node(10) destroyed
```

---

<a name="exercises"></a>
## Practice Exercises

### Exercise 1: Add a Remove Function

Implement a `removeHead()` method that removes and returns the first element:

```cpp
int removeHead() {
    // Your code here
    // Remember to:
    // 1. Check if list is empty
    // 2. Save the data
    // 3. Delete the node
    // 4. Update head and size
    // 5. Return the data
}
```

<details>
<summary>Click for solution</summary>

```cpp
int removeHead() {
    if (isEmpty()) {
        throw runtime_error("Cannot remove from empty list");
    }
    
    Node* temp = head;
    int data = temp->data;
    head = head->next;
    delete temp;
    list_size--;
    
    return data;
}
```
</details>

---

### Exercise 2: Test Copy Behavior

Write a test program that demonstrates:
1. Creating a list with values
2. Copying it
3. Modifying the original
4. Verifying the copy is unchanged

---

### Exercise 3: Prevent Copying

Sometimes you don't want objects to be copied. Implement a version that prevents copying:

```cpp
class LinkedList {
private:
    // Delete copy operations
    LinkedList(const LinkedList&) = delete;
    LinkedList& operator=(const LinkedList&) = delete;
    
    // Rest of class...
};
```

Try to compile code that copies - what error do you get?

---

### Exercise 4: Move Semantics (Advanced)

Research C++11 move semantics. How would you implement:
```cpp
LinkedList(LinkedList&& other) noexcept;  // Move constructor
LinkedList& operator=(LinkedList&& other) noexcept;  // Move assignment
```

---

## Summary

### Key Concepts

1. **Classes** provide encapsulation (public/private)
2. **Constructors** guarantee initialization
3. **Destructors** provide automatic cleanup (RAII)
4. **Rule of Three** ensures safe copying:
   - Destructor
   - Copy Constructor
   - Copy Assignment Operator

### C vs C++ Comparison

| Feature | C | C++ |
|---------|---|-----|
| Memory | `malloc`/`free` | `new`/`delete` + RAII |
| Data protection | None | `private`/`public` |
| Initialization | Manual, error-prone | Constructors (automatic) |
| Cleanup | Manual (`_destroy()`) | Destructors (automatic) |
| Copying | Pointer copy | Deep copy (Rule of Three) |
| Syntax | `func(obj, ...)` | `obj.func(...)` |

### Modern C++ (C++11 and beyond)

- **Smart pointers** (`unique_ptr`, `shared_ptr`) - Even safer memory management
- **Move semantics** - Efficient transfer of resources
- **Rule of Five** - Rule of Three + move operations
- **`= default`** - Let compiler generate functions
- **`= delete`** - Explicitly prevent operations

---

## Additional Resources

- [cppreference.com](https://en.cppreference.com/) - Comprehensive C++ reference
- [cplusplus.com](https://cplusplus.com/) - C++ documentation and tutorials
- [LearnCpp.com](https://www.learncpp.com/) - Free C++ tutorial
- *Effective C++* by Scott Meyers - Essential C++ best practices
- *C++ Primer* by Lippman, Lajoie, and Moo - Comprehensive textbook

---

**Next Steps:**
- Complete the practice exercises
- Convert your Assignment 4 solution from C to C++
- Experiment with the code examples
- Ask questions in class or office hours!
