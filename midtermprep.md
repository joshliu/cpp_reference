lvalue vs rvalue
```
f(int &x) {...}
g(int &&x) {...}
h(const int &x) {...}

1. Pick an STL Data Structure... Implement it.
- <queue>
- <stack>
- <priority_queue>
- <unordered_map>
- <list>
2. istream_iterator<int>(cin)
3. Use iterators

What the heck is RAII
- Resource Acquisition is Initialization
- Wrap resources in stack objects so that their destructors are guaranteed to run.
- vector_base, uniqueptr