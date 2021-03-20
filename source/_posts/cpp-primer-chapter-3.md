---
title: cpp-primer-chapter-3
date: 2021-03-20 17:48:16
tags: [cpp,  读书笔记]
categories: C++ primer
description:  C++ Primer Chapter Three -- Strings, Vectors, and Arrays
---

![cover](/images/cpp-primer/cover.png)

<!--more-->


###  Namespace 

 - namespace should not be used in header files (since it may cause unexpected name conflictions)


### String

#### ways to initialize a string object

```cpp
string s1; // default initialization; s1 is the empty string
string s2 = s1; // s2 is a copy of s1(copy initialization)
string s3 = "hiya"; // s3 is a copy of the string literal, not including the null character at the end
string s4(10, 'c'); // s4 is cccccccccc (direct initialization)
```

#### string operation

*something new*
```cpp
string line;
getline(cin, line); // read a line from iuput stream
```

#### string.size()
return type: `string::size_type`

*follow up*
1. `size_type` is  a **companion type**, which is used to realize machine-independence. In 32bit-machine, `size_type` is defined as **unsigned int**; while in 64bit-machine, it is defined as **unsigned long**

2. What is the difference between `size_t` and `size_type`?  A :  We can regard `size_t` as a global type, while `size_type` as a local type (related to its container, e.g string::size_type).

3. Should not use `int` to store a `size_type` variable (for the reason that the MAX value of `unsigned` is larger than`int`). (P.S but in fact I usually do this... 😅)

#### string iteration
Three ways to iterate a string:
1. Range-based `for`;
```cpp
for(auto &c: s) {...}  // & is a must when modifying string
```

2. Subscript
```cpp
for(decltype(s.size()) i = 0; i != s.size(); i++) {...}
```

3. Iterator
```cpp
for(auto it = s.begin(); it != s.end(); it++) {...}
```



### Vector

#### initialization

*something new*
In addition to *copy initialization* and *direct initialization*, vector provides *list initialization*
```cpp
vector<string> v1{"a", "an", "the"}; // list initialization

// int 
vector<int> v1(10); // v1 has ten elements with value 0
vector<int> v2{10}; // v2 has one element with value 10
vector<int> v3(10, 1); // v3 has ten elements with value 1
vector<int> v4{10, 1}; // v4 has two elements with values 10 and 1

// string (different from int)
vector<string> v7{10}; // v7 has ten default-initialized elements
vector<string> v8{10, "hi"}; // v8 has ten elements with value "hi"
```

#### ⚠ some vetcor operations invaliate iterators

> Should not add elements to the vector when using iterators, because changing the size of the vector will invalidates all iterators into it.

### Array

#### initialization

```cpp
int n = 10;
const int N = 10;
int a[n]; // error: n is not a constant expression
int a[N]; // ok: all elements are initialized to 0
int main()
{
	int m = 10;
	int b[m]; // ok: but the elements are random initialized
	const char a1[5] = "Jacky"; // error: no space for the null!
	return 0;
}
```

#### pointer and reference to array

```cpp
int arr[10];
int (*p) [10] = &arr; // p is a pointer points to 10 ints array.
int (&r) [10] = arr; // p is a reference binds to 10 ints array.
```
Trick to understand the type easily: **read from inside out**.

#### something special for array

 - array and pointer is closely interwined.
```cpp
string nums[] = {"one", "two", "three"}; // array of strings
string *p = &nums[0]; // p points to the first element in nums
string *p2 = nums; // equivalent to p2 = &nums[0]

int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints
auto ia2(ia); // ia2 is an int* that points to the first element in ia
ia2 = 42; // error: ia2 is a pointer, and we can't assign an int to a pointer

//pointers are iterators
int arr[] = {0,1,2,3,4,5,6,7,8,9};
int *p = arr; // p points to the first element in arr
++p; // p points to arr[1]

int ia[] = {0,2,4,6,8}; // array with 5 elements of type int
int last = *(ia + 4); // ok: initializes last to 8, the value of ia[4]
```

#### pointers can also use subscripts ❗

```cpp
int *p = &ia[2]; // p points to the element indexed by 2
int j = p[1]; // p[1] is equivalent to *(p + 1),
// p[1] is the same element as ia[3]
int k = p[-2]; // p[-2] is the same element as ia[0]
```

> ⚠ Unlike subscripts for vector and string, the index of the built-in subscript
operator is not an unsigned type.


#### char array -- C-style string

> ⚠ Using strcpy/strcat should be careful because it is error-prone and easy to lead to serious security leak.

 - Implementation of `strcpy` and `strcat`?  [source](https://songlee24.github.io/2015/03/15/string-operating-function/)
```cpp
/* condition: 
 * 1. the memory that des and src point to should not be overlapped
 * 2. the memory allocated for des should be large enough to contain src
 */
char* strcpy(char* des, const char* src)
{
	assert((des!=NULL) && (src!=NULL)); 
	char *address = des;  
	while((*des++ = *src++) != '\0')  
		;  
	return address;
}
```

```cpp
char* strcat(char* des, const char* src)  
{  
	assert((des!=NULL) && (src!=NULL));
	char* address = des;
	while(*des != '\0')  // move to end
		++des;
	while(*des++ = *src++)
		;
	return address;
}
```

Look at the source code in c++ library. [Source](https://zhuanlan.zhihu.com/p/45136157)

```cpp
/* Copy SRC to DEST.  */
char *
strcpy (char *dest, const char *src)
{
  char c;
  char *s = (char *) src;
  const ptrdiff_t off = dest - s - 1;

  do
    {
      c = *s++;   
      s[off] = c;  // more effective way! Only 1 pointer move per loop!
    }
  while (c != '\0');

  return dest;
}
```

### Exercise

**Exercise 3.43**: Write three different versions of a program to print the
elements of ia. One version should use a range for to manage the
iteration, the other two should use an ordinary for loop in one case using
subscripts and in the other using pointers. In all three programs write all the
types directly. That is, do not use a type alias, auto, or decltype to
simplify the code.
**Exercise 3.44**: Rewrite the programs from the previous exercises using aC++ Primer, Fifth Edition
type alias for the type of the loop control variables.
**Exercise 3.45**: Rewrite the programs again, this time using auto.

**Code for 3.43**
```cpp
#include <iostream>
using std::cout; using std::endl;

int main()
{
    int arr[3][4] = 
    { 
        { 0, 1, 2, 3 },
        { 4, 5, 6, 7 },
        { 8, 9, 10, 11 }
    };

    // range for
    for (const int(&row)[4] : arr)
        for (int col : row) cout << col << " ";
    cout << endl;

    // for loop
    for (size_t i = 0; i != 3; ++i)
        for (size_t j = 0; j != 4; ++j) cout << arr[i][j] << " ";
    cout << endl;

    // using pointers.
    for (int(*row)[4] = arr; row != arr + 3; ++row)
        for (int *col = *row; col != *row + 4; ++col) cout << *col << " ";
    cout << endl;

    return 0;
}
```
**Code for 3.44**
```cpp
#include <iostream>
using std::cout; using std::endl;

int main()
{
    int ia[3][4] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 };

    // a range for to manage the iteration
    // use type alias
    using int_array = int[4];
    for (int_array& p : ia)
        for (int q : p)
            cout << q << " ";
    cout << endl;

    // ordinary for loop using subscripts
    for (size_t i = 0; i != 3; ++i)
        for (size_t j = 0; j != 4; ++j)
            cout << ia[i][j] << " ";
    cout << endl;

    // using pointers.
    // use type alias
    for (int_array* p = ia; p != ia + 3; ++p)
        for (int *q = *p; q != *p + 4; ++q)
            cout << *q << " ";
    cout << endl;

    return 0;
}
```

**Code for 3.45**
```cpp
#include <iostream>
using std::cout; using std::endl;

int main()
{
    int ia[3][4] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 };

    // a range for to manage the iteration
    for (auto& p : ia)
        for (int q : p)
            cout << q << " ";
    cout << endl;

    // ordinary for loop using subscripts
    for (size_t i = 0; i != 3; ++i)
        for (size_t j = 0; j != 4; ++j)
            cout << ia[i][j] << " ";
    cout << endl;

    // using pointers.
    for (auto p = ia; p != ia + 3; ++p)
        for (int *q = *p; q != *p + 4; ++q)
            cout << *q << " ";
    cout << endl;

    return 0;
}
```