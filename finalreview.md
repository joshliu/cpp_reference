# FINAL REVIEW

Argument Dependent Lookup (ADL) - also called könig lookup.
- If the type of a function f's argument belongs to a namespace n, then C++ will search the namespace n, as well as the current scope, for a function matching f.
- This is the reason why we can say "std::cout << x" instead of "std::operator <<(std::cout, x)"

Formally: methods differ from functions in that methods have an implicit param called <u>this</u> that is a pointer to the receiver object.

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

