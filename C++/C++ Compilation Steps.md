# **C/C++ compilation process**

# Basics
Command to create a executable binay from a C++ source file (eg. `main.cpp`):

```
g++ main.cpp
```

This creates a binary file with name `a.out` (in Linux) or `a.exe` (in Windows).

To create a output file (executable binary) with a specific name such as `myapp` (in Linux) or `myapp.exe` (in Windows):

```
g++ main.cpp -o myapp
```

If multiple C++ source files exists:

```
g++ main.cpp file1.cpp file2.cpp -o myapp
```

---

# Behind the scenes
The C/C++ compilation is a 4 step process: 

1. Preprocessing
1. Compilation
1. Assembly
1. Linking

---

## 1. Preprocessing
- Removes comments
- Runs preprocessor directives
    * Copy Header files (`#include`)
    * Runs macros (`#define`)

To make `g++` just do preprocessing and generate an output (usually with `.i` extension) use `-E` flag. 

```
g++ -E main.cpp -o main.i
```

**Preprocessor directives examples**

- Defining macro constant:
```cpp
#define INCH_TO_CM 2.54
```

- Function-like macro:
```cpp
#define CIRCLE_AREA(r) (3.14*r*r)
```

- Conditionally including/excluding code:
```cpp
#define LOGGING 0

#if LOGGING
#define SHOW(x) std::cout<<x<<std::endl
#else
#define SHOW(x)
#endif
```

In above case, the macro `LOGGING` can be modified in source file to have definition `1` or `0` to get corresponding result. 

As an alternative, without modifying the source file, the value of macro can be defined by providing `-D` flag to `g++`. 

```cpp
#if LOGGING
#define SHOW(x) std::cout<<x<<std::endl
#else
#define SHOW(x)
#endif
```

Above code along with below command gives same result

```
g++ main.cpp -DLOGGING=0 -o myapp
```

- Header guards:
```cpp
#ifndef UTIL_H
#define UTIL_H

int sum(int, int);

#endif /* UTIL_H */
```

## 2. Compilation
- Converts C++ source file to assembly language
- This is still human readable
- The assembly language depends on the architecture (eg. x86, x86_64, ARM, RISC-V etc.) for which we are compiling

To stop `g++` at the _'compilation'_ step and generate an output (usually with `.s` extension) use `-S` flag. 

```
g++ -S main.cpp -o main.s
g++ -S file1.cpp -o file1.s
g++ -S file2.cpp -o file2.s
```

## 3. Assembly
- Converts C++ source file to machine code (byte code)
- Not human readable
- The result is 'Object' files. 
- Each source file gets turned into separate object files 

To stop `g++` at the _'assembly'_ step and generate an output (with `.o` (Linux) or `.obj` (Windows) extension) use `-c` flag. 

```
g++ -c main.cpp -o main.o
g++ -c file1.cpp -o file1.o
g++ -c file2.cpp -o file2.o
```

Object files could be distriuted and re-used. Libraries are usually done this way. If multiple applications in a machine use the same library, a '_shared object_' could also be used (`.so` files).

## 4. Linking
- Final step, in which all _object_ files along with libraries (local and external) are combined together (in other words, _Linked_ together) for form executable binary. 
- No separate flags, just `g++`. (can accept any of the earlier intermediate files as input)

(TODO: Library creation and Linking library)
```
g++ main.o file1.o file2.o -o myapp
```

Or

```
g++ main.cpp file1.cpp file2.cpp -o myapp
```

# Common `g++` flags

- Save all intermediate files
```
g++ -save-temps main.cpp file1.cpp file2.cpp -o myapp
```

- Looking for header files in folders (eg. MyHeaders) other than 'default' ones. Also known as 'include' folders. 
```
g++ -IMyHeaders main.cpp file1.cpp file2.cpp -o myapp
```
Compiler knows systems default Include folder (eg. `/usr/include` in Linux). Also the 'current directory' is present by default (`-I.`). 

- Show all warnings
```
g++ -Wall main.cpp file1.cpp file2.cpp -o myapp
```

- Convert warnings into errors (compilation fails)
```
g++ -Wall -Werror main.cpp file1.cpp file2.cpp -o myapp
```

- Get verbose output during compilation
```
g++ -v main.cpp file1.cpp file2.cpp -o myapp
```

- Set required C++ standard
```
g++ -std=c++20 main.cpp file1.cpp file2.cpp -o myapp
```
Other options include `-std=c++98`, `-std=c++03`, `-std=c++11`, `-std=c++14`, `-std=c++17`, `-std=c++23` etc. 

- Produce output with debugging symbols in it.
```
g++ -g main.cpp file1.cpp file2.cpp -o myapp
```

- Produce output with high optimisation
```
g++ -O3 main.cpp file1.cpp file2.cpp -o myapp
```
Other optimisation flags:

'`-O1`' - Low optimisation

'`-O2`' - Medium optimisation

'`-O3`' - High optimisation

'`-Ofast`' - Optimised for speed

'`-Og`' - Optimise for debugging

'`-Os`' - Optimised for size
