# **The MAKE Build System**

# MAKE Basics
- The commands are written in a file named `Makefile`. 
- Easy to relate with 'Baking' process. 

General syntax is as follow:
```makefile
target: prerequisites
    recipe
```

- The `make` system looks if the 'target' file already exists. 
- If it doesn't exist, `make` makes the target using the 'prerequisites', following the recepie. 
- If it already exist, `make` looks at the timestamp of the 'prerequisite' file. If it is updated compared to last run, it executes the recepie again. 
- Note that the 'prerequisite' my be another 'target' and not a file (a Phony target). 

A simple `Makefile` to create an executable from a single C++ source file will look as follows:

```makefile
myapp: main.cpp
    g++ main.cpp -o myapp
```

To run `make`, just run the command `make` from the directory where `Makefile` exists. 

```bash
$ make
```

Alternatively, we can explicitly ask `make` to build a target.

```bash
$ make myapp
```

## Example - 1 (Basic)
Consider a C++ project having multiple source files. 

- `main.cpp`
- `tools.cpp`
- `utilities.cpp`

Assuming the `main.cpp` file need `tools.cpp` and `utilities.cpp` files, following example shows `Makefile` that could be used for this project.

```makefile
# Comments in Makefile are written with '#' symbol. 

.PHONY: all clean run

all: myapp

myapp: main.o tools.o utilities.o
    g++ main.o tools.o utilities.o -o myapp

main.o: main.cpp
    g++ -c -Wall -Werror -g -O0 main.cpp -o main.o

tools.o: tools.cpp
    g++ -c -Wall -Werror -g -O0 tools.cpp -o tools.o

utilities.o: utilities.cpp
    g++ -c -Wall -Werror -g -O0 utilities.cpp -o utilities.o

clean: 
    rm -rf *.o myapp

run:
    ./myapp
```

To create the executable binary:

```bash
$ make
```

Or 

```bash
$ make all
```

To run the created binary, run:

```bash
$ make run
```

To delete the binary file and object files that were created, run:

```bash
$ make clean
```

Note: The `all`, `clean`, `run` are usual naming conventions for this purpose. 

## Example - 2 (Editing made easy with Variables)

Shows the use of variables.

```makefile
CC=g++
OPT=-O0
CFLAGS=-Wall -Werror -g $(OPT)

.PHONY: all clean run

all: myapp

myapp: main.o tools.o utilities.o
    g++ main.o tools.o utilities.o -o myapp

main.o: main.cpp
    g++ -c $(CFLAGS) $(OPT) main.cpp -o main.o

tools.o: tools.cpp
    g++ -c $(CFLAGS) $(OPT) tools.cpp -o tools.o

utilities.o: utilities.cpp
    g++ -c $(CFLAGS) $(OPT) utilities.cpp -o utilities.o

clean: 
    rm -rf *.o myapp

run:
    ./myapp
```

Some special variables (Automatic variables):
- `$@` - Name of target
- `$^` - All prerequisites 
- `$<` - First prerequisites

Rewriting above `Makefile` using automatic variables:

```makefile
CC=g++
OPT=-O0
CFLAGS=-Wall -Werror -g $(OPT)

.PHONY: all clean run

all: myapp

myapp: main.o tools.o utilities.o
    g++ $^ -o $@

main.o: main.cpp
    g++ -c $(CFLAGS) $(OPT) $^ -o $@

tools.o: tools.cpp
    g++ -c $(CFLAGS) $(OPT) $^ -o $@ 

utilities.o: utilities.cpp 
    g++ -c $(CFLAGS) $(OPT) $^ -o $@

clean: 
    rm -rf *.o myapp

run:
    ./myapp
```

## Example - 3 (Avoiding repetition)

### Patterns

- A pattern rule contains the character '`%`'
- The target is a pattern for matching file names
- The substring that the '`%`' matches is called the stem.
- '`%`' in a prerequisite of a pattern rule stands for the same stem that was matched by the '`%`' in the target. 

```makefile
CC=g++
OPT=-O0
CFLAGS=-Wall -Werror -g $(OPT)

OBJ_FILES=main.o tools.o utilities.o

.PHONY: all clean run

all: myapp

myapp: $(OBJ_FILES)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $(OPT) $^ -o $@

clean: 
    rm -rf *.o myapp

run:
    ./myapp
```

## More MAKE concepts - File Name Functions

### 1. Find all files with certain extension

Example:
```makefile
SRC_FILES=$(wildcard *.cpp)
```

### 2. Add Prefix

Example:
```makefile
SRC_FILES=main.cpp file1.cpp file2.cpp
SRC_FILES_WITH_PATH=$(addprefix src/,$(SRC_FILES))
```

## More MAKE concepts - String Manipulation Functions

### 1. Pattern substitution

Basic format: 
```makefile
$(patsubst pattern,replacement,text)
```

Example:
```makefile
SRC_FILES=main.cpp file1.cpp file2.cpp
OBJ_FILES=$(patsubst %.cpp, %.o, $(SRC_FILES))
```

Shorthand format: `$(var:pattern=replacement)`

Example:
```makefile
SRC_FILES=main.cpp file1.cpp file2.cpp
OBJ_FILES=$(SRC_FILES:.cpp=.o)
```

## More MAKE concepts - Other Functions

### 1. SHELL function

Example: 
```makefile
SRC_FILES=$(shell find *.cpp)
```

### 2. FOREACH function

Basic format: 
```makefile
$(foreach var,list,text)
```

The first two arguments, _var_ and _list_, are expanded before anything else is done; note that the last argument, _text_, is not expanded at the same time. Then for each word of the expanded value of _list_, the variable named by the expanded value of _var_ is set to that word, and _text_ is expanded.

Example: 
```makefile
dirs := src1 src2 src3
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
```

## Example - 4 (With automatic files detection)

Putting functions to work.  
Assuming all CPP source files and header files are in same folder.  
Also the build process is done in same folder.

```makefile
CC=g++
OPT=-O0
CFLAGS=-Wall -Werror -g
TARGET_BIN=myapp

SRC_FILES=$(wildcard *.cpp)
OBJ_FILES=$(patsubst %.cpp,%.o,$(SRC_FILES))
# Or, written in shorthand format as:
#    OBJ_FILES=$(SRC_FILES:.cpp=.o)

.PHONY: all clean run

all: $(TARGET_BIN)

$(TARGET_BIN): $(OBJ_FILES)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $(OPT) $^ -o $@

clean: 
    rm -rf *.o $(TARGET_BIN)

run:
    ./$(TARGET_BIN)
```

## Example - 5 (Project with Header files)

All earlier examples were considering multiple C++ source files. But in any C++ projects, header files will also be present. Now consider C++ project having following files:

- `main.cpp`
- `tools.cpp`
- `utilities.cpp`
- `tools.h`
- `utilities.h`
- `constants.h`

Assume that multiple header files are also incuded in differnt source files (in any order).  
Usually, we recopile a source file if the file itself is changed (timestamp updated). However, now we must recompile the source files if any header files included in it is changed.  

To do this, we can leverage the `g++` flags that can generate dependency files and use those files in the `make` process. 

```makefile
CC=g++
OPT=-O0
CFLAGS=-Wall -Werror -g
DEPFLAGS=-MMD
TARGET_BIN=myapp

SRC_FILES=$(wildcard *.cpp)
OBJ_FILES=$(patsubst %.cpp,%.o,$(SRC_FILES))
DEP_FILES=$(patsubst %.cpp,%.d,$(SRC_FILES))

-include $(DEP_FILES)

.PHONY: all clean run

all: $(TARGET_BIN)

$(TARGET_BIN): $(OBJ_FILES)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $(OPT) $(DEPFLAGS) $^ -o $@

clean: 
    rm -rf *.o $(TARGET_BIN)

run:
    ./$(TARGET_BIN)
```
