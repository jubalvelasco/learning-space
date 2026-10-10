## Boost Predefined Macros

https://www.boost.org/doc/libs/1_82_0/libs/predef/doc/index.html#_using_the_predefs

## compiler macros
```c++
#if defined(__clang__)
// clang only code goes here
#elif defined(__GNUC__) || defined(__GNUG__)
// gnu-gcc only code goes here
// (clang also defines __GNUC__, that's why __clang__ must be checked first)
#elif defined(_MSC_VER)
// microsoft compiler only code goes here
#endif
```

## assertion macro
```c++
#include <iostream>
#include <exception>

#define ENABLE_ASSERT

// the do { ... } while(0) wrapper makes the macro behave like a single statement,
// so `if (x) M_ASSERT(y); else ...` works correctly
#ifdef ENABLE_ASSERT
#define M_ASSERT(expr) \
      do { \
         if(!(expr)){ \
            std::cerr << "ASSERTION FAILURE: \n"; \
            std::cerr << " => Condition: " << #expr << "\n"; \
            std::cerr << " =>  Function: " << __func__ << "\n"; \
            std::cerr << __FILE__ << ":" << __LINE__ << ":" << "\n"; \
            std::terminate(); \
         } \
      } while(0)
#else
#define M_ASSERT(expr) do {} while(0)
#endif
```

## debug macro

```c++
#if defined(DEBUG_MODE)
  #define disp(expr)  std::cerr << __FILE__ << ":" << __LINE__ << ":" \
      << " ; " <<  #expr << " = " << (expr)  <<  std::endl

  #define dbgloc(msg)  std::cerr << __FILE__ << ":" << __LINE__ << ":" << msg 
  #define dbgline __FILE__ << ":" << __LINE__ << ":" <<
  #define dbgtrace(msg) std::cerr << __FILE__ << ":" << __LINE__ << ": TRACING " << msg << "\n";
#else
  #define disp(expr) 
  #define dbgloc(msg)  
  #define dbgline 
  #define dbgtrace(msg) 
#endif 
```

## Type assertion
```c++
#include <type_traits>

// `expr_type` must be a type, e.g. ASSERT_TYPE_EQUAL(int, decltype(x));
#define ASSERT_TYPE_EQUAL(type, expr_type) \
   static_assert(std::is_same<type, expr_type>::value, "Expected type equality")
```
 
 ## Predefined Macros
 
| Macro | Description | Compiler |
| --- | --- | --- |
| ```__LINE__``` | Contains current line number of the program. | all |
| ```__FILE__``` | Name of current file. | all |
| ```__DATE__``` | String with the compilation date in the format `"Mmm dd yyyy"` (e.g. `"Jan 14 2024"`) | all |
| ```__TIME__``` | String with the compilation time in the format `"hh:mm:ss"` | all |
| ```__cplusplus``` | Integer (`long`) constant of the C++ standard version | all |
| ```__func__``` | Name of the current function (standard since C99/C++11, it's a variable, not a macro) | all |
| ```__FUNCTION__``` | Name of the current function or method (member function), non-standard but supported by GCC, Clang and MSVC	| GCC, Clang, MSVC |
| ```__PRETTY_FUNCTION__```	| Print name of current function or method with type signature.	| GCC/G++, MinGW (GCC for Windows) and Clang |
| ```__FUNCSIG__``` | Print name of current function with type signature.	| MSVC(aka VC++) |

**Possible values of ```__cplusplus``` macro**

- 199711L	C++98/C++03
- 201103L	C++11
- 201402L	C++14
- 201703L	C++17
- 202002L	C++20
- 202302L	C++23

NOTE: MSVC always reports `199711L` unless you compile with the `/Zc:__cplusplus` flag (use `_MSVC_LANG` instead).


## Macros for detecting platform or operating system

| Operating System | OS Family | Macro | Description |
| --- | --- | --- | --- |
| Windows 32 and 64-bits | WinNT | ```_WIN32``` |	Defined when compiling on Windows with Msvc(Visual C++) or Gcc |
Windows 64 bits	| WinNT | ```_WIN64``` | Windows 64 bits. |
Windows CE | WinCE | ```_WIN32_WCE``` | Windows CE Kernel for embedded systems. |
| MSDOS | | ```MSDOS``` or ```__MSDOS__``` | |	 
| OS2 | | ```OS2``` or ```_OS2``` or ```__OS2__``` | IBM's old OS2 Operating System. | 	 	 	 
| Unix-like | Unix | ```__unix__``` | Check whether OS is unix-like (Linux, BSD variants). NOTE: macOS does **not** define `__unix__`, check `__APPLE__` instead |
| Linux | Unix | ```__linux__``` | Defined when compiling on Linux with Gcc or Clang |
| Android | Unix | ```__ANDROID__``` | Android NDK. (Android also defines `__linux__`) |
| MacOSX and IOS (Darwin Kernel) | Unix | ```__APPLE__```, ```__MACH__```	| |
| BSD | Unix | ```BSD``` (after `#include <sys/param.h>`) | |
| Free BSD | Unix | ```__FreeBSD__``` | |
| Net BSD | Unix | ```__NetBSD__``` | |	 
| QNX | Unix | ```__QNX__``` | |	 
| VxWorks | | ```__VXWORKS__``` or ```__vxworks``` | |	 
| Minix | Unix | ```__minix``` | |	 
| NaCL | | ```__native_client__``` | 
| AsmJS | | ```__asmjs__``` | |	 
       
-----

## Macros for detecting compiler

| Macro | Compiler |
| --- | --- |
| **```__GNUC__```** | GNU C/C++ Compiler. |
| **```__MINGW32__```** |	Mingw or GNU C/C++ Compiler ported for Windows NT. |
| **```__MINGW64__```** |	Mingw or GNU C/C++ Compiler ported for Windows NT - 64 bits only. |
| **```__GFORTRAN__```** |	Fortran / GNU Fortran Compiler |
| **```__clang__```** |	Clang / LLVM Compiler |
| **```_MSC_VER```** | Microsoft Visual Studio Compiler MSVC |
| **```_MANAGED```** or **```__cplusplus_cli```** | Compilation to C++/CLI .NET (CLR) bytecode |
| **```__INTEL_COMPILER```** |	Intel C/C++ Compiler |
| **```__PGI```** or **```__PGIC__```** | Portland Group C/C++ Compiler |
| **```__BORLANDC__```** |	Borland C++ Compiler [DEPRECATED] |
| **```__EMSCRIPTEN__```** | emscripten (asm.js - web assembly) |
| **```__asmjs__```** |	asm.js |
| **```__wasm__```** |	WebAssembly |
| **```__NVCC__```** |	NVCC |
| **```__CLING__```** |	CERN's ROOT Cling C++ Interactive Shell |
 	 
 	 
## C/C++-library Macros
| Macros | Library |
| --- | --- |
| **```__GLIBCXX__```** |	Libstdc++ |
| **```_LIBCPP_VERSION```** | LibC++ |
| **```_MSVC_STL_VERSION```** (or **```_CPPLIB_VER```**) | MSVC C++ Standard Library (STL) |
| **```__GLIBC__```** |	GNU LibC runtime |
| **```__BIONIC__```** |	Bionic LibC runtime. (Android's C-library modified from BSD) |


## Macros for detecting architecture

| Architecture | Macro |
| --- | --- |
| ***Intel x86*** | |
| x86 | **```_M_IX86```** ; ```__INTEL__``` ; **```__i386__```** |
| ***Intel or AMD X86-64 (64 bits)*** | |
| AMD64 (x86-64) | **```__amd64__```** |
| AMD64 | **```__amd64```** |
| AMD64 (MSVC) | **```_M_X64```** ; **```_M_AMD64```** |
| AMD64 | **```__x86_64__```** |
| AMD64 | **```__x86_64```** |
| Intel Itanium 64 bits | **```__IA64__```** , **```__ia64__```** |
| ***ARM Core***	 | |
| ARM (general) | **```__arm__```** , **```_ARM```**, **```_M_ARM```**, **```_M_ARMT```** |
| ARM thumb extension | **```__thumb__```** |
| ARM 2 CPU | **```__ARM_ARCH_2__```** |
| ARM 3 CPU | **```__ARM_ARCH_3__```** |
| ARM 4T CPU | **```__ARM_ARCH_4T__```** |
| ARM 5 CPU | **```__ARM_ARCH_5__```** |
| ARM 5T CPU | **```__ARM_ARCH_5T__```** |
| ARM 6 CPU | **```__ARM_ARCH_6__```** |
| ARM 7 CPU | **```__ARM_ARCH_7__```** |
| ARM 7 CPU | **```__ARM_ARCH_7A__```** |
| ARM 7 CPU | **```__ARM_ARCH_7R__```** |
| ARM 7 CPU | **```__ARM_ARCH_7M__```** |
| ARM 7 CPU | **```__ARM_ARCH_7S__```** |
| ARM64 (ARM 64 bits ) | **```__aarch64__```** ; **```_M_ARM64```** (MSVC) |
| ***PowerPC*** | |	 
| PowerPC core | **```__powerpc```** ; **```__powerpc__```** ; **```__POWERPC__```** ; **```_M_PPC```** |
| PowerPC core | **```__powerpc64__```** |
| ***SuperH*** | |
| SuperH core | **```__sh__```** ; **```__sh1__```** ; **```__sh2__```** ; **```__sh3__```** |
| SuperH core | **```__SH3__```** ; **```__SH4__```** ; **```__SH5__```** |
