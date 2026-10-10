# **C/C++ Common Errors**

- **Stack-Overflow**

  1. When we allocate too much on the stack (e.g. large local arrays or very deep recursion),
  to the point that we exceed the maximum stack size, we can get a stack overflow even if there
  is no out-of-bounds indexing. The maximum stack size depends on the OS and linker settings
  (e.g. ~8 MB default on Linux, check with `ulimit -s`; 1 MB default on Windows/MSVC, changed with `/STACK`).
  
  2. ...
