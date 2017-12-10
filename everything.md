[**TABLE OF CONTENTS**](toc.md) \| [Next >>>](2.md)

Problem 1: Program Input/Output
==================================================

Read Section 2.2, 4.3

Running a program from the command line
- ./program-name or path/to/program-name

Providing input: 2 ways
1> ./program-name arg1 arg2 ... argn
	- args are written into the program's memory
```wow
+---------------+
|     Code      |
+---------------+
|      n+1      |
+---------------+
|       * ------|----> ./program-name
+---------------+
|       * ------|----> arg2
+---------------+
|               |
+---------------+
|       * ------|----> argn
+---------------+
```
2> ./program-name, then type something
	- input comes through standard input stream (stdin)
	stdin -> program -> (stderr, stdout)
	- stderr never buffered, stdout maybe buffered

Redirection:
./myprogram < infile > outfile 2> errfile


Consider (C):
FILE: ../lec-code/cat

Observe: cmd line args/input from stdin are 2 different programming techniques.

To compile:
gcc -std=c00 -Wall myprogram.c -o myprogram

-Wall (show all warnings)

^D - end-of-file
^C - kill the program

This program - simplification of linux 'cat' command.
cat file1 file2 - opens them and prints one after another.

cat - echoes stdin
cat < file 	- file was used as source for stdin
		- the shell opens the file, displays

Now Can we write cat in C++?
- already valid C++.

The "C++ way"
- Command line args - same as in C.
- stdin/stdout: #include <iostream>
```c++
#include <iostream>
int main() {
	int x,y;
	std::cin >> x >> y;
	std::cout << x + y << std::endl;
}
```
- std::cin, std::cout, std::cerr - streams with types std::istream and std::ostream

- \>> is the input operator, cin >> x, populates x as a side effect, returns cin
- << is the output operator cout << x + y, prints x+y as a side effect, returns cout

File access: 
```c++
std::ifstream f{"name-of-file"};
char c;
while (f >> c) {
	std::cout << c;
}
```

f >> c returns f, which implicitly converts to a bool. true = read success, false = read failed

stream input skips whitespace, just like scanf.
to include whitespace:
```c++
std::ifstream f{"...."};
f >> std::noskipws;
char c;
```

Note: important!
- no explicit calls to fopen/fclose
- initializing f {"name of file"} opens the file.
- file is closed when scope of variable ends

Try 'cat' in C++:
```c++
#include <iostream>
#include <fstream>

using namespace std;

void echo(istream &f) {
	char c;
	f >> noskipws;
	while (f >> c) cout << c;
}

int main(int argc, char *argv[]) {
	if (argc == 1) echo(cin);
	else {
		for (int i = 1; i < argc; ++i) {
			ifstream f {argv[i]};
			echo(f);
		}
	}
}
```

cin has type istream - echo takes an istream
f has type ifstream - is echo(f) a type mismatch? no, this is fine. ifstream is a subtype of istream, so any ifstream can be treated as ifstream.

The error is: you can't write a function that takes an istream the way that echo does.
Compare:
```C
int x;
scanf("%d", &x);
```
vs
```c++
cin >> x;
```

C and C++ are pass by value languages, so scanf needs the address of x in order to change x, via pointers.
So why is it not cin >> &x?
C++ has another pointer-like type: Reference.

References:
```c++
int y = 10;
int &z = y; 	// z is called an lvalue reference to y.
		// similar to int * const z = y; but with auto dereferencing.

z = 12;		// y is now 12

int *p = &z;	// gives the addr of y.
```

In all cases, z acts as if it were y. z is an alias for y.

lvalue references must be initialized to something that has an address.

Problem 1 Continued 9/12/17 (Lecture 2)
=======================================

Why won't cat in C++ work?

## References
```c++
int y = 10;
int &z = y;
z = 12;
&z;
```


Cannot create a pointer to a reference, but a reference to a pointer is ok. e.g. int&\*x;

Cannot create a reference to a reference; e.g. int &&r = z;

Cannot create an array of references.

Can use references as function parameters.
```c++
void inc(int n) {
        ++n;
}
int x = 5;
inc(x);
cout << x; 		// returns 5
```

```c++
void inc(int &n) {	// const pointer to arg x
	++n;		// no pointer dereference
}
int x = 5;
inc(x);
cout << x; 		// returns 6
```

cin >> x works because x is passed by ref.

istream&operator >> (istream &in, int &n);

Now consider struct ReallyBig{ ... };
int f(ReallyBig rb) { ... }	// no more struct Really Big rb. Pass by value, so it copies the struct (slow)

int g(ReallyBig &rb) {...}	// reference, not copy (fast)

int h(const ReallyBig &rb) {...} // fast, no copy, h can't change rb.

Prefer pass-by-const-ref over pass-by-value for anything larger than a pointer, unless the function needs to make a copy anyway - just use pass by value.

Consider:
```c++
int f(int &n) {...}
int g(const int &n) {...}

f(5) // fails because can't initialize an lvalue ref to a literal value
g(5) // allowed, since n can never be changed. 5 stored in smoe temp location, so n has something to point to.
```

## Back to cat.

void echo(istream f) {...}
f passed by value, so istream is copied. but istream can't be copied lmao.

Works if you pass the stream by ref.

To compile:
```
g++ -std=c++14 -Wall
```

Separate Compilation
- put echo function in its own module.

Correct:
```
g++14 -c echo.cc
g++14 -c main.cc

g++14 echo.o main.o -o mycat
```
-c flag means only compile, dont link.

Advantage: only have to recompile the parts you change and relink.
Change echo.h - must recompile echo.cc and main.cc and relink.

What if you don't remember what we changed, or what depends on what?

Linux Tool: make
Create a Makefile.
Contents:
```
mycat: main.o echo.o
	g++ main.o echo.o -o mycat

main.o: main.cc echo.h
	g++ std=c++14 -Wall -g -c main.cc

echo.o: echo.cc echo.h
	g++ -std=c++14 -Wall -g -c echo.cc

.PHONY: clean

clean:
	rm main.o echo.o main
```

targets: mycat, main.o
dependencies: everything right of colon
recipes: tabbed information

How make works: list a dir in long form

-rw-r----- 	1	j629liu	j629liu	25	Sep 9 15:27	echo.cc
permissions	links	owner	group	size	last modified	name

Starting at the leaves of the dependency graph:
- if the dependency is newer than the target, rebuild the target, and so on, up until the root target.

Shortcuts - use variables:
```
CXX = g++
CXXFLAGS = 0std=c++14 -Wall
EXEC = myprogram
OBJECTS = main.o echo.o
${EXEC}: ${OBJECTS}
	${CXX} ${OBJECTS} -o ${EXEC}

main.o: main.cc echo.h
echo.o: echo.cc echo.h

.PHONY: clean

clean:
	rm ${OBJECTS} ${EXEC}
```

Omit recipes- make will guess the right one.

Writing dependencies is still hard, but the compiler can help.
g++14 -c -MMD echo.cc // generates echo.o, echo.d

```
cat echo.d:
	echo.o echo.cc echo.h

CXX = g++
CXXFLAGS = -std=c++14 -Wall -MMD
EXEC = mycat
OBJECTS = main.o echo.o
DEPENDS = ${OBJECTS:.o=.d}
${EXEC}: ${OBJECTS}
	${CXX} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}

.PHONY: clean

clean:
	rm ${EXEC} ${OBJECTS} ${DEPENDS}
```

Moral of the story: always use makefiles, and create the makefiles before coding.

<hr>

[Next >>>](2.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](1.md)   \|   [Next >>>](3.md)

Problem 2: Linear Collections and Modularity
================================================================
- Linked lists and arrays

Linked List:

Node.h:
```c++
#include <cstddef> // provides the size_t

struct Node {
	int data;
	Node *next;
};

size_t size(Node *n);
```

Node.cc:
```c++
#include "node.h"

size_t size(Node *n) {
	size_t count = 0;
	for (Node *cur = n; cur; cur = cur->next)
		++count;
	return count;
}
```

Main.cc
```c++
#include "node.h"

int main() {
	Node *n = new Node;
	n->data = 3;
	n->next = nullptr;

	Node *n2 = new Node {3, new Node {6, nullptr}};
	Node *n3 = new Node {4, new Node {5, nullptr}};

	delete n;
	delete n2->next;
	delete n2;

	while (n3) {
		Node *tmp = n3;
		n3 = n3->next;
		delete tmp;
	}


}
```

Note: DO NOT USE MALLOC/FREE in C++
Also, don't say NULL. Say nullptr.

<hr>

What happens if we do this:
```c++
#include "node.h"
#include "node.h"
```
Won't compile, because the struct is defined twice.
How do we prevent this?

### C Preprocessor:
- Transforms the program before the compiler sees it.

eg.

\#include ______ - drops the contents of a file "right here"

Including old C headers: \#include \<stdio.h\> => \#include \<cstdio\>
\#define VAR VALUE - preprocessor variable, all subsequent occurrences of VAR are replaced with VALUE, except in strings

Eg.
```c++
#define MAX 10
int x[MAX]; // Transleted into int x[10];
```

myprogram.cc
```c++
int main () {
	int x[MAX];

}
```

```
$ g++14 -DMAX=10 myprogram.cc 
```

makes a #define on the cmd line.

Conditional Compilation:
```c++
#if SECURITYLEVEL == 1
	short int
#elif SECURITYLEVEL == 2
	long long int
#endif
	publicKey;
```
Choose at most one of these to present to the compiler.

Special Case:
```c++
#if 0 // industrial strength "comment out"
...
#endif
```

Fixing the double include problem: #include guard

node.h
```c++
#ifndef NODE_H - "if NODE_H is not define"
#define NODE.H - value is the empty string
... (file contents)
#endif
```

First time node.h is included - symbol NODE_H not defined, so file included.
Subsequently, the symbol is defined, so the file is suppressed.

Always put #include guards in header files.
Never compile .h files
Never include .cc files

What if someone writes:
```c++
struct Node {
	int data;
	Node * left, *right;
};

size_t size(Node *n);
```
Can't use both tree and list size functions in the same program.

### NAMESPACES
Example Usage:

list.h:
```c++
namespace List {
	struct Node {
		int data;
		Node *next;
	};

	size_t size(Node *n);
}
```

tree.h:
```c++
namespace Tree {
	struct Node {
		int data;
		Node *right, *left;
	};

	size_t size(Node *n);
}
```

list.cc:
```c++
#include "list.h"

size_t List::size (Node *n) {
	...
}
```

tree.cc
```c++
#include "tree.h"

namespace Tree {
	size.t size(Node *n) {
		...
	}
}
```

main.cc
```c++
#include "list.h"
#include "tree.h"

int main() {
	List::Node *ln = new List::Node{1, nullptr};
	Tree::Node *tn = new Tree::Node{2, nullptr, nullptr};

	delete ln;
	delete tn;
}
```

Namespaces are open:
- anyone can add items to any namespace

some_other_file.h:
```c++
namespace List {
	int some_other_function();
	struct some_other_struct{...};
};
```

However: you are not allowed to add members to namespace std.
That is undefined behavior.

<hr>

[<<< Previous](1.md)   |   [Next >>>](3.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](2.md)   \|   [Next >>>](4.md)


Problem 3: Linear Collections and Memory Management
===========================================================

Arrays on the stack, fixed size:
```c++
int a[10]
```
- a on the stack, fixed size.

On the heap:
```c++
int *p = new int[10];
```
- p is a pointer on the stack pointing at the heap.

To delete: delete []p;
Use new with delete. new[...] with delete[]

Mismatching these is undefined behavior.

Problem: What if our array isn't big enough:?
Note: no realloc for new/delete arrays

Use abstraction to solve the problem.

vector.h:
```c++
#ifndef VECTOR_H
#define VECTOR_H
namespace CS246E {

struct vector {
	size_t size, cap;
	int *theVector;
};

const size_t startSize = 1;

vector make_vector();
size_t size(const vector &v);
int &itemAt(const vector &v, size_t i);
void push_back(const vector &v, int x);
void pop_back(const vector &v, int x);
void dispose(vector &v);

}

#endif
```

# Lecture 4, 9/19/17

vector.cc:
```c++
#include "vector.h"

namespace {
void increaseCap(CS246E::Vector &v) {
	if (v.size == v.cap) {
		int *newVec = new int[2*v.cap];
		for (size_t i = 0; i < v.cap; ++i) newVec[i] = v.theVector[i];
		delete [] v.theVector;
		v.theVector = newVec;
		v.cap *= 2;
	}
}
}

CS246E::Vector CS246E::make_vector() {
	Vector v {0, startSize, new int[startSize]};
	return v;
}

size_t CS246E::size(const Vector &v) { return v.size; }

int &CS246E::itemAt(const Vector &v, size_t i) { return v.theVector[i]; }

void CS246E::push_back(Vector &v, int n) {
	increaseCap(v);
	v.theVector[v.size++] = n;
}

void CS246E::pop_back(Vector &v, int n) {
	if (v.size > 0) --v.size;
}

void CS246E::dispose(Vector &v) {
	delete v.theVector;
}
```

### Reading: Section 7.7.1, Chapter 14, 16.2

main.cc:
```c++
#include "vector.h"
using CS246E::vector;

int main () {
	vector v = CS246E::make_vector();
	push_back(v,1);
	push_back(v,10);
	push_back(v,100);
	itemAt(v,0) = 2;

	dispose(v);
}
```

Question: Why don't we have to say CS246E::push_back, CS246E::itemAt, CS246E::dispose?
Answer: Argument Dependent Lookup (ADL) - also called könig lookup.
- If the type of a function f's argument belongs to a namespace n, then C++ will search the namespace n, as well as the current scope, for a function matching f.
- This is the reason why we can say "std::cout << x" instead of "std::operator <<(std::cout, x)"

Problems:
What if we forget to call make_vector?
- We have an unitialized object.

What if we forget to call dispose?
- Memory leak.

#### How can we make this more robust?

# Introduction to Classes
First concept in OOP - functions inside of structs.

```c++
struct Student {
	int assns, mt, final;
	float grade() { return assns*0.4 + mt*0.2 + final*0.4 }
};

```

Structs that can contain functions are what we call classes.
Functions inside structs are called <b>methods</b>.
Instances of a class are called <b>objects</b>.

```c++
Student bob {90, 70, 80};

cout << bob.grade();
```

What do assins, mt, final mean within grade() { ... } ?
- fields of the current object- the receiver of the method call, e.g. bob

Formally: methods differ from functions in that methods have an implicit param called <u>this</u> that is a pointer to the receiver object.

```c++
bob.grade() // this == &bob
```

Could have written (equivalent):
```c++
struct Student {
	float grade() {
		return 	this->assns * 0.4
			+	this->mt * 0.2
			+	this->final * 0.4;
	}
};
```

### Initializing Objects
```c++
Student bob {90,70,80}; // C-style struct initialization, field by field, ok but limited

// Better: initialization method - a constructor.
struct Student {
	int assns, mt, final;

	Student(int assns, int mt, int final) {
		this->assns = assns; // using this->assns bc they have the same name, but if different u dont need arrows
		this->mt = mt;
		this->final = final;
	}

};

// Now can use:
Student bob {90,70,80}; // Now calls the constructor function with args 90,70,80.
```

Note: once a constructor function is defined, C-style field by field initialization is no longer available.

Equiv:
```c++
Student bob = Student {90,70,80};

// Heap:
Student *p = new Student {90,70,80};
delete p;
```

Advantages of constructors: default parameters, overloading, sanity checks
e.g.

Default Values:
```c++
struct Student {
	student(int assns = 0, int mt = 0, int final = 0) {
		this->assns = assns;
	}
};

Student laura {70}; // 70, 0, 0
Student newKid; 	// 0, 0, 0
```

Note:
Every class comes with a built in default constructor. (i.e. zero argument constructor)

e.g.
```c++
Node n; // calls default constructor. default-constructs all fields that are objects. 
		// (does nothing in this case)
```
- Goes away if you write any constructor at all.

e.g.
```c++
struct Node {
	int data;
	Node *next;
	Node (int data, Node *next = nullptr) {

	}
};

Node n{3};	// works
Node n; 	// can't do this, no default constructor.
```

Object creation protocol
- When an object is created, there are 4 steps:
	1. Space is allocated
	2. (later)
	3. Fields constructed in declaration order (field constructors called for fields that are objects)
	4. Constructor body runs.

Field initialization should happen in step 3, but constructor body is step 4.

Consequence: object fields initialized twice.
```c++
include <string>

struct Student {
	int assns, mt, final;
	std::string name;

	Student(std::string name, int assns, int mt, int final) {
		this->name = name;
		...
	}
};

Student mike {"Mike", 90, 70, 80};
// name default initialized in step 3 ("")
// reassigned in step 4 ("Mike")
```

To fix: the Member Initialization List (MIL)
```c++
struct Student {
	...
	Student (std::string name, int assns, int mt, int final):	// field{param}
		name{name}, assns{assns}, mt{mt}, final{final}			// step 3
		{}														// step 4
}
```

# Tutorial 2, 9/20/17

Note
- MIL must be used for fields that are constants, references, objects without default constructor
- Should be used as much as possible

Careful: single argument constructors:
```c++
struct Node {
	Node(int data, Node *next = nullptr) : {}
}

// single argument constructors create implicit conversions.

Node n{4}; 	// OK
Node n = 4; // Also OK

void f(Node n);
f(4); 		// OK - may be trouble

// implicit conversion from int to Node
```
Explicit:
```c++
// To fix, add word explicit to constructor to disable implicit conversion

struct Node {
	explicit Node(int data, Node *next = nullptr): {

	}
}

Node n{4};	// OK
Node n = 4;	// NO
f(4);		// NO
f(Node{4});	// OK
```

### Object Destruction:
- A method called the destructor runs automatically
- Built in destructor: calls destructor on all fields that are objects


Object Destruction Protocol:
1. Destructor body runs
2. Fields destructed (destructor called on fields that are objects) in reverse declaration order
3. (later)
4. Space is deallocated.

```c++
struct Node {
	int data;
	Node *next;
};

// what does the built in destructor do?
// Nothing - neither field is an object.

Node *n = new Node{3, new Node{4, new Node{5, nullptr}}};
delete n; // only deletes the first node

```

Writing our own destructor:
```c++
struct Node {
	~Node() {
		delete next;
	}
}

delete n; // now frees the whole list
```

Also:
```c++
{
	Node n {1, new Node{2, new Node{3, nullptr}}};
	// n is stack allocated, other nodes are heap allocated.
}	// Scope of n ends; whole list is freed.
```

### Objects:
- a constructor always runs when they are created
- a destructor always runs when they are destroyed.

### Back to the vector example - lets turn it into a class.
vector.h:
```c++
#ifndef VECTOR_H
#define VECTOR_H

namespace CS246E {
struct Vector {
	size.t n, cap;
	int *theVector;
	Vector();
	size_t size();
	int &itemAt(size_t i);
	void push_back(int n);
	void pop_back();
	~vector();
};
}
#endif
```

vector.cc:
```c++
#include "vector.h"

namespace {
void increaseCap(Vector &v) {

}
}
const size_t startSize = 1;
CS246E::Vector::Vector() : size{0}, cap{startSize}, theVector{new int[cap]} {}
size_t CS246E::Vector::size() {return n;}

// etc...

CS246E::Vector::~Vector() {
	delete [] theVector;
}

```

main.cc:
```c++
int main() {
	Vector v; // constructor automatically called - no make_vector
	v.push_back(1);
	v.push_back(10);
	v.push_back(100);
	v.itemAt(0) = 2;

}	// no dispose, destructor cleans v up.
```

<hr>

[<<< Previous](2.md)   |   [Next >>>](4.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](3.md)   \|   [Next >>>](5.md)

Problem 4: Copies
=================

```c++
vector v;
v.push_back(100);

vector w = v;	// Allowed - constructs w as a copy of v.
w.itemAt(0);	// 100
v.itemAt(0) = 200;
w.itemAt(0);	// 200;
```

What we have here is a <u>Shallow Copy</u>.
- v and w are sharing data.
- vector w = v - constructs w as a copy of v.
	- invokes the <u>copy constructor</u>.

```c++
struct Vector {
	vector (const Vector &other) { ... } // copy constructor
};
```

If you want a deep copy - write your own copy constructor.
```c++
struct Node {
	int data;
	Node *next;
	...
	Node(const Node &other): data{other.data}, 
		next{ other.next? new Node{*other.next} : nullptr } { }
}
```

# Lecture 5 (9/21/17)

Now consider this:
```c++
vector v;
vector w;
w = v;	// this is a copy but not a construction

// this uses copy assignment operator
// compiler-supplied: copies each field (shallow)
// w's old fields are leaked
```

Deep copy assignment:
```c++
struct Node {
	int data;
	Node *next;
	...
	Node &operator=(const Node &other) {
		data = other.data;
		delete next;

		next = other.next ? new Node{*other.next} : nullptr;
		return *this;

	}
}
```
But this is wrong - dangerous. 

Consider: 
```c++
Node n { ... };
n=n; // goodbye world;
```
This destroys n's data and then copies it.

Must always make sure that operator = works in the case of self-assignment.
```c++
Node &Node::operator=(const Node &other) {
	if (this == &other) return *this
	// else is the same as before
}
```

### Alternative: copy and swap idiom

```c++
#include <utility>
Struct Node {

	void swap(Node &other) {
		using std::swap;
		swap(data, other.data);
		swap(next, other.next);
	}

	Node &operator=(const Node &other) {
		Node tmp = other; // uses copy constructor
		swap(tmp);
		return *this;
	}
};

```

<hr>

[<<< Previous](3.md)   |   [Next >>>](5.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](4.md)   \|   [Next >>>](6.md)

Problem 5: Moves
================

Consider:
```c++
Node plusOne(Node n) {
	for (Node *p = &n; p; p = p->next) {
		++p->data;
	}
	return n;
}

Node n {1, new Node{2, nullptr}};

Node m = plusOne(n); // call to copy constructor
// but what is "other" here? reference to what?

Node m;
m = plusOne(n); // call to copy assignment
```
- Other is a reference to a Temporary Object: Created to hold the result of plusOne(n)
- Copy constructor deep copies the data from this temporary.

But: the temporary is just going to be thrown out anyways, as soon as the statement Node m = plusOne(n) is done.
- wasteful to deep copy the temp, why not just steal the data instead?
- Need to be able to tell whether other is a reference to a temporary object, or a standalone object.

## Rvalue References
Node && is a reference to a temporary object (rvalue) of type Node.

Version of the constructor that takes a Node &&.
```c++
struct Node {
	...
	// below is called a move constructor - steals other's data
	Node (Node &&other): data{other.data}, next{other.next} {
		other.next = nullptr;
	}

	// move assignment operator - need to: steal other's data, delete my old data
	Node &operator=(Node &&other) {
		using std::swap;			// lets just swap with other
		swap(data, other.data);
		swap(next, other.next);
		return *this;

	}

};
```

Can combine copy/move assignment:
```
struct Node {
	...
	// below is a unified assignment operator
	// pass by value - invokes copy constructor if other is an lvalue,
	// invokes move constructor if other is an rvalue
	Node *operator=(Node other) {
		swap(other);
		return *this;
	}
};
```
Note: copy/swap can be more expensive than necessary, so hand coded operators may do less copying.

But now consider:
```c++
struct Student {
	std::string name;
	Student(const string &name):name {name} {}
	// what if name poitns to an rvalue?
};
```

```c++
struct Student {
	std::string name;
	Student(std::string name): name{name} {}
	// copies if name is an lvalue, moves if name is an rvalue
}
```

```c++
struct Student {
	std::string name;
	// name is an lvalue in this case
	Student(std::string name): name{std::move(name)} {}
	// std::move() forces name to be treated as an rvalue
	// now uses string's move constructor.
};
```

Move constructor:
```c++
Struct Student {
	...
	// other points at an rvalue, but is an lvalue.
	Student(Student &&other): name{std::move(other.name)} {}

	Student &operator=(Student other) {
		name = std::move(other.name);
	}
};
```

If you don't define move operations, copy ones will be used.
If you do define move operations, they replace copy operations whenever the argument is an rvalue (temporary).

# Lecture 8, 9/26/17
Recall: copies, moves, std::move

```c++
vector makeAVector() {
	return vector{};		// basic constructor
};

vector v = makeAVector();	// move constructor? copy constructor?
```

This is known as <u>Copy/Move Elision</u>

Try this in g++ - just the basic constructor called, no copy/move.

In some circumstances, the compiler is allowed to skip calling copy/move constructors (but doesn't have to)

makeAVector writes result directly into the space occupied by v, rather than copy/move it later.

```c++
vector v = vector{}; // Formally a basic construction and a copy/move construction.

// Here though, the compiler is required to elide the copy/move, so basic constructor only.

vector v = vector{vector{vector{}}};

// this is still one basic constructor only.

void doSomething(vector v) { ... } // pass by value (copy/move constructor)

doSomething(makeAVector());
// result of makeAVector written directly into the param, so no copy/move.
// this is allowed, even if dropping constructor calls would change the behavior of the program
```

If you need all constructors to run:
```
g++14 -fno_elide_constructors
```
- can slow down program considerably

### In summary: Rule of 5 (Big 5)
- If you need to customize any one of 
	1. Copy constructor
	2. Copy assignment
	3. Destructor
	4. Move constructor
	5. Move assignment
	then you usually need to customize all 5.

<hr>

[<<< Previous](4.md)   |   [Next >>>](6.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](5.md)   \|   [Next >>>](7.md)

Problem 6: I want a constant vector
===================================

Say we want to print a vector:
```c++
ostream &operator<<(ostream &out, const vector &v) {
	for (size_t i = 0; i < v.size(); ++i) 
		out << v.itemAt[i] << " ";
	return out;
}

int main() {
	vector v;
	...
	cout << v;
}
```
- This wont compile.
	- can't call size and itemAt on a const object.
	- what if these methods change fields?
	- since they don't, it should be safe to use them, but the compiler doesn't know that

You must declare methods to be const.

vector.h:
```c++
struct vector {
	...

	size_t size() const;
	int &itemAt(size_t i) const; 
	// Means these methods will not modify fields
	// Can be called on const objects
};
```

vector.cc:
```c++

size_t vector::size() const { return n; }
int &vector::itemAt(size_t i) const { return theVector[i]; }
```

Now the loop will work.
But:

```c++
void f(const vector &v) {
	v.itemAt(0) = 4; // Works, but v is not very const.
}
```
- v is a const object, cannot change n, cap, theVector (pointer)
	- but we can change items pointed to by theVector.

Can we fix this?
```c++
struct vector {
	...
	const int &itemAt(size_t i) const;
};

const int vector::&itemAt(size_t i) const { return theVector[i]; }
```

Now v.itemAt(0) = 4 won't compile if v is const. But it also won't compile if v is not const.

To fix this: <u>const overloading</u>
```c++
struct vector {
	...
	const int &itemAt(size_t i) const;	// will be called if object is const
	int &itemAt(size_t i);				// will be called if object not const
};

inline const int vector::&itemAt(size_t i) const {return theVector[i];}
inline int vector::&itemAt(size_t i) {return theVector[i];}

// suggests replacing the func call with func body - save cost of func call.
// do this in the header.
```
v.itemAt(0) = 4 will compile if and only if v is non-const.

Now lets make it prettier:

vector.h:
```c++
struct vector {
	...
	size_t size() const { return n; }
	const int &operator[](size_t i) const { return theVector[i]; }
	int &operator[](size_t i) { return theVector[i]; }
	// if you put the method body in the class, implicitly declares method is inline.
};
```

Back to the start:
```c++
ostream &operator<<(ostream &out, vector v) {
	for (size_t i = 0; i < v.size(); ++i) {
		out << v[i] << " ";
	}
	return out;
}
```


<hr>

[<<< Previous](5.md)   |   [Next >>>](7.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](6.md)   \|   [Next >>>](8.md)

Problem 7: Tampering
====================

```c++
vector v;
v.cap = 100;		// sets cap without allocating memory
v.push_back(1);
v.push_back(10);
...

// Undefinied behavior, will likely crash.
```

Interfereing with ADTs
1. Forgery: create an object without using a constructor (impossible now)
2. Tampering: accessing internals without using provided interface methods

What's the big deal? Invariants.

<u>Invariants</u> are statements that will always be true about abstractions.

ADT's provide and rely on invariants.
- stacks: provide the invariant that the last item pushed is the first popped off.
- vectors: rely on the invariant that elements 0,...,cap-1 denote valid locations.

Cannot guarantee invariants if the user can interfere.
This makes the program hard to reason about.

Fix: <u>encapsulation</u> - seal objects into "black boxes"
```c++
struct vector {
	private:
		size_t n, cap;
		int *theVector;
		// fields are only accessible within the vector class
	public:
		vector();
		size_t size() const;
		int push_back(int n);
		// visible to all
};
```
- If no access specifier is given, then that means public.


vector.cc:
```
#include <vector.h>

namespace {
	void increaseCap(vector &v) {

	}
}
```
- increaseCap doesn't work anymore. Doesn't have access to cap and theVector.

### Try again:
vector.h:
```c++
struct vector {
	private:
		...
	public:
		...
	private:
		void increaseCap(); // Now a private method.
};
```

vector.cc:
```c++
#include <vector.h>

namespace CS246E {
vector::vector() { ... }
// rest as before

void vector::increaseCap() {
	...
}
}
```
- no more anonymous namespace



- structs - default public access
- classes - default private access

classes and structs in c++ are the same, but classes are private by default!

vector.h:
```c++
class vector {
	size_t n, cap;
	int *theVector;

	public:
		vector();
		...

	private:
		void increaseCap();
};
```

vector.cc: no change.

# Lecture 8, 9/28/17

A problem with linked lists:

```c++
Node n {3, nullptr};
Node m {4, &n}; 
// when m's destructor runs, it will try to delete &n, which is on the stack
```
- There was an invariant - next is nullptr, or was allocated by new

How can we enforce this? Encapsulate node inside a wrapper class.

```c++
class list {
	struct Node {	// private nested class, not available outside of list
		int data;
		Node *next; 

		// ... methods
	}

	Node *theList;
	size_t size;
	public:
		list(): theList{nullptr} {}
		~list() { delete theList; }
		size_t size() const;
		void push_front(int n) {
			theList = new Node{n, theList};
		}
		void pop_front() {
			if (theList) {
				node *temp = theList;
				theList = theList->next;
				temp->next = nullptr;
				delete temp;
			}
		}
		const int &operator[](size_t i) const {
			Node *cur = theList;
			for (size_t j = 0; j<i && cur, ++j, cur = cur->next);
			return cur->data;
		}
		int &operator[](size_t i) { ... }
};
```
- Client cannot manipulate the list directly.
- no access to next pointers, invariant is maintained

<hr>

[<<< Previous](6.md)   |   [Next >>>](8.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](7.md)   \|   [Next >>>](9.md)

Problem 8: Efficient Iteration
===============================

```c++
vector v;
v.push_back();
...

list l;
l.push_front();
...

for (size_t i=0; i<v.size(); ++i) {
	cout << v[i] << endl;
}
// Array access is efficient, O(n) traversal

for (size_t i=0; i<l.size(); ++i) {
	cout << l[i] << endl;
}
// l[i] is O(n),
// O(n^2) traversal

```

No direct access to the "next" pointers - how can we do efficient iteration?

## Design Patterns
- well known solutions to well studied problems
- adapted to suit needs

### Iterator Pattern
- efficient iteration over a collection, without exposing the underlying structure

Idea: create a class that "remembers" where you are in the list
- abstraction of a pointer

Inspiration: C
```c
for (int *p = arr; p != a + size; ++p) {
	print("%d\n", *p);
}
```

List with iterator:
```c++
class list {
	struct Node {...};
	Node *theList;

	public:
		...
		class iterator {
			Node *p;
			public:
				iterator(Node *p): p{p} {}
				bool operator!=(const iterator &other) const {return p != other.p; }
				int &operator*() { p->data; }
				iterator &operator++() { p = p->next; return *this; }
		}
		iterator begin() {
			return iterator{theList};
		}
		iterator end() {
			return iterator{nullptr};
		}
};
```

List with iterator Usage:
```c++
list l;
for (list::iterator it=l.begin(); it!=l.end(); ++it) {
	cout << *it << '\n';
}
```
// O(n)

### Question: Should list::begin, list::end be const methods?
Consider:
```c++
ostream &operator<<(ostream &out, const list &l) {
	for (list::iterator it=l.begin(); it!=l.end(); ++it) {
		cout << *it << " ";
	}
	return out;
}
// wont compile if begin/end are not const, bc list param is const.
```

If begin/end are const:
```c++
ostream &operator<<(ostream &out, const list &l) {
	for (list::iterator it=l.begin(); it!=l.end(); ++it) {
		cout << *it << " ";
		++*it;
	}
	return out;
}
// will compile, even though its incrementing items in the list
// shouldn't compile, the list is supposed to be const.
// compiles because operator * returns a non-const ref
```

Conclusion: iteration over const is different from iteration over non-const.
- Thus, we should make a second iterator class.
```c++
class list {
	...
	public:
		class iterator {
			...
			public:
				int &operator*() const {...}
		};
		class const_iterator{
			Node *p;
		public:
			const_iterator(Node *p): p{p} {}
			bool operator!=(const const_iterator &other) {return p!=other.p;}
			const_terator &operator++() {p=p->next; return *this;}
			const int&operator*() const { return p->data; } // only difference is return type is const
		};
		iterator begin() { return iterator{theList};}
		iterator end() { return iterator{nullptr};}
		const_iterator begin() const { return const_iterator{theList};}
		const_iterator end() const { return const_iterator{nullptr};}

		// const overloading
};
```

Usage:
```c++
ostream &operator<<(ostream &out, const list &l) {
	for (list::const_iterator it=l.begin(); it != l.end(); ++it) {
		out << *it << " ";
	}
	return out;
}
```

list::const_iterator it = l.begin() is a mouthful.

Shorter:
```c++
ostream &operator<<(ostream &out, const list &l) {
	for (auto it = l.begin(); it != l.end(); ++it) {
		out << *it << " ";
	}
	return out;
}
```

Auto:
```c++
auto x = expr; 
// saves writing down x's type
// x will be given expr's type
```

Even Shorter:
```c++
osteram *operator<<(ostream &out, const list &l) {
	for (auto n:l) {		// range based for loop
		out < n << "\n";
	}
	return out;
}
```
- Range based for loops are available for any class with:
	- methods (or functions) begin and end that return an iterator object
	- the iterator class must support unary \*, prefix ++, and !=

Note:
```c++
for (auto n:l) ++n; 
```
- n is declared by value, ++n increments n, not the list items


```c++
for (auto &n:l) ++n;
```
- n is a reference, this will update the list items


```c++
for (const auto &n:l) ...;
```
- const ref, cannot be mutated


One small encapsulation problem:
The client can write this:
```c++
list::iterator it {nullptr};
```
- forgery - create an end iterator without calling end();
One fix: make iterator constructors private.
But then: list can't create iterators either.

Solution: <u>friendship</u>
```c++
class list {
	...
	public:
		class iterator {
			...
			iterator(Node *p) {}
		public:
			friend class list;		// list has access to all of iterator's implementation
		};
		class const_iterator {
			...
			const_iterator(Node *p) {}
		public:
			friend class list;
		};
}
```
- Limit friendships because they weaken encapsulation.

### Lecture 9 10/3/17

Recall: Encapsulation and Iterators for linked list

Can do the same for vectors:
```c++
class vector {
	size_t n, cap;
	int *theVector;
public:
	class iterator {
		int *p;
		...
	};
	class const_iterator {
		...
	}

	iterator begin() { return iterator{theVector}; }
	iterator end() { return iterator{theVector + n}; }
	const_iterator begin() const { return const_iterator{theVector}; }
	const_iterator end() const { return const_iterator{theVector + n}; }
}
```
OR
```c++
typedef int *iterator;
typedef const int *const_iterator;

using iterator = int*;
using const.iterator = const int*;
iterator begin() {return theVector;}
iterator end() {return theVector+n;}
```

<hr>

[<<< Previous](7.md)   |   [Next >>>](9.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](8.md)   \|   [Next >>>](10.md)

Problem 9: Staying in bounds
=============================
```c++
vector v;
v.push_back(4);
v.push_back(5);
v[2];	// out of bounds, may or may not crash
```

Can we make this safer?
- Adding bounds checks to operator[] may be needlessly expensive.
- Could have a second, safer fetch operator

```c++
class vector {
	...
public:
	int &at(size_t i) {
		if (i < n) return theVector[i];
		// how do we show that v.at(2) is wrong
		// return any int - looks like non-error
		// Returning a non-int - not type correct
		// Crash the program - can we do better?
	}
};
```

### Solution: Raise an exception
```c++
class range_error {};

class vector {
	...
public:
	int &at(size_t i) {
		if (i < n) return theVector[i];
		else throw range_error{};
	}
};
```
Client's options:
1. do nothing
```c++
vector v;
v.push_back(1);
v.at(1); // the exception will crash the program
```
2. catch the exception
```c++
try {
	vector v;
	v.push_back(1);
	v.at(1);
}
catch (rangeError &r) {
	// do something
}
```
3. let your caller catch it
```c++
int f() {
	vector v;
	v.push_back(1);
	v.at(1);
}
int g() {
	try{f();}
	catch (range_error &r) { /* do something */ }
}
```

Exception will <u>propagate</u> back through the call chain until a <u>handler</u> is found.
- This is called <u>unwinding</u> the stack.
- If no handler is found, the program aborts (std::terminate gets called)
- Control resumes after the catch block. (problem code is not retried)

### What happens if a constructor throws an exception?
- object is considered partial constructed
- destructor will not run on partially constructed objects.

E.g.
```c++
class C { ... };
class D {
	C a;
	C b;
	int *c;
public:
	D() {
		c = new int[100];
		...
		if (...) throw something {}; // *
	}
	~D() { delete [] c; }
	}
};

D d;
// at *, the D object is not fully constructed, so ~D() will not run on d.
// however, a and b are fully constructed, so their destructors will run
```

So if a constructor wants to throw, it must clean up after itself.
```c++
class D {
	...
	public:
	D() {
		c = new int [100];
		...
		if (...) {
			delete [] c; // delete the array before throwing
			throw something{};
		}
	}
};
```

### What happens if a destructor throws?
```c++
class myexn{};
class C {
	...
	public:
	~C() noexcept(false) {throw myexn{};}
};

// by default, program aborts immediately
// std::terminate is called
// we must tag it with noexcept(false)
// watch out:

void h() {
	C c1;	// destructor for c1 throws at end of h
}

void g() {
	C c2;	// unwind through g
	h();	// destructor for c2 throws at end of g
};			// there are now 2 unhandled exceptions

void f() {
	try {
		g();
	}
	catch (myexn &ex) {

	}

}
```
As soon as we get more than 1 unhandled exception, termination is guaranteed, std::terminate called.

**Basically, don't let your destructor throw.**

Also note - you can throw any value, not just objects
```c++
void fact(int n) {
	if (n == 1) throw 1;
	try {
		fact(n-1);
	}
	catch (int m) {
		throw(n*m);
	}
}

void main() {
	int n;
	try {
		fact(n);
	}
	catch (int m) {
		cout << m << endl;
	}
}
```

<hr>

[<<< Previous](8.md)   |   [Next >>>](10.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](9.md)   \|   [Next >>>](11.md)

Problem 10: I wanted a vector of chars :(
======================================
Start over? No.

Introduce a major abstraction mechanism - templates
- generalize overtypes

vector.h
```c++
namespace CS246E {

template <typename T> class vector {
	size_t n, cap;
	T *theVector;

public:
	vector();
	...
	void push_back(T n);
	T &operator[](size_t i);
	const T &operator[] const (size_t i);
	using iterator = T*;
	using const_iterator = T*;
};
}

// implementation:
template <typename T> vector<T>::vector() n{0}, cap{1}, theVector{new T[cap]} {}
template <typename T> void push_back(T n) { ... }
// etc.
```
Note: must put implementation in .h file.

main.cc
```c++
int main() {
	vector<int> v; 	// vector of ints
	v.push_back(1);
	...
	vector<char> w; // vector of char
	v.push_back('a');

}
```

### Semantics:
- The first time the compiler encounters `vector<int>` it creates a version of the vector code where int replaces T, and compiles that new class.
- Can't do that unless it has all the details about the class
- So implementation must be available in .h
- Can also write method bodies in line.

<hr>

[<<< Previous](9.md)   |   [Next >>>](11.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](10.md)   \|   [Next >>>](12.md)


Problem 11: Better Initialization
=================================
Arrays: `int a[] = {1,2,3,4,5}`
Vector: 
```c++
vector<int> v;
v.push_back(1);
...
```
Long sequence of push_backs can be clunky

Goal: Better initialization
```c++
template <typename T> class vector {
	...
	public:
	vector(): ...
	vector(size_t n, T i = T {}): n{n}, cap{n == 0 ? 1 : n}, theVector{new T[cap]} {
		for (size_t j = 0; j < n; ++j) {
			theVector[j] = i;
		}
	}
}
```

Now:
```c++
vector <int> v;		// empty
vector <int> w {5}; // 0 0 0 0 0
vector <int> z {3, 4};	// 4 4 4
```

Note: T{} (default constructor) means 0 if T is a built-in type

**True Array-Style Initialization**
```c++
#include <initializer_list>
template <typename T> class Vector {
	...
	public:

	vector() ... { ... }
	vector(size_t n, T i = T{}) ... { ... }
	vector(std::initializer_list<T> init): n {init.size()}, 
		size_t cap{init.size()}, theVector{new T[cap]} {
		size_t i = 0;
		for (auto &t: init) theVector[i++] = t;
	}
};

vector <int> v {1,2,3,4,5}; // 1 2 3 4 5
vector <int> v;				// empty
vector <int> v {5};			// 5
vector <int> v {3,4};		// 3 4

// the other constructor is not being called.
```
Default constructors take precedence over initializer lists, which take precedence over other constructors.

To get the other constructor to run: use round bracket initialization
```c++
vector <int> v (5);
vector <int> v (3,4);
```

A note on cost: item in a init list are stored in contiguous memory (begin method returns a pointer). So we are using one array to build another, 2 copies in memory.
- Note: initializer_lists are meant to be immutable
- Do not try to modify their contents, and do not use them as standalone data structures

But also note: only one allocation in vector, not several:
- no doubling and reallocating, as there was with a sequence of pushbacks

In general, if you know how big your vector will be, you can save reallocation cost by requesting space upfront.

```c++
template <typename T> class vector {
	...
	public:
	...
	void reserve(size_t newCap) {
		if (cap < newCap) {
			T *newVec = new T[newCap];
			for (size_t i = 0; i < n; ++i) newVec[i] = theVector[i];
			delete [] theVector;
			theVector = newVec;
			cap = newCap;
		}
	}
}
```
Exercise: rewrite vector so that psuh_back uses reserve instead of increaseCap

```c++
vector <int> v;
v.reserve(10);
v.push_back(...);
...
// can do 10 push_backs without needing to reallocate
```

<hr>

[<<< Previous](10.md)   \|   [Next >>>](12.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](11.md)   \|   [Next >>>](13.md)


Problem 12: I want a vector of Posns
=====================================
```c++
struct Posn {
	int x,y;
	Posn(int x, int y): x{x}, y{y} {}
}

int main() {
	vector <Posn> v;
	...
}
```
This will not compile. Why not?

Take a look at vector's constructor:
```c++
template <typename T> vector<T>::vector(): n{0}, cap{1}, theVector{new T[cap]} {}

// T[cap] creates an array of T objects
// Which T objects will be stored in the array?
// C++ always calls a constructor when creating an object
// Which constructor gets called? Default constructor T{}
// Posn doesn't have one.
```

Need to separate memory allocation (step 1) from object initialization (step 2-4)

**Allocation:** `void *operator new(size_t)`
- allocates size_t bytes
- no initialization
- returns void *

Note: 
- C: void* implicitly converts to any pointer type
- C++: the conversion requires a cast

Initialization: "placement new"
- `new (addr) type`
- constructs a 'type' object at 'addr'
- does not allocate memory

```c++
template <typename T> class vector {
	...
	public:
	vector(): n{0}, cap{1}, 
		theVector{ static_cast <T*> (operator new(sizeof(T)))} //CAST
	{}
	vector(size_t n, T x = T{}): n{n}, cap{n},
		theVector{static_cast <T*> (operator new(n * sizeof(T)))} {
		for (size_t i = 0; i < n; ++i) {
			new (theVector + i) T {x};
		}
	}

	void push_back(T x) {
		increaseCap();
		new (theVector + (n++)) T (x);
	}

	void pop_back() {
		if (n) {
			theVector[n-1].~T();
			--n;
		}
	}

	~vector() {
		destroy_items();
		operator delete(theVector);
	}

	void destroy_items() {
		for (auto &x:*this)
			x.~T();
	}
}
```
<hr>

[<<< Previous](11.md)   \|   [Next >>>](13.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](12.md)   \|   [Next >>>](14.md)

Problem 13: Less Copying!
=========================

Before: `void push_back(int n);`

Now:
```c++
void push_back(T x) { // (1)
	increaseCap();
	new (theVector + (n++)) T (x); // (2)
}
```
If the arg is an lvalue:
- (1) is a copy constructor
- (2) is also a copy constructor

If the arg is an rvalue:
- (1) is a move constructor
- (2) is a copy constructor

The Fix:
```c++
void push_back(T x) {
	increaseCap();
	new (theVector + (n++)) T (std::move(x));
}
```

Now:
lvalue: copy + move
rvalue: move + move

**But what if T doesn't have a move constructor? 2 Copies**

Better: take T by reference:
```c++
void push_back(const T&x) {
	increaseCap();
	new(theVector+(n++)) T (x);
}

void push_back(T &&x) {
	increaseCap;
	new(theVector+(n++)) T (std::move(x));
}
```
lvalues: 1 copy, rvalues: 1 move
if no move constructor: 1 copy

Now consider:
```c++
vector <Posn> v;
v.push_back(Posn{3,4});
```
1. constructor call to create the posn object
2. copy or move construction into the vector(depending on whether Posn has move constructors)
3. destructor call on the temp object

Could eliminate 1 and 3 if we got the vector to make the object instead of the client
- pass args to the vector, not the actual object.
- How? Soon... But first....

A note on template functions:
Consider: std::swap seems to work on all types

Implementation:
```c++
template <typename T> void swap (T&a, T&b) {
	T tmp(std::move(a));
	a = std::move(b);
	b = std::move(tmp);
}

int x = 1, y = 2;
swap(x,y); // equiv. swap<int>(x,y);
// Don't have to say swap<int>
// C++ can deduce thsi from the types of x and y.
// In general, only have to say f<T>(...) if T cannot be deduced from the args.
```

Type deduction for template args follows the same rules as type deduction for auto.

<hr>

Back to vector - passing constructor args
- we don't know what types constructor args should have.
- T could be any class, could ahve several constructors

Idea - member template function - like swap, it could take anything
- Don't know how many constructor arguments there are

Solution: __variadic templates__
- similar to Racket macros.

```c++
template <typename T> class Vector {
	...
	public:
	...
	// these dots in the following function must be here.
	template <typename... Args> void emplace_back(Args... args) {
		increaseCap();
		new(theVector+(n++)) T (args...);
	}

};
```
- Args is a sequence of type vars denoting the types of the actual args of <i>emplace_back</i>.
- args is sequence of program vars denoting the actual arguments of <i>emplace_back</i>.

Now we can do this:
```c++
vector <Posn> v;
v.emplace_back(3,4);
```

The Problem: args is being taken by value. Can we take args by reference?
- lvalue or rvalue ref? Mixture?

```c++
{
...
	template <typename... Args> void emplace_back(Args&&... args) {
		increaseCap();
		new(theVector+(n++)) T (args...);
	}
};
```
Special Rules here:
- Args&& is a __universal reference__
- Args can point to either an lvalue or an rvalue

When is a reference universal?
- Must have the form T&&, where T is the type arg being deduced for the current template function call

Examples:
```c++
template <typename T> class C {
public:
	template <typename U> void g(U&& x); // universal

	template <typename U> void h(const U& x); // not universal

	void j(T&& x); // not universal
};
```

Now recall:
```c++
class C {...};

void f(C&& x) { 	// rvalue reference, x points to an rvalue, but x is an lvalue
	g(x); 			// x is passed as an lvalue to g
}
```

If you want to preserve the fact that x is an rvalue ref, so that a "moving" version of g is called:
```c++
void f(C&& x) {
	g(std::move(x));
}
```

In the case of args:
- We don't know if the args are lvalues, rvalues, or a mix.
- Want to call move on args __iff__ the args are rvalues.

__std::forward__ - calls _std::move_ if its argument is an rvalue ref, else does nothing.
```c++
{
...
	template <typename... Args> void emplace_back(Args&&... args) {
		increaseCap();
		new(theVector+(n++)) T (std::forward<Args> (args)...);
	}
};
```
- Now args is passed to T's constructor with lvalue/rvalue information preserved.
- This is called __Perfect Forwarding__
<hr>
[<<< Previous](12.md)   |   [Next >>>](14.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](13.md)   \|   [Next >>>](15.md)

Problem 14: Memory management is hard!
=======================================
Imagine a conversation:

*Complainer: Memory Management is hard!*

No it isn't:
- Vectors 
	- do everything arrays can
	- grow as needed in O(1) amortized time or better
	- clean up automatically once out of scope

Just use vectors, and you'll never have to manage arrays again.
C++ has enough abstraction facilities to make programming easier than C.

*But what about single objects?*

```c++
void f() {
	posn *p = new Posn{1,2};
	...
	delete p; // must deallocate the Posn!
}
```
- Do you really need to use the heap though?
- Could you use the stack instead?
```c++
void f() {
	Posn p {1,2};
	...
} // No cleanup necessary
```

Sometimes you do need the heap:
```c++
void f() {
	Posn *p = new Posn{1,2};
	if (some_condition) throw BadNews{};
	delete p;

	// p is leaked if f throws. Unacceptable.
}
```

Raising and handling an exception should not corrupt the program.
- We desire __exception safety__.

Leaks are a corruption of your program's memory, they will eventually degrade performance and crash the program.

If a program cannot recover from an exception without corrupting its memory then what is the point of recovering?

There are 3 levels of __exception safety__:
1. __Basic guarantee__ - once an exception has been handled, the program is in some valid state - No leaked memory, no corrupted data structures, all invariants maintained.
2. __Strong guarantee__ - if an exception propogates out of a function f, then the state of the program will be as if f had not been called - f either succeeds completely or not at all.
3. __Nothrow guarantee__ - a function f offers the nothrow guarantee if f never emits an exception, but must always accomplish its purpose

We will revisit this. For now, back to f...

```c++
void f() {
	Posn *p = new Posn{1,2};
	if (some_condition) {
		delete p;
		throw BadNews{};
	}
	delete p;
	// two deletes: duplicated effort! memory management is even harder!
}
```

Want to guarantee that `delete p;` happens no matter what.

What guarantees does C++ offer? Only one: Destructors for stack allocated objects will be called when the objects go out of scope.

So create a class with a destructor that deletes the pointer.

```c++
template <typename T> class unique_ptr {
	T *p;
public:
	unique_ptr(T *p): p{p} {}
	~unique_ptr() { delete p; }
	T *get() const { return p; }
	T release() { 
		T *q = p;
		p = nullptr;
		return q;
	}
}

void f() {
	unique_ptr<Posn> p {new Posn{1,2}};
	if (some_condition) throw BadNews{};
}
```
Now we have less memory management effor than we started with.

Using unique pointer: can use get to fetch the raw pointer.
Better: make unique_ptr act like a pointer.
```c++
template <typename T> class unique_ptr {
	T *p;
public:
	unique_ptr(T *p): p{p} {}
	~unique_ptr() { delete p; }
	T *get() const { return p; }
	T release() { 
		T *q = p;
		p = nullptr;
		return q;
	}

	// new:
	T &operator*() const { return *p; }
	T *operator->() const { return p; }
	explicit operator bool() const { return p; }
	// prohibits bool b = p;
	void reset(T *p1) {
		delete p;
		p = p1;
	}
	void swap(unique_ptr<T> &x) { std::swap(p, x.p); }
};

void f() {
	unique_ptr<Posn> p {new Posn{1,2}};
	cout << p->x << " " << p->y << endl;
}
```

But consider:
```c++
unique_ptr<Posn> p {new Posn{1,2}};
unique_ptr<Posn> q = p;		// 2 pointers to the same object
// when destructors are called, it will be deleted twice
```
Solution: copying unique_ptrs is not allowed. Moving is ok though!
```c++
template <typename T> class unique_ptr {
	...
public:
	unique_ptr(const unique_ptr &other) = delete;
	unique_ptr &operator=(const unique_ptr &other) = delete;
	unique_ptr(const unique_ptr &&other): p{other.p} {other.p = nullptr;}
	unique_ptr &operator=(const unique_ptr &&other) {
		swap(other);
		return *this;
	}
};
```

Small exception safety issue: Consider:
```c++
class C {...};
void f(unique_ptr<c> x, int y) {...}
int g() {...}
f(unique_ptr<c> {new c;}, g());
```
C++ does not enforce order of argument evaluation:
It could be:
1. new C
2. g()
3. `unique_ptr<c> {1}`

Then what if g throws? 1 is leaked.
- Can fix this by making 1 and 3 inseparable: use a helper function

```c++
template<typename T, typename... Args> unique_ptr<T> make_unique(Args&&... args) {
	return unique_ptr<T> {new T(std::forward<Args>(args)...)};
}

f(make_unique<C>(), g());
// no leak if g throws.
```

unique_ptr is an example of the C++ idiom: **Resource Acquisition Is Initialization (RAII)**
- any resource that must be propery released (memory, file handle, etc) should be wrapped in a stack-allocated object whose destructor frees it.
- Examples: unique_ptr, ifstream/ofstream
- Acquire the resource when the object is initialized.
- Release it when the object's destructor runs.


<hr>

[<<< Previous](13.md)   |   [Next >>>](15.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](14.md)   \|   [Next >>>](16.md)

Problem 15: Is vector exception-safe?
=====================================
Consider:
```c++
template <tyopename T> class vector {
	size_t n, cap;
	T *theVector;
public:
	vector(size_t n, const T&x): n{n}, cap{n}, 
		theVector{static_cast<T*>(operator new(n*sizeof(T)))} {
		for (size_t i = 0; i < n; ++i) {
			new(theVector + i) T (x);
			// copy constructor for T could throw, then what?
		}
	}
};
```
- Partially constructed vector - the destructor will not run.
- Broken invariant - don't have n valid objects

Note: if operator new throws: nothing has been allocated; no problem, strong guarantee

Fix:
```c++
template <typename T> vector<T>::vector (size_t n, const T&x):
	n{n}, cap{n}, theVector{static_cast<T*>(operator new(n*sizeof(T)))} {
		size_t progress = 0;
		try{
			for (size_t i = 0; i < n; ++i) {
				new(theVector + i) T(x);
				++progress;
			}
		} 
		catch (...) {	// catch anything
			for (size_t i = 0; i < progress; ++i) theVector[i].~T();
			operator delete(theVector);
			throw;		// rethrow
		}
	}
}
```

Abstract the filling part into its own function:
```c++
tempate <typename T> void uninitialized_fill(T* start, T* finish, const T&x) {
	T* p;
	try {
		for (p = start; p != finish, ++p) {
			new(static_Cast<void> (p)) T(x);
		}
	} 
	catch (...) {
		for (T*q = start; q!=p; ++q) q->~T();
		throw
	}
}

template <typename T> vector<T>::vector(size_t n, const T&x):
	n{n}, cap{n}, theVector{static_cast<T*>(operator new(n*sizeof(T)))} {
	try { 
		uninitialized_fill(theVector, theVector+n, x);
	} 
	catch(...) {
		operator delete(theVector);
		throw;
	}
}
```

Can clean this up by using RAII on the array.
```c++
template <typename T> struct vector_base {
	size_t n, cap;
	T *v;
	vector_base(size_t n): n{n}, cap{n == 0 ? 1 : n},
		v{static_cast<T*>(operator new(n*sizeof(T)))} {}
	~vector_base() {operator delete(v);}
}

template <typename T> class vector {
	vector_base<t> vb; // cleaned up implicitly when vector is destroyed.
public:
	vector(size_t n, const T&x): vb{n} {
		uninitialized_fill(vb.v, vb.v+vb.n, x);
	}
	~vector() { destroy_elements(); }
};
```

Copy constructor:
```c++
template <typename T> vector<T>::vector(const vector &other): vb{other.size()} {
	uninitialized_copy(other.begin(), other.end(), vb.v);
	// similar to uninitialized_fill, details: exercise
}
```

Exception safety of Vector
==========================
- Assignment: copy and swap is exception safe, because swap is no-throw.
- Push-back:
```c++
void push_back(const T&x) {
	increaseCap();
	new (vb.v + (vb.n)) T {x}; 
	// doesn't increment n before you know the construction succeeded
	// if this throws, have the same vector
	++vb.n;
}
```
- IncreaseCap:
```c++
void increaseCap() {
	if (vb.n == vb.cap) {
		vector_base vb2 {2*vb.cap};	// RAII
		uninitialized_copy(vb.v, vb.v+vb.n, vb2.v);	// strong guarantee
		destroy_elements();
		std::swap(vb,vb2);			// no-throw
	}
}
```

Note: only try blocks are in uninitialized_copy and uninitialized_fill

But we have an efficiency issue- copying from the old array to the new one- moving would be better
- but moving destroys the old array, so if an exception si thrown during moving, our vector is destroyed.
- we can only move if we are sure that the move operation is nothrow.

```c++
void increaseCap() {
	if (vb.n == vb.cap) {
		vector_base vb2 {2*vb.cap};
		uninitialized_copy_or_move(vb.v, vb.v+vb.n, vb2.v);
		destroy_elements();
		std::swap(vb,vb2);
	}
}

template <typename T> void uninitialized_copy_or_move(T *start, T *finish, T *target) {
	T *p;
	try {
		for (p = start; p != finish; ++p, ++target) {
			new(static_cast<void*>(target)) T (std::move_if_noexcept(*p));
		}
	}
	catch {
		for (T *q = start; q!=p; ++q) (target+(q-p))->~T();
		throw;
	}
}
```
- `std::move_if_noexcept(x)` produces `std::move(x)` if x has a non-throwing move constructor. Otherwise it produces x.
But how should the compiler know if T's move constructor is non-throwing? You have to tell it:

```c++
class C {
	public:
	C(C &&other) noexcept;
	...
};
```
In general: moves and swaps should be non-throwing if possible, so declare them so, will allow more optimized code to run.
- Any function you are sure will never throw or propogate an exception you should declare no-except.

Question: Is std::swap noexcept?
```c++
template <typename T> void swap(T&a, T&b) {
	T c (std::move(a));
	a = std::move(b);
	b = std::move(c);
}
```
Answer:
- only if T has a noexcept move constructor and a noexcept move assignment. How do we specify this?

```c++
template <typename T> void swap(T&a, T&b)
	noexcept(std::is_nothrow_move_constructible<T>::value &&
		std::is_nothrow_move_assignable<T>::value) {
	...

}
```
Note: noexcept = noexcept(true)

<hr>

[<<< Previous](14.md)   |   [Next >>>](16.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](15.md)   \|   [Next >>>](17.md)

Problem 16 - Insert/remove in the middle
========================================

A method like `vector<T>::insert(size_t i, const T &x)` is easy to write.

But the same for `list<T>` requires an up-front traversal.

Using iterators can be good for both.
```c++
template <typename T> class vector {
	...
public:
	iterator insert(iterator posn, const T&x) {
		increaseCap();
		ptrdiff_t offset = posn - begin();
		iterator newPosn = begin() + offset;
		new (static_cast<void*>(end())) T (std::move(*(end() - 1)));
		++vb.n;
		for (iterator it = end-1; it != posn; --it) {
			*it = std::move(*(it - 1));
		}
		*newPosn = x;
		return newPosn;
	}
};
```
Exception safe? Assuming T's copy/move operations are exception safe (at least basic guarantee), insert offers the basic guarantee.
- May get a partially shuffled vector, but it will be a valid vector.

Note: if you have other iterators pointing at the vector:
```
+---+---+---+---+-------+
| 1 | 2 | 3 | 4 | ..... |
+---+---+---+---+-------+
  ^	  ^   ^   ^ 
  i1  i2  h   i4
```
and you insert at "h":

```
+---+---+---+---+---+-----+
| 1 | 2 | 5 | 3 | 4 | ... |
+---+---+---+---+---+-----+
  ^	      ^   ^ 
  i1     h,i2 i4
```
- i2 will now point at a different item.

Convention: after a call to insert or erase all iterators pointing after the point of insertion/erasure are considered invalid and should not be used.

Also, if a reallocation happens, all iterators pointing into the vector become invalid.

Exercises:
- erase- remove the item pointed to by an iterator, return an iterator to the point of erasure.
- emplace- like insert, but takes constructor args.

BUT:
- that means that there is a problem with push_back.
- If increaseCap succsessfully reallocates and placement new (constructor) trhwos, the vector is the same, but iterators were invalidated

Exercise: fix this.

<hr>

[<<< Previous](15.md)  \|   [Next >>>](17.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](16.md)   \|   [Next >>>](18.md)

Problem 17 - Abstraction over Containers
========================================
Recall: map from Racket

```racket
(map f (list a b c)) ; -> (list (f a) (f b) (f c))
```

May want to do the same with vectors.

Assume: target has enough space to hold as much of source as we want to send.

```c++
template <typename T1, typename T2>
void transform(const vector<T1> &source, const vector<T2> &target, T2(*f)(T1)) {
	auto it = target.begin();
	for (auto &x: source) {
		*it = f(x);
		++it;
	}
}
```

OK, but 
- what if we only want part of the source?
- what if we want to send the source to the middle of target?

More general: use iterators.
```c++
template <typename T1, typename T2>
void transform(vector<T1>::iterator start, vector<T1>::iterator finish, vector<T2>::iterator target, T2(*f)(T1)) {
	while (start!= finish) {
		*target = f(*start);
		++start;
		++target;
	}
}
```
- this wont compile lol

Ok, but:
- If I want to transform a list, I'll write the same code again.
- What if I want to transform a list into a vector, or vice versa:
- Make the types stand for the iterators, not the container elements.
- But then how do we indicate the type for f?

```c++
template <typename InIter, typename OutIter, typename Fn>
void transform(InIter start, InIter finish, OutIter target, Fn f) {
	while (start!= finish) {
		*target = f(*start);
		++start;
		++target;
	}
}
```

Works over vector iterators, list iterators, or any other kinds of iterators.

InIter/OutIter can be any types that support `++, *, !=`, including ordinary pointers.

C++ will instatiate a template function with any type that has the operations being used by the function.

Function can be any type that supports function application.
```
class Plus {
	int n;
public:
	Plus(int n): n {n} {}
	int operator() (int m) { return n+m; }
};

Plus p{5};
cout << p(7); // 12
```
- p is called a *function object*

More interesting:
```c++
transform(v.begin(), v.end(), w.begin(), Plus{1});
// or
transform(v.begin(), v.end(), w.begin(), [](int n) {return n+1;});
// "lambda"
```

# Lambda!!!!!!!!!!!
```
			param list
			   |
lambda: [...](...) mutable? noexcept? {...}
		  |							 |
		capture list				   body
```
Semantics:
```c++
void f(T1 a, T2 b) {
	[a, &b](int x) {body}
}
```
translated to:
```c++
void f (T1 a, T2 b) {
	class haha {
		T1 a;
		T2 &b;
	public:
		haha(T1 a, T2 &b): a{a}, b{b} {}
		auto operator()(int x) const {
			body;
		}
	};
}
```
- Creates an anonymous class: lambda has a type, we cannot access the name.
- operator() is a const method, so it cannot change the field.

If the lambda is declared mutable, then operator() is not const.
- capture list: provides access to selected variables in the enclosing scope.


<hr>

[<<< Previous](16.md)  \|   [Next >>>](18.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](17.md)   \|   [Next >>>](19.md)

Problem 18 - Heterogeneous Data
===============================
I want a mixture of types in my vector >:(
- Can't do this with a template.

e.g.
```c++
vector <template <typename T> T> v;
// lol not allowed, templates are compile-time entities
```

e.g. fields of a struct
```c++
class MediaPlayer {
	template<typename T> T nowPlaying;
};
```

**What's available in C:**
- unions:
	```c++
	union Media {song s; Movie m;};
	Media nowPlaying;
	```
	- stores one or the other
	- but how do you know which one it is?
- void\* :
	- `void *nowPlaying;`
- These are not type-safe.


**Items in a heterogeneous collection usually have something in common, e.g. provide a common interface.**
- Can be viewed as different kinds of a more general "thing", so have a vector of "things" or a field of type "thing."

### We'll use the standard CS246 example:
```c++
class Book {
	string title, author;
	int length;
public:
	Book(string title, string author, int length): 
		title{title}, author{author}, length{length} {}
	bool isHeavy() const { return length > 100; }
	string getTitle() const { return title; }
	// etc.
};
```

```
Book:
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+
```

```c++
class Text: public Book {
	string topic:
public:
	Text(string title, string author, int length, string topic):
		Book{title,author,length}, topic{topic} {}
	bool isHeavy() const { return length > 500; }
	string getTopic() const { return topic; }
};
```

```
Text:
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+
| topic      |
+------------+
```

```c++
class Comic: public Book {
	string hero;
public:
	comic(string title, string author, int length, string hero):
		Book{title,author,length}, topic{topic} {}
	bool isHeavy() const { return length > 50; }
	string getHero() const { return hero; }
};
```

```
Comic:
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+
| hero       |
+------------+
```

Subclasses inherit all members (fields and methods) from their superclass.
All three classes have fields: title, author, length, and methods getTitle, getAuthor, getLength, isHeavy.
...except that this doesnt work
```c++
bool Text::isHeavy() const {return length > 500;}
// length is private in Book, text cannot access it.
```

2 options:
1. 
```c++
class Book{
	string title, author;
protected:
	int length; // accessible only to this class and its subclasses.
public:
	...
};
```

2. 
```c++
bool Text::isHeavy() const {return getLength() > 500; }
// call public method.
```

Recommended to use option 2: 
- You have no control over what subclasses might do.
- protected weakens encapsulation; cannot enforce invariants on protected fields.

If you want subclasses to have privileged access:
- keep the fields private
- provide protected __get__ and __set__ methods.

Updated object creation/destruction protocols
### Creation Protocol:
1. Space is allocated
2. Superclass part is constructed
3. Fields constructed in declaration order
4. Constructor body runs

### Destruction Protocol:
1. Destructor body runs
2. Fields destructed in reverse declaration order
3. Superclass part is destructed
4. Space is deallocated

Must revisit everything we know to see the effect of inheritance.

Type compatibility 
- Texts and Comics are special kinds of books.
- They should be usable in place of Books.
```c++
Book b = Comic{"Spoderman", "Stan Lee", 75, "Spiderman"};
b.isHeavy();	// false. 75 is a heavy comic, but not a heavy book.
```
If b is a Comic, why is it acting like a Book?
- Because b is a book.

Consequence of **stack-allocated** objects:
```
Book b - sets aside enough space to hold a Book.
+---------+
|         |
+---------+
|         |
+---------+
|         |
+---------+

Comic {...} - doesn't fit into the book sized hole.
+---------+
|         |
+---------+
|         |
+---------+
|         |
+---------+
|         |
+---------+
```
- Keeps only the book part - the comic part is chopped off.
- This is known as **slicing.**
- So it really is just a Book now, therefore `Book::isHeavy` runs.
- Slicing happens even if superclass and subclass are the same size.

Similarly, if you want to collect your books:
```c++
vector <Book> library;
library.push_back(Comic{...});
```
- only the book parts will be pushed - not a heterogeneous collection.

Also note:
```c++
void f(Book books[]); // Raw array
Comic comics[] = {...};
f(comics);
```
- Will compile, but never do this! Undefined behavior.
- array will be misaligned, and will not act like an array of books.

Slicing does not happen through pointers. However:
```c++
Book *p = new Comic{"Spoderman", "Stan Lee", 75, "Spiderman"};
p->isHeavy(); // still false!
```
- the choice of which isHeavy to run is based on the type of the pointer (static type), not the type of the object (dynamic type).
- Why? Cheaper. C++ Design Principle: if you don't use you, you shouldn't have to pay for it.

i.e. if you want something more expensive, you have to ask for it.
To make \*p act like a Comic when it is a Comic:
```c++
class Book {
	...
public:
	...
	virtual bool isHeavy() const {...};
};

class Comic: public Book {
	...
public:
	...
	bool isHeavy() const override {...};
};

p->isHeavy(); // true!!!!
```

Assume isHeavy is virtual.

Can now have a truly heterogeneous colection.
```c++
vector<Book*> library;
library.push_back(new Book{...});
library.push_back(new Comic{...});
```

even better: `vector<unique_ptr<Book>> library;`

Now we can do this:
```c++
int howManyHeavy(const vector<Book*>&v) {
	int count = 0;
	for (auto &b:v) {
		if (b->isHeavy()) ++count;
	}
	return count;
}
```
- correct version of isHeavy always chosen, even though we don't know what's in the vector, and the items are probably not the same type.
- polymorphism

```c++
for (auto &b:library) delete b; // not necessary if library is a vector of unique_ptrs
```
How do virtual methods work and why are they more expensive?
- Implementation-dependent, but the following is typical:

```
Book b1;
+------------+
| vptr       | // points at book vtable.
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+

Book b2;
+------------+
| vptr       | // points at book vtable.
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+

comic c;
+------------+
| vptr       | // points at comic vtable.
+------------+
| title      |
+------------+
| author     |
+------------+
| length     |
+------------+
| hero       |
+------------+


// Vtables: virtual methods only (decided at runtime)
+------------+
| "Book"     |
+------------+
| isHeavy    | // points at Book::isHeavy
+------------+

+------------+
| "Comic"    |
+------------+
| isHeavy    | // points at Comic::isHeavy
+------------+
```

Non-virtual methods - ordinary function calls.

If there is at least one virtual method 
- compiler creates a table of function pointers
	- one per class
	- called the vtable
- each object cointains a pointer to its class's vtable - the vptr.
- calling the virtual method: 
	- follow the vptr to the vtable
	- follow the function pointer to the correct function.

vptr is often the first field:
- so that a subclass object still looks like a superclass object
- so the program knows where the vptr is
- so virtual methods incur a cost in time (extra pointer dereferences) and space (each object gets a vptr)
- if a subclass doesn't override a virtual method, its vtable will point to the superclass implementation.

<hr>

[<<< Previous](17.md)  \|   [Next >>>](19.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](18.md)   \|   [Next >>>](20.md)

# Problem 19 - I'm Leaking!
Consider:
```c++
class X {
	int *a;
public:
	X (int n): a{new int[n]} {}
	~X() { delete [] a }
};

class Y: public X {
	int *b;
public:
	Y(int n, int m): X{n}, b{new int[m]} {}
	~Y() {delete [] b;}
};

// Note: Y's destructor will call X's destructor (Step 3)

X *px = new Y{3,4};
delete px; // Leaks.
```

Calls X's destructor, but not Y's.
Solution - make the destructor virtual.
```c++
class X {
	...
public:
	...
	virtual ~X() {delete [] a;}
}
```
- No more leak.

Always make the destructor virtual in classes that are meant to be superclasses. Even if the destructor does nothing, you never know what the subclass might do, so you need to make sure its destructor gets called.

If a class is not meant to be a superclass- no need to incur the cost of virtual methods.
- leave the destructor non-virtual.

```
class X final { // cannot be subclassed.
	...
};
```

<hr>

[<<< Previous](18.md)  \|   [Next >>>](20.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](19.md)   \|   [Next >>>](21.md)

# Problem 20 - I want a class with no objects
```c++
class Student {
 public:
  virtual float fees() const;
};

class RegularStudent: public Student {
 public:
  Float fees() const override; // regular student fees.
};

class CoOpStudent: public Student {
 public:
  Float fees() const override; // co-op student fees
}
```

What should student::fees do?

Don't know - every student should be Regular or Co-op.

Solution: explicitly give `Student::fees` no implementation.
```c++
class Student { // called an abstract class
 public:
  virtual float fees() const = 0; // called a pure virtual method
};
```
Abstract classes cannot be instatiated.
```c++
Student s;
Student *s = new Student;
```
Can point to instances of concrete classes.
```c++
Student *s = new RegularStudent;
```

Subclasses of abstract classes are abstract, unless they implement all of the pure virtual methods.

Abstract classes 
- used to organize concrete classes
- can contain common fields, methods, default implementation.

<hr>

[<<< Previous](19.md)  \|   [Next >>>](21.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](20.md)   \|   [Next >>>](22.md)

# Problem 21 - The copier is broken.

How do copies and moves interact with inheritance?

Copy constructor: `Text::Text(const Text &other): Book{other}, topic{other.topic} {}`

Move constructor: `Text::Text(Text &&other): Book{std::move(other)}, topic{std::move(other.topic)} {}`

Copy/Move assignment:
```c++
Text &Text::operator=(Text other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic);
}
```

But consider:
```c++
Book *b1 = new Text{...}, *b2 = new Text{...};
*b1 = *b2;
```
What happens? - only the Book part is copied - partial assignment
- topic doesn't match the title and author, object is corrupted.

Possible solution- Make operator= virtual
```c++
class Book {
  ...
  public:
    virtual Book *operator=(const Book &other);
};

class Text: public Book {
  ...
  public:
    // Text &operator=(const Text &other) override;
    Text &operator=(const Book &other) override; // must take a Book or its not an override.
};
```
- This doesn't compile

Another solution: Make all superclasses abstract.

Old:
```
   [Book]
    /   \
[Text] [Comic]
```
Instead:
```
    [Abstract Book]
     /     |     \
 [Book] [Text] [Comic]
```

```c++
class AbstractBook {
  ...
  protected:
    AbstractBook &operator=(AbstractBook other) {...} // non virtual

};

class Text: public AbstractBook {
  public:
    Text &operator=(Text other) {
      AbstractBook::operator=(std::move(other));
      topic = std::move(other.topic);
    }
    // operator= non virtual, therefore no mixed assignment
};
```
`AbstractBook::operator=` not accessibile to outsiders, therefore `*b1 = *b2` won't compile.


<hr>

[<<< Previous](20.md)  \|   [Next >>>](22.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](21.md)   \|   [Next >>>](22a.md)

# Problem 22 - I want to know what kind of Book I have
For simplicity: Assume:
```
   [Book]
    /   \
[Text] [Comic]
```

The C++ casting operators - 4 operators.

C style casting: `(type) expr`
- forces expr to be treated as type "type"
- e.g. 
```c++
int *p;
int q = (int) p;
```

1. static_cast - for conversions with a well-defined semantics
- e.g.
```c++
void f(int a); void f(double d);
int x;
f(static_cast<double>(x));
```
e.g. superclass pointer to subclass pointer
```c++
Book *b = new Text {...};
Text *t = static_cast<Text*>(b);
```
- You have to know that b really points to a Text, otherwise undefined behavior.

2. reinterpret_cast - for casts without a well-defined semantics
- unsafe, inplementation-dependent
```c++
Book *b = new Book {...};
int *p = reinterpret_cast<int*>(b);
```

3. const_cast
- for adding and removing const
- the only C++ cast that can "cast away const"
```c++
void g(Book &b); // assume we know g won't change b
f(const Book &b) {
  g(const_cast<Book&>(b));
}
```

4. dynamic_cast
- What if we don't know whether pb poitns at a Text?
```c++
Book *pb = ...
Text *pt = dynamic_cast<Text*>(pb);
// if *pb is a Text or a subclass of text, then cast succeeds, pt points at the object
// else, pt = nullptr;
```
e.g.
```c++
void whatIsIt(Book *pb) {
    if (dynamic_cast<Text*>(pb)) cout << "Text";
    else if (dynamic_cast<Comic*>(pb)) cout << "Comic";
    else cout << "Book";
}
```
- not good style - when you create a new book type the function has to be changed.

Dynamic reference casting:
```c++
Book *pb = ...;
Text &t = dynamic_cast<Text&>(*pb);
// If *pb is a Text - ok
// else, raises std::bad_cast
```

Dynamic Casting works by accessing an object's Run-Time Type Information (RTTI)
- this is stored in the vtable for the class
- therefore, can only dynamic_cast on objects with at least one virtual method.

Dynamic Reference Casting offers a possible solution to the polymorphic assignment problem.
```c++
Text &Text::operator=(Book other) {
  Text &textother = dynamic_cast<Text&>(other); // throws if other is not a Text
  if (this == &textother) return *this;
  Book::operator=(std::move(textother));
  topic=std::move(textother.topic);
  return *this;
}
```

<hr>

[<<< Previous](21.md)  \|   [Next >>>](22a.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](22a.md)   \|   [Next >>>](24.md)

# Problem 23: Shared Ownership
Is C++ hard? No (if you're a client programmer)

But explicit memory management
- use vector if you need an array
- use unique_ptr if you need a heap obj.
- use stack-allocated objects as much as possible

If you follow these, you should never have to say delete or delete[].

But unique_ptr doesn't respect is-a

```c++
unique_ptr<Base> p (new Derived); // OK
p->virtfn(); // runs derived version, OK

unique_ptr<Derived> q = ...
unique_ptr<Base> = std::move(q); // Type error.
// No conversion between unique_ptr<Derived> and unique_ptr<Base>
// Note: std::unique_ptr can do this, but our version cannot.
```

Easy to fix:
```c++
template <typename T> class unique_ptr {
	T *p;
	...
	public:
	...
	template <typename U> unique_ptr(unique_ptr<U> &&q): p{q.p} {q.p = nullptr;}
	template <typename U> unique_ptr &operator=(unique_ptr<U> &&q) {
		std::swap(p, q.p);
		return *this;
	}
}
```
- Works for any unique_ptr whose ptr is assignment_compatible with this->p
e.g. subtypes of T, but not supertypes of T.

But I want two smart ptrs pointing at the same object. 
- Why? the pointer that owns the object should be a unique_ptr
- all others could be raw pointers

When would you want true shared ownership?

Recall (Racket):
```racket
(define l1 (cons 1 (cons 2 (cons 3 empty))))
(define l2 (cons 4 (rest (l1))))
+-+-+   +-+-+   +-+-+
|1|*|-->|2|*|-->|3|\|
+-+-+   +-+-+   +-+-+
         ^
+-+-+    |
|4|*|----+
+-+-+
```
- Shared data structures ar ea nightmare in C - how can we ensure each node is freed exactly once?
- Easy in garbage collected languages.

What can C++ do?
```c++
template <typename T> class shared_ptr {
	T *p;
	int *refcount;
	public:
};
// refcount counts how many shared_ptrs point at *p
// updated each time a shared_ptr is initalized/assigned/destroyed
// refcount is shared among all shared_ptrs that point to *p
// p is only deleted if its refcount reaches 0.
// implementation details - exercise
```

Now deallocation is easy - as easy as garbage collection

Just watch: cycles
```c++
+-+-+   +-+-+
|1|*|-->|2|*|
+-+-+   +-+-+ 
 ^         |
 +---------+
```
- If you have cyclic data - you may have to physically break the cycle. (or use weak_ptrs)

Also watch: 
```c++
Book *p = new ...
shared_ptr<Book> p1 {p};
shared_ptr<Book> p2 {p};
```
- shared_ptrs are not mind-readers
- p1 and p2 will not share a ref count. - BAD
- if you want 2 shared_ptrs at an object, create one shared_ptr and copy it.

But - you can't dynamic_cast these pointers:
```c++
template <typename T, typename U> 
shared_ptr<T> dynamic_pointer_cast(const shared_ptr<U> &spu) {
	return shared_ptr<T>(dynamic_cast<T*>(spu.get()));
}
similarly- const_pointer_cast, static_pointer_cast
```

<hr>

[<<< Previous](22a.md)  \|   [Next >>>](24.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](23.md)   \|   [Next >>>](25.md)

# Problem 24: Abstraction over Iterators

I want to jump ahead n spots in my container.
```c++
template <typename Iter> Iter advance(Iter it, ptrdiff_t n);
```

How should we do it?
```c++
for (ptrdiff_t r = 0; i < n; ++i) ++it;
return it; 

// slow - O(n)
// can't we just say it += n?
```

Depends - for vectors - yes O(1) time.
- for list - no, += not supported

Related - can we go backwards?
- vectors - yes
- lists - no

So all iterators support !=, \*, +
- but some iterators support other operations
- `list::iterator` - called a **forward iterator** - can only go one step forward.
- `vector::iterator` - called a **random access iterator** - arbitrary pointer arithmetic

How can we write advance to use += for random-access iterators, and a loop for forward iterators?

Since we have different kinds of iterators, let's create a type hierarchy:
```c++
struct input_iterator_tag{};
struct output_iterator_tag{};
struct forward_iterator_tag: public input_iterator_tag{};
struct bidirectional_iterator_tag: public forward_iterator_tag{};
struct random_access_iterator_tag: public bidirectional_iterator_tag{};
```

To associate each iterator class with a tag - could use inheritance:
```c++
class list {
	...
	public:
	class iterator: public forward_iterator_tag {
	...
	};
};
```
- But, makes it hard to ask what kind of iterator we have (Can't dynamic_cast, no vtables)
- doesn't work for iterators that aren't classes (e.g. ptrs for vectors)

Instead - make the tag a memeber:
```c++
class list{
	...
	public: 
	class iterator {
		...
		public:
		using iterator_category = forward_iterator_tag;
	};
	// convention - every iterator class will define a type member called iterator_category
};
```
- also doesn't work for iterators that aren't classes.
- But we aren't done yet.
- Make a template that associates ever iterator type with its category:

```c++
template <typename It> struct iterator_traits {
	using iterator_category = typename It::iterator_category;
}
// eg. iterator_traits<List<T>::iterator>::iterator_category => forward_iterator_tag
```

Provide a specialized version for pointers:
```c++
template <typename T> struct iterator_traits<T*> {
	using iterator_category = random_access_iterator_tag;
};
```
Why typename? Needed that so that C++ can tell that It::iterator_category is a type..

consider:
```c++
template <typename T> void f() {
	T::something x;
}

// but:

template <typename T> void f() {
	T::something *x; 
	// ptr declaration or multiplication? need to know if T::something is a type
	// C++ always assumes is value unless told otherwise
}

// so do this:
template <typename T> void f() {
	typename T::something x;
	typename T::something *y;
}

// need to say typename whenever you refer to a member type of a template param.
```

Back to iterator_traits
For any iterator type T, iterator_traits<T>::iterator_category resolves to the tag struct for T. (Incl. if T is a pointer):

What do we do with this?

Want:
```c++
template <typename Iter>
Iter advance(Iter it, ptrdiff_t n) {
	if (typeid(typename iterator_traits<Iter>::iterator_category) 
		== typeid(random_access_iterator_tag)) {
		return it += n;
	} else {
		...
	}

}
```
- Won't compile.
- If the iterator is not random access, and doesn't have a += operator, it += n will cause a compilation error, even though it will never be used.
- Moreover, the choice of which implementation to use is being made at run-time, when the right choice is known at compile-time

To make a compile-time decision - overloading
- make a dummy parameter with type of the iterator tag.
```c++
template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, random_access_iterator_tag) {
	return it += n;
}

template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, bidirectional_iterator_tag) {
	if (n > 0) for (ptrdiff_t i = 0; i < n; ++i) ++it;
	else if (n < 0) for (ptrdiff_t i = 0; i > n; --i) --it;
	return it;
}

template <typename Iter>
Iter doAdvance(Iter it, ptrdiff_t n, forward_iterator_tag) {
	if (n >= 0) {
		for (ptrdiff_t i = 0; i < n; ++i) ++it;
		return it;
	}
	throw SomeError{};
}
```

Finally, create a wrapper function to select the right overload:
```c++
template <typename Iter>
Iter advance(Iter it, ptrdiff_t n) {
	return doAdvance(it, n, typename iterator_traits<Iter>::iterator_category {});
}
```

Now the compiler will select the fast doAdvance for random access iterators, the slow doAdvance for bidirectional iterators, and the throwing doAdvance for forward iterators.

These choices made at compile-time - no runtime cost.

Using templates to perform compile-time computation - called **template metaprogramming**

C++ templates form a functional language that operates at the level of types.
- Express conditions by overloading, repetition via recursive template instatiation.

Example:
```c++
template <int N> struct Fact {
	static const int value = N * Fact<N-1>::value;
};

template<> struct Fact<0> {
	static const int value = 1;
};

int x = Fact<5>::value; // 120 - evaluated at compile-time!!!
```

But for compile-time computation of values, C++ 11/14 offer a more straightforward facility:

```c++
constexpr int fact(int n) {
	if (n == 0) return 1;
	return n * cat(n-1);
}
```
### constexpr functions:
- means evaluate this at compile time if possible, e.g. if arguments known at compile time.
- else, evaluate at runtime.
- A consexpr funciton must be something that can actually be evaluated at compile-time, can't be virtual, can't mutate non-local vars, etc.

<hr>

[<<< Previous](23.md)  \|   [Next >>>](25.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](24.md)   \|   [Next >>>](26.md)

# Problem 25: I want an even faster vector.

In the good old days of C, you could copy an array (even an array of structs) very quickly, by calling memcpy (similar to strcpy, but for arbitrary memory, not just string)

memcpy was probably written in assembly, and was as fast as the machine could possibly be.

Nowadays in C++, copies invoke copy constructures, which are costly function calls.

But there is good news.
In C++, a type is considered POD (plain old data) if it:
- has a trivial default constructor (equiv. to =default)
- is trivially copyable - copy/move operations, destructor have default implementations
- is standard layout - no virtual methods or bases, all members have same visibility, no reference members, no fields in both base class and subclass, or in multiple base classes.

For POD types, semantics is compatible with C, and memcpy is safe to use.

How can we use it - only safe if T is a POD type.

One option: use std::is_pod<T>::value at runtime.
```c++
template <typename T> class vector {
    size_t size, cap;
    T *theVector;
    public:
    ...
    vector(const vector &other): ... {
        if (std::is_pod<T>::value) {
            memcpy(theVector, other.theVector, n*sizeof(T));
        }
        else {
            // as before
        }
    }
};
```
- Works, but condition is evaluated at run-time, but result is known at compile time. (compiler may or may not optimize)

Second option: no runtime cost
```c++
template <typename T> class vector {
    ...
    public:
    ...
    template <typename X=T> 
    vector(enable_if<std::is_pod<X>::value, const T&>::type other): ... {
        memcpy(theVector, other.theVector, n*sizeof(T));
    }

    template <typename X=T>
    vector(enable_if<!std::is_pod<X>::value, const T&>::type other): ... {
        // original implementation
    }
};
```
How does this work?
```c++
template <bool b, typename T> struct enable_if;
template <typename T> struct enable_if<true, T> {
    using type = T;
};
```
With metaprogramming, what you don't say is as important as what you do say.

If b is true, enable_if defines a struct whose 'type' member typedef is T.

So if `std::is_pod<T>::value` is true, then `enable_if<is_pod<T>::value, const vector<T>&>::type` is `const vector<T>&`

If b is false, the struct is declared but not defined, so `enable_if<b, T>` will not compile. One of the two versions of the copy constructor won't compile (false condition).

Then how is this a valid program?

**C++ rule: SFINAE - Substitution Failure Is Not An Error**
- In other words, if t is a type, `template <typename T> _ f (_) {_}` is a template function, and substituting `T = t` results in an invalid function, the compiler does not signal an error, it just removes that instantiation from consideration during overload resolution.
- On the other hand, if no version of the function is in scope and substitutes validly, that is an error.

Question: Why is this wrong?
```c++
template <typename T> class vector {
    ...
    public:
    vector(typename enable_if<std::is_pod<T>::value, const vector<T>&>::type other): ... {...}
};
```
I.e. why do we need the extra template outfront?

Because SFINAE applies to template functions, and these methods are ordinary functions, not templates
- They depend on T, but T's value was determined when you decided what to put in the vector.
- If substituting T=T fails, it will invalidate the entire vector class, not just the method.

So make the mthod a separate template, with a new arg X, which can be defaulted to T, and do `is_pod<x>`, `not is_pod<x>`.

...this compiles, but when we run it, it crashes.

Why? Hint: if you put debug statements into both of these constructors, they don't print.
- Answer: we're getting the compiler supplied copy constructor, which is doing shallow copies.

These templates are not enough to suppress the auto-generated copy constructor
- and a non-templated match is always preferred to a templated one.

What do we do about it? Could try:
```c++
template <typename T> class vector {
    ...
    public:
    vector(const vector &other)=delete; // i.e. disable the copy constructor
};
```
- However the compiler won't let you do this, can't disable the copy constructor and then create one.

Solution that works: overloading.
```c++
template <typename T> class vector {
    struct dummy{};
    ...
    public:
    vector(const vector &other): vector{other, dummy{}} {}
    // this is constructor delegation.
    template <typename X=T>
    vector(typename enable_if<std::is_pod<T>::value, const vector <T>&>::type other, dummy) {
        ...
    }
    template <typename X=T>
    vector(typename enable_if<!std::is_pod<T>::value, const vector <T>&>::type other, dummy) {
        ...
    }
}
```
- overload the constructor with an unused dummy arg.
- have the copy constructor delegate to the overloaded constructor
- copy constructor is inline, so no function call overhead
- **this works.**

As of 2014, can write some "helper" definitions to make `is_pod` and `enable_if` easier to use.
```c++
template <typename T> constexpr bool is_pod_v = std::ispod<T>::value;
template <bool v, typename T> using enable_if_t = typename enable_if<T>::type;

template <typename T> class vector {
    ...
    public:
    ...
    template <typename X=T> vector(enable_if_t<is_pod_v<x>, const vector<T>&> other, dummy) {...}
     template <typename X=T> vector(enable_if_t<!is_pod_v<x>, const vector<T>&> other, dummy) {...}
};
```

## Move/forward implementation:
- We not have enough machinery to implement `std::move` and `std::forward`.

`std::move` - first attempt
```c++
template <typename T> T&& move(T &&x) {
    return static_cast<T&&> (x);
}
```
- Doesn't quite work. T&& is a universal ref, not an rvalue ref.
- If x was an lvalue ref, T&& is an lvalue ref.
- Need to make sure T is not an lvalue ref.
    - If T is an lvalue ref, get rid of the ref.

std::move - correct:
```c++
template <typename T> inline typename std::remove_reference<T>::type&& move(T&& x) {
    return static_cast<typename std::remove_reference<T>::type &&>(x);
}
```
- `std::remove_reference<T>` turns t&, t&& into t.
- Exercise - Write remove_reference

Question: Can we save typing and use auto?
```c++
template <typename T> auto move(T &&x) {...}
```

Answer: NO!
```c++
int z;
int &y = z;
auto x = y; // x is int.
```

By ref auto&& is a universal reference.

Need a type definition rule that doesn't discard refs

decltype(...) - returns the type that ... was declared to have.
decltype(var) - returns the declared type of the var
decltype(expr) - returns lvalue or rvalue ref, depending on whether the expr is an lvalue or rvalue.

```c++
int z;
int &y = z;
decltype(y) x = z; // x is int&
x = 4; // affects z

auto a = z;
a = 4; // does not affect z.

decltype (z) s= z; // s is an int
s = 5; // does not affect z.

decltype((z)) r = z; // r is int&
r = 5; // does affect z.
```

Thus:
```c++
decltype(auto) - perform type deduction, like auto, but use decltype rules.
```

Correct move:
```c++
template <typename T> decltype(auto) move(T &&x) {
    return static_cast<std::remove_reference_t<T> &&>(x);
}
```

`std::forward`:
```c++
template <typename T> inline T&& forward(T &&x) {
    return static_cast<T&&>(x);
}
```
- Reasoning: If x is an lvalue, T&& is an lvalue ref. If x is an rvalue, T&& is an rvalue ref.
- Doesn't work. forward is called on exprs that are lvalues, that may "point" at rvalues.

e.g.
```c++
template <typename T> void f(T&& y) {
    ...forward(y)...
}
```
- forward(y) will always yield an lvalue ref.

In order to work, forward must know what type (including l/rvalue) was deduced for y, i.e. needs to know T.

So in principle, `forward<T>(y)` would work.

But: 2 problems
- Supplying T means T&& is no longer universal.
- Want to prevent user from omitting `<T>`

Instead: separate lvalue/rvalue cases.
```c++
template <typename T> inline constexpr T&& forward(std::remove_reference_t<T> &x) noexcept {
    return static_cast<T&&>(x);
}
template <typename T> inline constexpr T&& forward(std::remove_reference_t<T> &&x) noexcept {
    return static_cast<T&&>(x);
}
```


<hr>

[<<< Previous](24.md)  \|   [Next >>>](26.md)[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](25.md)   \|   [Next >>>](27.md)

# Problem 26: Collecting Stats
I want to know how many Students I create.

```
class Student {
    int assns, mt, final;
    static int count; // static - associated with the class, not one per object
  public:
    student(...) { ++count; }
    // accessors
    static int getCount() {return count;} 
    // static method - have no "this" param, not really methods, scoped functions.
}
```

.cc:
```c++
int Student::count = 0; // must define the var
Student s1{...}, s2{...}, s3{...};
cout << Student::getCount() << endl; // no receiver obj.
```

Now I want to count objects in other classes. How do we abstract this solution into reusable code?

```c++
template <typename T> struct Count {
    static int count;
    Count() {++count;}
    Count(const Count &) {++count;}
    Count(Count &&) {++ count;}
    ~Count(){--count;}
    public:
    static int getCount() {return count;}
}

template <typename T> int Count<T>::count = 0;


// private inheritance
// inherits count's impl, without creating an is_a relationship
// members of Count become private in Student.
class Student: Count<Student> {
    int assns, mt, final;
    public:
    Student(...): .... {}
    // accessors
    using Count<Student>::getCount; // makes Count::getCount visible.
};

class Book: Count<Book> {
    ...
    public:
    using Count<Book>::getCount;
};

```
Why is Count a template?
- So that for each class C, class C: Count<C> creates a new, unique instantiation of Count, for each C.
- This gives each C its own counter, vs. sharing one for over all subclasses.
- This technique - inheriting from a template specialized by yourself
    - looks weird, but happens enough to have its own name: the **Curiously Recurring Template Pattern (CRTP)**


<hr>

[<<< Previous](25.md)  \|   [Next >>>](27.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](26.md)   \|   [Next >>>](28.md)

# Problem 27: Resolving Method Overrides at Compile-Time
Recall: Template Method Pattern:

```c++
class Turtle {
    public:
    void draw() {
        drawHead();
        drawShell();
        drawFeet();
    }
    private:
    void drawHead();
    virtual void drawShell() = 0;
    void drawFeet();
};

class RedTurtle: public Turtle {
    void drawShell() override;
};
```

Consider:
```c++
template <typename T> class Turtle {
    public:
    void draw() {
        drawHead();
        static_cast<T*>(this)->drawShell();
        drawFeet();
    }
    private:
    void drawHead();
    void drawFeet();
}

class RedTurtle: public Turtle<RedTurtle> {
    friend class Turtle;
    void drawShell();
}
```
- No virtual methods, no vtable lookup

```c++
class GreenTurtle: public Turtle<GreenTurtle> {
    ...
};
```
Drawback: 
- no relationship between RedTurtle + GreenTurtle, cant store a mix of them in a container.
- Can give Turtle class a parent.

```c++
template <typename T> class Turtle: public Enemy {
    ...
}
```

Then can store RedTurtle and GreenTurtles.
- But can't access draw method.
- Could give Enemy a virtual method.

```c++
class Enemy {
    public:
    virtual void draw() = 0;
};
```
- but then there will be a vtable lookup.
- on the other hand, if Turtle::draw calls several would be virtual helpers, could trade away several vtable lookups for one.


<hr>

[<<< Previous](26.md)  \|   [Next >>>](28.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](27.md)   \|   [Next >>>](29.md)

# Problem 28 - Polymorphic Cloning
```c++
Book *pb = ...;
Book *pb2 = // I want an exact copy of *pb.
```
- Can't call a constructor directly, don't know what \*pb1 is. Don't know which constructor to call.

Standard Solution: virtual clone method.
```c++
class Book {
    ...
    public:
    virtual Book *clone() {return new Book{*this};}
};

class Text {
    ...
    public:
    Text *clone() override {return new Text{*this};}
};

// Comic - similar
```

Boilerplate code - can we reuse?

Works better with an abstract base class.
```c++
class AbstractBook {
    public:
    virtual AbstractBook *clone() = 0;
    virtual ~AbstractBook();
};

template <typename T> class Book_cloneable: public AbstractBook {
    public:
    T *clone() override {return new T{static_cast<T&>(*this)};}
};

class Book: public Book_cloneable<Book> {...};
class Text: public Book_cloneable<Text> {...};
// comic - similar
```


<hr>

[<<< Previous](27.md)  \|   [Next >>>](29.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](28.md)   \|   [Next >>>](30.md)

# Problem 29: Logging
- want to encapsulate logging functionality and "add" it to any class.

```c++
template <typename T, typename Data> class Logger {
    public:
    void loggedSet(Data x) {
        cout << "setting data to" << x << endl;
        static_cast<T*>(this)->set(x);
    }
};

class Box:public Logger<Box, int> {
    friend class Logger<Box, int>;
    int x;
    void set(int y) {x = y;}
    public:
    Box(): x{0} {loggedSet(0);}
};

Box b;
b.loggedSet(1);
b.loggedSet(4);
```

Another approach:

```c++
class Box {
    int x;
    public:
    Box(): x{0} {}
    void set(int y) {x = y;}
};

template <typename T, typename Data> class Logger: public T {
    public:
    void loggedSet(Data x) {
        cout << "Setting data to" << x << ednl;
        set(x); // no vtable overhead.
    }
};

using Boxlogger = Logger<Box,int>;
BoxLogger b;
b.loggedSet(1);
b.loggedSet(4);
```

**Mixins** - can mix and match subclass functionality without writing new classes.
- Note: if SpecialBox is a subclass of Box, then SpecialBox has no relation to Logger<Box,int>.
- Nor is there any relationship between Logger<SpecialBox,int> and Logger<Box,int>

But with CRTP, SpecialBox is a subptype of Logger<Box,int>
- can specialize behavior of virtual functions.



<hr>

[<<< Previous](28.md)   \|   [Next >>>](30.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](29.md)   \|   [Next >>>](31.md)


# Problem 30 - Total Control

Can control:
- param passing
- initialization
- method call resolution
- etc.

### Control over memory allocation
Memory allocators are tricky to write - 2 Questions: Why and how?

#### Why write an allocator?
- built-in one is too slow
    - general purpose, not optimized for specific use.
    - eg. if you know you will always allocate objects of the same size, a custom allocator may perform better.
- you want to optimize locality.
    - maybe you want a separate heap, jsut for objects of some class C - keeps the objects close to each other, may improve performance
- you want to use "special memory"
    - put objects in shared memory
- you want to profile the program.
    - collect stats

### How do you customize allocation?
- overload operator new
    - if you define a global non-member operator new, all allocations in your program will use your allocator.
    - Also, if you write operator new, you need to write operator delete. otherwise undefined behavior.

```c++
void *operator new(size_t size) {
    cout << "Request for " << size << " bytes \n";
    return malloc(size);
}

void operator delete(void *p) {
    cout << "Freeing " << p << endl;
    free(p);
}

int main() {
    int *x = new int;
    delete x;
}
```
- Works, but is not correct. Doesn't adhere to convention. If operator new fails, it is supposed to throw bad_alloc.
- Actually, if operator new fails, it is supposed to call the new_handler function.
    - the new_handler can: 
        - free up space(somehow)
        - install a different new_handler / uninstall the current
        - throw bad_alloc
        - abort or exit
    - new_handler should be called in an infinite loop
    - if new_handler is nullptr, operator new throws
- Also - new must return a valid pointer if size == 0, and delete of a null ptr must be safe.

Corrected implementation:
```c++
#include <new>
void *operator new(size_t size) {
    cout << "Request for " << size << " bytes \n";
    if (size == 0) size = 1;
    while (true) {
        void *p = malloc(size);
        if (p) return p;
        std::new_handler h = std::set_new_handler(0);
        std::set_new_handler(h);
        if (h) h();
        else throw std::bad_alloc{};
    }
}

void operator delete(void *p) {
    if (p == nullptr) return;
    cout << "Freeing " << p << endl;
    free(p);
}
```

Replacing global operator new/delete affects your entire program.

More likely - replace on a class-by-class basis
- especially if you are writing allocators specifically tuned to the sizes of your objects.

To do this - make operator new/delete members
- must be *static* members.

```c++
class C {
    public: ...
    static void *operator new(size_t size) {
        cout << "Running C's allocator" << endl;
        return ::operator new(size);
    }
    static void operator delete(void *p) noexcept {
        cout << "Freeing " << p << endl;
        ::operator delete(p);
    }
};
```

Generalize - log to an abritrary stream
```c++
class C {
    public:
    static void *operator new(size_t size, std::ostream &out) {
        out << ... << endl;
        return ::operator new(size);
    }
};

C *x = new(cout) c; // log to cout
ofstream f {...};
C *y = new(f) c;
```

Must also write the corresponding "placement" delete.
```c++
class C {
    public:
    ...
    static void operator delete(void *p, std::ostream &out) noexcept {
        out << "Placement delete: " << p << endl;
        ::operator delete(p);
    }
};
```
Won't compile! You also need to write ordinary delete.

```c++
class C {
    public:
    ...
    static void operator delete(void *p) noexcept {
        out << "Ordinary delete (cout): " << p << endl;
        ::operator delete(p);
    }
};

// This will compile, however:

C *p = new (cout) c;    // Running C's allocator
delete p;               // Ordinary delete (cout)
```
- If the client calls delete p, there needs to be a non-specialized operator delete, else compile error.
- How can you call specialized delete? You can't. Then why do we need it?
    - If the constructor that called specialized operator new throws, it will call the specialized operator delete that matches operator new.
    - If there isn't one, no delete gets called, so leak.

e.g.
```c++
class C {
    ...
    public:
    C (bool b) {if (b) throw 0; }
};

try {
    C *p = new (cout) C {true}; // throws
    delete p; // not reached
}
catch (...) {}

C*q = new (cout) C {false};
delete q;
```

Recall - specialized member new
- Specialized member delete AND standard member delete must be written.

Customizing array allocation - overload `operator new []` and `operator delete []`


<hr>

[<<< Previous](29.md)   \|   [Next >>>](31.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](30.md)   \|   [Next >>>](32.md)

# Problem 31 - I want total control over vectors and lists

Incorporating custom allocation into our containers.

Issue - may want different allocators for different kinds of vectors

Solution - Make the allocator an arg to the template.
- Since most users won't write allocators, we will need a default value.

Template signature:
```c++
template <typename T, typename Alloc=allocator<T>> class vector {...};

// now write the interface for allocators + the allocator template:

template <typename T> struct allocator {
    using value_type = T;
    using pointer = T*;
    using reference = T&;
    using const_pointer = const T*;
    using const_reference = const T&
    using size_type = size_t;
    using difference_type = ptrdiff_t;
    allocator() noexcept {}
    allocator(const allocator &) noexcept {}
    template <typename U> allocator(const allocator<u> &) noexcept {}
    ~allocator() {}
    pointer address(reference x) const noexcept {return &x;}
    pointer allocate(size_type n) {return ::operator new(n*sizeof(T));}
    void dellocate(pointer p, size_type n) {::operator delete(p);}
    size_type max_size() const noexcept {
        return std::numeric_type<size_type>::max/sizeof(value_type);
    }
    template <typename U, typename... Args>
    void constrcut(U *p, Args&&... args) {
        ::new(static_cast<void*>(p))U(forward<Args>(args)...);
    }
    template <typename U> void destroy(U *p) {p->~U();}
};
```

If you want to write an allocator for an STL container, this is its interface.
Note:
- operator new takes a # of bytes, but allocate takes a # of objects.

What happens if a vector is copied? copy the allocator?

What happens if you copy an allocator? Can 2 copies allocate/deallocate each other's memory?

- C++03 - allocators must be stateless, so copying allocators is allowed and trivial.
- C++11 - allocators can have state, must specify own copying behavior via allocator traits.

Adapting vector:
- vector has a field Alloc alloc;
- everywhere the vector calls:
    - operator new,     replace with alloc.allocate
    - placement new,    replace with alloc.construct
    - explicit destructor            alloc.destroy
    - operator delete                alloc.deallocate
    - takes an address               alloc.address

Details - exercise.

Can we do the same with list?
Not exactly.
```c++
template <typename T, typename Alloc=allocator<T>> class list {...};
```
- Correct so far... but curiously, Alloc will never be used to allocate memory in list.
- Why not? lists are node-based, means you don't want to actually allocate T's objects, you want to allocate nodes (which contain T objs and pointers)
- but Alloc allocates T objs.
- Every conforming allocator has a member template called rebind that gives the allocator type for another type:

```c++
template <typename U> struct allocator {
    ...
    template <typename U> struct rebind {
        using other = allocator<U>;
    };
    ...
};
```
Within list- to create an allocator for nodes as a field of list:
```c++
Allocator::rebind<Node>::other alloc;
```
Then use as in vector. Details: exercise.

<hr>

[<<< Previous](30.md)   \|   [Next >>>](32.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](31.md)   \|   [Next >>>](33.md)

# Problem 32: A fixed-size object allocator
A custom allocator can be fast.

Fixed size allocator: all allocated chunks are the same size (i.e customized for one class), so no need to keep track of sizes.

(aside - many traditional allocators store the size of the block before the pointer, so that the allocator knows how much space is attatched to that pointer.)

Fixed-size - saves space (no hidden size field)
- saves time - no hunting for a block of the right size.

Approach:
Create a pool of memory - an array large enough to hold n T objects.

Initially:
```
+--+--+--+--+--+-----+--+
|1 |2 |3 |4 |5 | ... |-1|
+--+--+--+--+--+-----+--+
 0  1  2  3  4        n-1
```

When the client has a slot: T obj:
When we have it:
- node in a linked list.
- each slot stores the index of the next slot in the list.

Allocation: from the front.
```
First free: 1
+--+--+--+--+--+-----+--+
|//|2 |3 |4 |5 | ... |-1|
+--+--+--+--+--+-----+--+
 0  1  2  3  4        n-1

First free: 2
+--+--+--+--+--+-----+--+
|//|//|3 |4 |5 | ... |-1|
+--+--+--+--+--+-----+--+
 0  1  2  3  4        n-1
```

Deallocation:
```
Free item 0:
First free: 0
+--+--+--+--+--+-----+--+
|2 |//|3 |4 |5 | ... |-1|
+--+--+--+--+--+-----+--+
 0  1  2  3  4        n-1

Free item 1:
First free: 1
+--+--+--+--+--+-----+--+
|2 |0 |3 |4 |5 | ... |-1|
+--+--+--+--+--+-----+--+
 0  1  2  3  4        n-1
```

Implementation:
```c++
template <typename T, int n> class fsAlloc {
    union Slot { // slot large enough for a T, but still usable as an int.
        int next;
        T data;
        Slot(): next{0} {}
    };
    Slot theSlots[n];
    int first = 0;
    public:
    fsAlloc() {
        for (int i = 0; i < n; ++i) theSlots[i] = i+1;
        theSlots[n-1] = -1;
    }
    T *allocate() noexcept {
        if (first == -1) return nullptr;
        T *result = &(theSlots[first].data);
        first = theSlots[first].next;
        return result;
    }
    void deallocate(void *item) noexcept {
        int index = (static_cast<char*>(item)-reinterpret_cast<char*>(theSlots))/sizeof(Slot);
        // char* has size 1
        theSlots[index].next = first;
        first = index;
    }
};
```

Use in a class:
```c++
class Student final {
    int assns, mt, final;
    static fsAlloc<Student, SIZE> pool; // size is how many you can allocate.
    public:
    ...
    static void* operator new(size_t size) {
        if (size != sizeof(Student)) throw std::bad_alloc;
        // already kinda avoided because Student is final class:
        // can't have subclasses which are likely larger
        while (true) {
            void *p = pool.allocate();
            if (p) return p;
            std::new_handler h = std::set_new_handler(0);
            std::set_new_handler(h);
            if (h) h();
            else throw std::bad_alloc{};
        }
    }
    static void operator delete(void *p) noexcept {
        if (p == nullptr) return;
        pool.deallocate(p);
    }
};

fsAlloc<Student,SIZE> Student::pool;
```

main.cc:
```
int main() {
    Student *s1 = new Student;
    Student *s2 = new Student; // these are using custom allocator
    delete s1;
    delete s2; // these are using custom deallocator
}
```

Q: Where do s1, s2 reside?

A: static memory (Not the heap)
- could arrange for stack/heap memory

More notes: we used a union to simultaneously hold both int and T.
- could have used a struct `[next][T]`
- disadvantage: if you access a dangling T pointer, you can corrupt the linked list.

eg. 
```c++
Student *s = new Student;
delete s;
s->setAssns(...); // destroys the pool
```
Lesson: following a dangling pointer can be very dangerous.

With a struct, next is before the T obj, so you have to work hard to corrupt it.

```c++
reinterpret_cast<int *>(s)[-1] = ...;
```

On the other hand - with a struct - problems if T doesn't have a default constructor.

eg.
```c++
struct Slot {
    int next;
    T data;
};
slot theSlots[n]; // doesn't work if T has no default constructor.
// can't do operator new/placement new
```

```c++
union SlotChar {
    char dummy;
    Slot S;
    SlotChar():dummy{0} {}
};
```
Also, why store indices instead of pointers?
- smaller than pointers on this machine, so waste no memory as long as T >= size of an int.
- would waste if T smaller than an int
- could use a smaller index type (as long as you don't want more items than the type can hold)
- could make the index type a param of the template.
Student final - fixed size allocator - subclasses might be larger
    - options have no subclasses.
    - check size, throw if it isn't the right size.
    - or, give derived class its own allocator.
    
<hr>

[<<< Previous](31.md)   \|   [Next >>>](33.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](32.md)

# Problem 33 - I want a (tiny bit) smaller vector class

Currently: vector/vector_base have an allocator field

Standard allocator is stateless - has no fields

What is its size? 0? No - C++ does not allow 0 size types, messes things up like pointer arithmetic.

Every type has size >= 1. 

Practically - copiler adds a dummy char to the allocator.

So having an allocator field makes the vector larger by a byte. (probably more, due to alignment)

Having an allocator field in vector_base may add another byte (or more)

To save this space, C++ provides the empty base optimization.

Under EBO - an empty base class does not have to occupy space in an object.

So, we can eliminate the space cost of an allocator by making it a base class.

At the same time, lets make vector_base a base class of vector.

```c++
struct vector_base: Alloc {
    size_t n, cap;
    T* v;
    using Alloc::allocate;
    using Allot::deallocate;
    // etc.
    vector_base(size_t n): n{0}, cap{n}, v{allocate(n)} {}
    ~vector_base() {deallocate(v);}
};

template <typename T, typename Alloc=allocator<T>>

class vector:vector_base <T,Alloc> { //private inheritance, no is-a
    using vector_base<T,Alloc>::n;
    using vector_base<T,Alloc>::cap;
    using vector_base<T,Alloc>::v;
    using Alloc::allocate;
    using Alloc::deallocate; // or say this->allocate, this->deallocate
    public:
    ...
    // now use n,cap,v instead of vb.n, vb.cap, vb.v
}
```

`uninitialized_copy`, etc - need to call construct/destroy
- simplest - let them take an allocator as param
```c++
template <typename T, typename Alloc>
void uninitialized_fill(T *start, T *finish, const T&a, Alloc a) {
    ...
    a.construct(...);
    ...
    a.destroy(...);
    ...
}
```

How can vector pass an allocator to these functions?
`uninitialized_fill(v,v+n,x,static_cast<Alloc&>(*this));`
- cast yourself to base class reference.
Remaining details - exercise.

# Conclusions
C++ is easy because it is hard.

<hr>

[<<< Previous](32.md)
[**TABLE OF CONTENTS**](toc.md)

[<<< Previous](22.md)   \|   [Next >>>](23.md)

# A Big Unit on Object Oriented Design

## System Modelling - UML (Unified Modelling Language)
- Make ideas easy to communicate
- aid design discussions

```
- private
# protected
+ public

			+--------------------+
			|Book                | - class name, italics = abstract
			+--------------------+
			|-title:String       |
			|-author:String      | } - fields (optional)
			|#length:Integer     |
			+--------------------+
			|+getTitle:String    |
			|+getAuthor:String   | } - methods (optional)
			|+getLength:Integer  |   - italics = virtual
			|+/isHeavy/:Boolean  |
			+--------------------+
		               ^
		               |
		   +-----------+------------+
		   |                        |
+--------------------+      +------------------+
|Text                |      |Comic             |
+--------------------+      +------------------+
|-topic:String       |      |-hero:String      |
+--------------------+      +------------------+
|+getTopic:String    |      |+getHero:String   |
|+/isHeavy/:Boolean  |      |+isHeavy:Boolean  |
+--------------------+      +------------------+


+------------+    m +-------+
|Car         |⬦---->|Motor  |
+------------+      +-------+
|-VIN:Integer|
+------------+
owns-a relationship (composition)
- means the motor is part of the car
	- does not have an independent existence
	- copy/destroy the car => copy/destroy the motor (deep copies)
- typical implementation - class composition, i.e. object fields

e.g.
class Car {Motor m;};


+----------+  0..*+-------+
|Pond      |⬦---->|Duck   |
+----------+      +-------+
has-a relationship (aggregation)
- duck has its own independent existence
- copy/destroy the pond !=> copy/destroy the ducks
- typical implementation - pointer field

e.g.
class Pond {vector <Duck*> ducks;};
```
- The concept of ownership is central to Object Oriented design in C++
- Every resource (memory, file, window, etc.) should be owned by an object that will release it - RAII
- A unique_ptr owns the memory it points to.
- A raw pointer should be regarded as not owning the memory it points to.
- If you need to point at the same object with several pointers, one pointer should own it and be a unique ptr. The rest should be raw pointers. Moving a unique_ptr = transferring ownership
- If you need true shared ownership - later


## Measures of Design Quality
- Coupling and Cohesion
- Coupling - how many different modules depend on each other
	- low to high:
		- function calls with params/results of basic type
		- function clals with array/struct params
		- modules affect each other's control flow
		- modules sharing global data
		- modules access each other's implementation (friends)
- Cohesion - how closely are elements of a module related to each other
	- low to high:
		- arbitrary grouping (e.g. utility library)
		- common theme, otherwise unrelated, maybe some common base code (e.g. algorithm)
		- elements manipulate state over the lifetime of an object (e.g. open/read/close files)
		- elements pass data to each other
		- elements cooperate to perform exactly one task

High coupling - changes to a module affect other modules
- harder to reuse individual modules

E.g. function whatIsIt(dynamic_cast) - tightly coupled to the Book hierarchy, must change this function if you create another Book subclass

low cohesion => poorly organized code - hard to understand, maintain
Want high cohesion, low coupling.

## SOLID Principles of OO Design

### 1: Single Responsibility Principle
- a class should only have one reason to change
	- i.e. a class should do one thing, not several
- Any change to the problem spec require a change to the program
- If changes to >= 2 different parts of the spec cause changes to the same class, SRP is violated.
E.g. Don't let your (main) classes print things

Consider:
```c++
class Chessboard{
	...
	cout << "Your move";
	...
};
```
- Bad design, inhibits code reuse.

What if you want a version of your program that:
- Communicates over different streams (file/network)?
- Works in another language?
- Uses graphics instead of text?

- Major changes to Chessboard class, instead of reuse. Violates SRP.
	- must change the class if there is any change to the specificatino for:
		- game rules
		- strategy
		- interface
		- etc.
	- low cohesion situation, should split these up.
		- one module responsible for communication
		- if a class wants to say something - do it via params, results, exceptions
			- pass info to communications object and let it do the talking
	- On the other hand - specificaitons that are unlikely to change may not need their own class, avoid needless complexity. Judgement call.

### 2: Open/Closed Principle
- Classes, Modules, Functions, etc. should be open for extension, and closed for modification
- Changes in a program's behaviour should happen by writing new code- not by changing old code.
```
+------------+      +---------+
|Carpenter   |⬦---->|HandSaw  |
+------------+      +---------+
```
What if the carpenter buys a tablesaw?
- this design is not open for extension, have to change the carpenter code.

Solution: Abstraction
```
+------------+      +---------+
|Carpenter   |⬦---->|/Saw/    |
+------------+      +---------+
                         ^
                         |
               Handsaw --+-- Tablesaw
```

Also note:
```c++
int countHeavy(const vector<Book*> &v) {
	int count = 0;
	for (auto &p:v) if (p->isHeavy()) ++count;
	return count;
}
```

Note: can't really be 100% closed. Same chanes may require source modification.
Plan for the most likely changes and make your code closed with respect to those changes.

### 3: Liskov Substitution Principle
- Simply put: public inheritance must indicate an Is-A relationship.
- But there smore to it:
	- If B is a subtype (subclass) of A, then we should be able to make an object b of type B in any context that requires an object of type A, without affecting the correctness of the program.
- C++'s inheritance rules already allow us to use subclass objects in place of superclass objects.
- \* - this is the important point.
- Informally, the program should "not be able to tell" if it is using a superclass object or a subclass object.

Formally: If an invariant I is true of class A, then it must be true of class B.
If an invariant I is true of method A::f, and B::f overrides A::f, then I must hold for B::f.
If A::f has a precondition P and a postcondition Q, then B::f must have a precondition P'<= P, and a postcondition Q' => Q.

If A::f and B::f behave differently, the difference in behavior must fall within what is allowed by the program's correctness specification.

Examples:
1. Contravariance Problem
	- Arises any time you have a binary operator, i.e. a method with an other parameter of the same type as \*this.
	- Violates Liskov Substitution.
		- A circle is a shape
		- A shape can be compared with any other shape.
		- Therefore Circle can be compared with any other shape.
		- We saw this with virtual operator=
		- C++ will flag this problem iwtha compiler error.

```
    Shape
      |
   +--+--+
   |     |
Circle  Square
```

```c++
class Shape {
	public:
	virtual bool operator==(const shape &other) const;
};

class Circle: public Shape {
	public:
	bool operator==(const Shape &other) const override;
};
```
	
Fix:
```c++
#include <typeinfo>
bool Circle::operator==(const Shape &other) const {
	if (typeid(other) != typeid(Circle)) return false;
	const Circle &cother = static_cast <const Circle &> (other);
	// compare fields of other with fields of *this.
}
```

#### dynamic_cast vs typeid:
`dynamic_cast <Circle &> (other);`
	- is other a Circle or a subclass of Circle
`typeid(other) == typeid(Circle)`
	- is other precisely a circle?
	- typeid returns an object of type typeinfo.

Is square a rectangle?
- a square has all the properties of a rectangle.

```c++
class Rectangle {
	int length, width;
	public:
	Rectangle(...);
	int getLength() const; 
	virtual void setLength(int);
	int getWidth() const;
	virtual void setWidth(int);
	int area() const{ return length * width; }
};

class Square: public Rectangle {
	public:
	Square(int side): Rectangle (side, side) {}
	void setLength(int l) override {
		Rectangle::setLength(l);
		Rectangle::setWidth(l);
	}
	void setWidth(int w); //similar.
}

int f(Rectangle &r) {
	r.setLength(10);
	r.setWidth(20);
	return r.area(); // expect 200
}

Sqaure s {1};
f(s); // 400
```

Rectangles have the property that their length and width can vary independently; Squares don't have that property.
- Therefore, this violates LSP.
- On the other hand, an immutable square could substitute for an immutable Rectangle.

```
		       Shape
		         ^
		         |
		    +----+---...
		    |
/RightAngledQuadrilateral/
            ^
            |
       +----+-----+
       |          |
    Square    Rectangle
```
- Constraining what subclasses can do

Consider:
```c++
class Turtle {
	public:
	virtual void draw() = 0;
};

class RedTurtle: public Turtle {
	public:
	void draw() override {
		drawHead(); drawRedShell(); drawFeet();
	}
};

class GreenTurtle: public Turtle {
	public:
	void draw() override {
		drawHead(); drawGreenShell(), drawFeet();
	}
}
```
- code duplication
- Plus - how can we ensure that overrides of draw will always do these things?

```
class Turtle {
	public:
	void draw() {
		drawHead();
		drawShell();
		drawFeet();
	}
	private:
	void drawHead();
	virtual void drawShell() = 0;
	void drawFeet();
};

class RedTurtle: public Turtle {
	void drawShell() override;
};

class GreenTurtle: public Turtle {
	void drawShell() override;
};
```

Subclasses cannot control the steps of drawing a turtle, nor the drawing of head + feet.
Can only control the drawing of a Shell - called the **Template Method Pattern**.

Extension - **Non-virtual Interface** (NVI) idiom
- public virtual methods are simultaneously: 
	- part of a class's interface - pre/post conditions, respect invariatnes
	- "hooks" for customization by subclasses - overriding code could be anything

NVI says: all virtual methods should be private, i.e. all public methods should be non-virtual.
Eg.
```c++
class DigitalMedia {
	public:
	virtual void play() = 0;
};

// should be

class DigitalMedia {
	public:
	void play() {
		doPlay();
	}
	private:
	virtual void doPlay() = 0;
};
```
- generalizes template method pattern
- puts every virtual function inside a template method.

### 4: Interface Segregation Principle
Many small interfaces is better than one large interface.
- If a class has many functionalities, each client of the class should only see the functionality it needs. 

Eg. video game
```c++
class Enemy {
	public:
	virtual void strike();	// needed by game logic
	virtual void draw();	// needed by user interface
};

class UI {
	vector<Enemy*> v;
};

class Battlefield {
	vector<Enemy*> v;
};
```
If we need to chang ethe drawing interface, Battlefield must recompile for no reason.

Creates needless coupling between UI and Battlefield.

One solution: Multiple Inheritance
```c++
class Draw {
	public:
	virtual void draw() = 0;
};

class Combat {
	public:
	virtual void strike() = 0;
};

class Enemy: public Draw, public Combat {};

class UI {
	vector <Draw *> v;
};

class Battlefield {
	vector <Combat *> v;
};
```
Example of the Adapter Pattern
General use of Adapter - when a class provides an interface different from the one you need:
```
Eg.

+----------------+   +----------------+
|Needed Interface|   |Provided Class  |
+----------------+   +----------------+
|+g()            |   |+f()            |
+----------------+   +----------------+
        ^                     ^
        |                     |
        +----------+----------+
                   |
            +--------------+
            |Adapter       |
            +--------------+  +--------------+
            |+g()----------|--|{f();}        |
            +--------------+  +--------------+

```
Detour: issues with Multiple Inheritance
```
+------+    +------+
|A1    |    |A2    |
+------+    +------+
|+a()  |    |+a()  |
+------+    +------+
    ^         ^
    |         |
    +----+----+
         |
       +---+
       |B  |
       +---+ - has 2 a() methods

       +---+
       |A  |
       +---+
         ^
         |
    +----+-----+
    |          |
 +---+        +---+
 |B  |        |C  |
 +---+        +---+
    ^          ^
    |          |
    +----+-----+
         |
       +---+ - has 2 a() methods, and they're different.
       |D  |
       +---+
```

```c++
class D: public B, public C {
	void f() {... a() ...}
};

D d;
d.a(); // use d.B::a() or d.C::a()
```

OR,

Maybe there should only be one A base, and therefore only one a().

```c++
class B: virtual public A {...};
class C: virtual public A {...};
d.a() - no longer ambiguous
```

Eg. iostream hierarchy
```
       ios_base
          |
         ios
       v/   \v
  istream   ostream
        \   /
       iostream
        /    \
  fstream    stringstream
```

Problem- How will a class like D be laid out in memory? (implementation specific)

Consider:
```
+---------+ <--- should look like an A*, B*, C*, D*, but doesn't look like a C*.
|vptr     |
+---------+
|A fields |
+---------+
|B fields |
+---------+
|C fields |
+---------+
|D fields |
+---------+

but g++ gives us:

+---------+ <--- D*, B*(?)
|vptr     |
+---------+
|B fields |
+---------+ <--- C*(?)
|vptr     |
+---------+
|C fields |
+---------+
|D fields |
+---------+ <--- A*
|vptr     |
+---------+
|A fields |
+---------+
```
- B and C need to be laid out so that we can find the A part, but the distance is not known (depends on the run-time type of the object)
- **Solution:** location of base object stored in vtable.
- Also note - diagram doesn't simultaneously look like A,B,C,D, but slices of it do.
Therefore pointer assignment among A,B,C,D ptrs  may change the address stored in the pointer.

```c++
D *d = ...;
A *a = d; // changes the address.
```
- static_cast, const_cast, dynamic_cast under Multiple Inheritance will also adjust the value of the pointer.
- reinterpret_cast will not.

### 5: Dependency Inversion Principle
- High level modules should not depend on low-level modules. Both should depend on abstraction.
- Abstract classes should never depend on concrete classes.

Traditional top-down design:
```
+------------------+         +----------------+
|High-level module |-------->|Low-level module|
+------------------+         +----------------+

e.g.
+------------------+         +----------------+
|Word Count        |-------->|Keyboard Reader |
+------------------+         +----------------+
- what if I want to use a file reader?
- changes to details affect the higher level word count module.
```

Dependency inversion:
```
+------------------+         +---------------------+
|High-level module |-------->|Low-level Abstraction|
+------------------+         +---------------------+
                                        ^
                                        |
                                        |
                             +---------------------+
                             |Low-level module     |
                             +---------------------+

+------------------+         +---------------------+
|WordCount         |-------->|/Input Interface/    |
+------------------+         +---------------------+
                                        ^
                                        |
                                 +------+------+
                                 |             |
                        +------------+      +----------+
                        |Keyboard    |      |File      |
                        |Reader      |      |Reader    |
                        +------------+      +----------+
```

Works over several layers as well.
```
high -> medium -> low

=>

high -> medInterface
             ^
             |
           medium -> lowInterface
                          ^
                          |
                         low
```

eg.
```
timer⬦---->bell
           +notify()

when the timer hits some specified time, it rings the bell.
(calls Bell::notify, which rings the bell)

What if we want the timer to trigger other events?

timer⬦---->/responder/
           +notify()
                ^
                |
          +-----+------+
          |            |
        Bell         Lights
        +notify()    +notify()
```

Maybe we want a dynamic set of responders:
```
+---------------------+        +-------------+
|Timer                |⬦------>|Responder    |
+---------------------+        +-------------+
|+register(Responder) |<------⬦|+notify()    |
|+unregister(Responder)        +-------------+
+---------------------+               ^
                                      |
                                +-----+------+
                                |            |
                               Bell         Light

Now Responder is depending on the concrete timer class, apply dependency inversion again.
                              *
/Source/⬦--------------------->/Responder/
+register(Responder)           +notify()
+unregister(Responder)             ^
    ^                              |
    |                        +-----+------+
    |                        |            |
Timer()<------------------⬦Bell     +--⬦Light
    ^                               |
    +-------------------------------+

If Bell/Light's behavior depends on the time, 
may need to depend on the concrete timer for a getTime method.
- could dependency invert this again, if you wanted.

General Solution: known as the Observer Pattern.
+----------------+             +----------------+ 
|/Subject/       |             |/Observer/      |
+----------------+⬦----------->+----------------+ 
|+notifyObservers|             |+notify()       |
|+attatch(Obs)   |             +----------------+ 
|+detatch(Obs)   |                      ^
+----------------+                      |
        ^                               |
        |                               |
+----------------+             +-----------------+
|ConcreteSubject |             |Concrete Observer|
+----------------+<-----------⬦+-----------------+
|+getState()     |             |+notify()        |
+----------------+             +-----------------+
```

Sequence of Calls:
1. Subject's state changes
2. `Subject::notifyObservers(either by the Subject itself or an external controller)`
	- calls each observer's `notify()`
3. Each observer calls concreteSubject::getState to query the state and react accordingly

# Some more design patterns

**Factory Method Pattern** - when you don't know exactly what kind of object you want, and your preferences may vary.
- also called the virtual constructor pattern
- strategy pattern applied to object construction.

```
        +-----------+
        |/Enemy/    |
        +-----------+
              ^
              |
        +-----+-----+
        |           |
    +-------+    +-------+
    |Turtle |    |Bullet |
    +-------+    +-------+

- randomly generated
- more turtles in easy levels
- more bullets in hard levels

        +-----------+
        |/Level/    |
        +-----------+
              ^
              |
        +-----+-----+
        |           |
    +-------+    +-------+
    |Easy   |    |Hard   |
    +-------+    +-------+

class Level {
	public:
	virtual Enemy* getEnemy() = 0;
};

class Easy: public Level {
	public:
	Enemy *getEnemy() override {
		// mostly turtles
	}
};

class Hard: public Level {
	public:
	Enemy *getEnemy() override {
		// mostly bullets
	}
};

Level *l = new Easy;
Enemy *e = l->getEnemy();
```

Decorator Pattern - add/remove functionality to/from objects at runtime
eg. add menus/scrollbars to windows - either or both
- without a combinatorial explosion of subclasses.

```
             +-------------+
             |/Component/  |
             +-------------+<----------------+
             |+/Operation/ |                 |
             +-------------+                 |
                    ^                        |
                    |                        |
       +------------+------------+           |
       |                         |           |
+------------------+      +--------------+   |
|Concrete Component|      |/Decorator/   |⬦--+
+------------------+      +--------------+
|+Operation        |             ^
+------------------+             |
                                 |
                       +---------+------------+
                       |                      |
             +-------------------+   +-------------------+
             |ConcreteDecoratorA |   |ConcreteDecoratorB |
             +-------------------+   +-------------------+
             |+Operation         |   |+Operation         |
             +-------------------+   +-------------------+
```

Every Decorator IS a component and HAS a component.
- window w/ scrollbar is a kind of window, and has a pointer to the underlying plain window.
- window w/ scrollbar and menu is a window and has a pointer to a window w/ scrollbar, which as a pointer to a plain window.

Eg.
```c++
WindowInterface *w = new WindowWithMenu {
	new WindowWithScrollbar {
		new Window {}}};
```

Pizza Example in repository.

### Visitor Pattern
- for implementing **double dispatch**
	- method chosen based on the runtime of 2 objects

```
        +-----------+
        |/Enemy/    |
        +-----------+
              ^
              |
        +-----+-----+
        |           |
    +-------+    +-------+
    |Turtle |    |Bullet |
    +-------+    +-------+



        +-----------+
        |/Weapon/   |
        +-----------+
              ^
              |
        +-----+-----+
        |           |
    +-------+    +-------+
    |Stick  |    |Rock   |
    +-------+    +-------+
```
- The effect of striking an enemy with a weapon depends on both the enemy and the weapon.
- C++ virtual methods are dispatched on the type of the receiver object, and not method params. No way to specify two receiver objects.

Visitor - two-stage dispatch, combining overriding with overloading.
```c++
class Enemy {
	public:
	virtual void beStruckBy(Weapon &w) = 0;
};

class Turtle: public Enemy {
	public:
	void beStruckBy(Weapon &w) override {w.strike(*this);}
};

class Bullet: public Enemy {
	public:
	void beStruckBy(Weapon &w) override {w.strike(*this);}
};

class Weapon {
	public:
	virtual void strike(Turtle &t) = 0;
	virtual void strike(Bullet &b) = 0;
};

class Stick: public Weapon {
	public:
	void strike(Turtle &t) override {
		//strike turtle with stick
	}

	void strike(Bullet &b) override {
		//strike bullet with stick
	}
}
```

```c++
Enemy *e = new Bullet{...};
Weapon *e = new Rock{...};
e->beStruckBy(*w); 

// What happens?
// Bullet::beStruckBy runs (virt. method dispatch)
// which calls Weapon::strike(Bullet &b) since *this is a Bullet
// this fact is known at compile time, overload resolution

// virtual method resolves to Rock::Strike(Bullet &)
```

Visitor can also be used to add functionality to a class hierarchy without adding new virtual methods.

Add a visitor to the book hierarchy:
```c++
class Book {
	...
	public:
	...
	virtual void accept(BookVisitor &v) {v.visit(*this);}
};

class Text: public Book {
	...
	public:
	void accept(BookVisitor &v) override {
		v.visit(*this);
	}
};

class BookVisitor {
	public:
	virtual void visit(Book &b) = 0;
	virtual void visit(Text &t) = 0;
	virtual void visit(Comic &c) = 0;
};
```

Example: Categorize and Count.
- For Books: - by Author
- Texts: - by Topic
- Comics: - by Hero

Could do this with a virtual method, or write a visitor.
```c++
class Catalog: public BookVisitor {
	public:
	map<string, int> theCat;
	void visit(Book &b) override {++theCat[b.getAuthor()];}
	void visit(Text &t) override {++theCat[b.getTopic()];}
	void visit(Comic &c) override {++theCat[c.getHero()];}
};
```

### But it won't compile!
- Circular include dependency
- book.h, BookVisitor.h include each other.
- include guard prevents multiple inclusion
- whichever ends up occurring first will refer to things not yet defined.
- needless includes create artificial compilation dependencies, and slow down compilation, or prevent compilation altogether.

Sometimes a forward class declaration is not good enough.

Consider:
```c++
class A {...}; // A.h

class B {
	A a;
};

class C {
	A *a;
};

class D: public A {
	...
};

class E {
	A f(A);
};

class F {
	A f(A a) {a. someMethod();}
};

class G {
	t<A> x;
};
```
Which need includes? B,D,F need includes

C,E forward declare ok.

G - it depends on how the template t uses A.
- should collapse to one of the other cases.

Note: class F only needs an include because method f's implementation is present.
- a good resason to keep implementation in .cc
- where possible: forward declare in .h, include in .cc

Also notice: B needs an include; C does not.
- If we want to break the compilation dependency of B on A, we could make B like C.

More generally:
```c++
class A1{}; class A2{}; class A3{};

class B {
	A1 a1;
	A2 a2;
	A3 a3;
};
```

b.h:
```c++
class BImpl;

class B {
	unique_ptr<BImpl> pImpl;
};
```

bimpl.h:
```c++
#include "a1.h"
...

struct BImpl {
	A1 a1;
	A2 a2;
	A3 a3;
};
```

b.cc:
```c++
#include "b.h"
#include "bimpl.h"

methods reference pImpl -> a1, a2, a3
```

b.h no longer compilation-dependent on a1.h, etc.
- called the pImpl idiom.

Another advantage of pImpl - pointers have a non-throwing swap.

Can provide the strong guarantee on a B method by
- copying the Impl into a new BImpl structure (heap-allocated)
- method modifies the copy
- if anything throws, discard the new structure (easy and automatic with unique_ptrs)
- if all succeeds swap impl structs (ptrswap - nothrow)
- previous impl automcatically destroyed by the smartptr.

Example:
```c++
class B {
	unique_ptr<BImpl> pImpl;
	...
	void f() {
		auto temp = make_unique<BImpl>(*pImpl);
		temp->doSomething();
		temp->doSomethingElse();
		std::swap(pImpl,temp); // nothrow
	} // strong guarantee
};
```

<hr>

[<<< Previous](22.md)  \|   [Next >>>](23.md)