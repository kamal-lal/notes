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
CXX    := g++
OPT    := -O0
CFLAGS := -Wall -Werror -g $(OPT)

.PHONY: all clean run

all: myapp

myapp: main.o tools.o utilities.o
    g++ main.o tools.o utilities.o -o myapp

main.o: main.cpp
    g++ -c $(CFLAGS) main.cpp -o main.o

tools.o: tools.cpp
    g++ -c $(CFLAGS) tools.cpp -o tools.o

utilities.o: utilities.cpp
    g++ -c $(CFLAGS) utilities.cpp -o utilities.o

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
CXX    := g++
OPT    := -O0
CFLAGS := -Wall -Werror -g $(OPT)

.PHONY: all clean run

all: myapp

myapp: main.o tools.o utilities.o
    g++ $^ -o $@

main.o: main.cpp
    g++ -c $(CFLAGS) $< -o $@

tools.o: tools.cpp
    g++ -c $(CFLAGS) $< -o $@ 

utilities.o: utilities.cpp 
    g++ -c $(CFLAGS) $< -o $@

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
CXX    := g++
OPT    := -O0
CFLAGS := -Wall -Werror -g $(OPT)

OBJECTS := main.o tools.o utilities.o

.PHONY: all clean run

all: myapp

myapp: $(OBJECTS)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $< -o $@

clean: 
    rm -rf *.o myapp

run:
    ./myapp
```

## More MAKE concepts - File Name Functions

### 1. Find all files with certain extension

Example:
```makefile
SOURCES := $(wildcard *.cpp)
```

### 2. Add Prefix

Example:
```makefile
SOURCES := main.cpp file1.cpp file2.cpp
SOURCES_WITH_PATH := $(addprefix src/, $(SOURCES))
```

## More MAKE concepts - String Manipulation Functions

### 1. Pattern substitution

Basic format: 
```makefile
$(patsubst pattern, replacement, text)
```

Example:
```makefile
SOURCES := main.cpp file1.cpp file2.cpp
OBJECTS := $(patsubst %.cpp, %.o, $(SOURCES))
```

Shorthand format: `$(var:pattern=replacement)`

Example:
```makefile
SOURCES := main.cpp file1.cpp file2.cpp
OBJECTS := $(SOURCES:.cpp=.o)
```

## More MAKE concepts - Other Functions

### 1. SHELL function

Example: 
```makefile
SOURCES := $(shell find *.cpp)
```

### 2. FOREACH function

Basic format: 
```makefile
$(foreach var,list,text)
```

The first two arguments, _var_ and _list_, are expanded before anything else is done; note that the last argument, _text_, is not expanded at the same time. Then for each word of the expanded value of _list_, the variable named by the expanded value of _var_ is set to that word, and _text_ is expanded.

Example: 
```makefile
dirs  := src1 src2 src3
files := $(foreach dir, $(dirs), $(wildcard $(dir)/*))
```

## Example - 4 (With automatic files detection)

Putting functions to work.  
Assuming all CPP source files and header files are in same folder.  
Also the build process is done in same folder.

```makefile
CXX    := g++
OPT    := -O0
CFLAGS := -Wall -Werror -g
TARGET := myapp

SOURCES := $(wildcard *.cpp)
OBJECTS := $(patsubst %.cpp,%.o,$(SOURCES))

# Or, written in shorthand format as:
#    OBJECTS := $(SOURCES:.cpp=.o)

.PHONY: all clean run

all: $(TARGET)

$(TARGET): $(OBJECTS)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $(OPT) $< -o $@

clean: 
    rm -rf *.o $(TARGET)

run:
    ./$(TARGET)
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
CXX      := g++
OPT      := -O0
CFLAGS   := -Wall -Werror -g
DEPFLAGS := -MMD
TARGET   := myapp

SOURCES := $(wildcard *.cpp)
OBJECTS := $(patsubst %.cpp, %.o, $(SOURCES))
DEPS    := $(patsubst %.cpp, %.d, $(SOURCES))

.PHONY: all clean run

all: $(TARGET)

$(TARGET): $(OBJECTS)
    g++ $^ -o $@

%.o: %.cpp
    g++ -c $(CFLAGS) $(OPT) $(DEPFLAGS) $< -o $@

clean: 
    rm -rf *.o *.d $(TARGET)

run:
    ./$(TARGET)

-include $(DEPS)
```

## Example - 6 (Project with structured sub-folders)

```
# ~~~~ Folder Before ~~~~ #
- project root
  	|- inc/
  	|	|- header1.h
  	|	|- header2.h
  	|	  ....
  	|- src/
  	|	|- source1.cpp
  	|	|- source2.cpp
  	|	  ....
  	|- Makefile


# ~~~~ Folder After ~~~~ #
- project root
  	|- obj/
  	|	|- obj1.o 
  	|	|- obj1.d
  	|	|- obj2.o 
  	|	|- obj2.d
  	|	  ....
  	|- inc/
  	|	|- header1.h
  	|	|- header2.h
  	|	  ....
  	|- src/
  	|	|- source1.cpp
  	|	|- source2.cpp
  	|	  ....
  	|- Makefile
  	|- myapp
```

Makefile for this folder structure:

```makefile
# Compiler and flags
CXX      := g++
OPT      := -O0 -g
CFLAGS   := -Wall -Werror
DEPFLAGS := -MMD

# Folders
SRCDIR := src
INCDIR := inc
OBJDIR := obj

# Executable binary
TARGET := myapp

# File names
SOURCES := $(wildcard $(SRCDIR)/*.cpp)
OBJECTS := $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(SOURCES))
DEPS    := $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.d, $(SOURCES))

# Default target
all: $(TARGET)

# Build target
$(TARGET): $(OBJECTS)
	g++ $^ -o $@

# Compile source files
$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
	@mkdir -p $(OBJDIR)
	g++ -c $(CFLAGS) $(OPT) $(DEPFLAGS) -I$(INCDIR) $< -o $@

# Misc phony targets
clean: 
	rm -rf $(TARGET) $(OBJDIR)

run:
	./$(TARGET)

# Include dependency files
-include $(DEPS)

.PHONY: all clean run
```

## Example - 7 (Project with structured sub-folders and multiple builds)

```
# ~~~~ Folder Before ~~~~ #
- project root
 	|- inc/
 	|	|- header1.h
 	|	|- header2.h
 	|	  ....
 	|- src/
 	|	|- source1.cpp
 	|	|- source2.cpp
 	|	  ....
 	|- Makefile


# ~~~~ Folder After ~~~~ #
- project root
 	|- bin/
 	|	|- debug/
 	|	|	|- myapp
 	|	|- release/
 	|	|	|- myapp
 	|- obj/
 	|	|- debug/
 	|	|	|- objd1.o
 	|	|	|- objd1.d
 	|	|	|- objd2.o
 	|	|	|- objd2.d
 	|	|	  ....
 	|	|- release/
 	|	|	|- objr1.o
 	|	|	|- objr1.d
 	|	|	|- objr2.o
 	|	|	|- objr2.d
 	|	|	  ....
 	|- inc/
 	|	|- header1.h
 	|	|- header2.h
 	|	  ....
 	|- src/
 	|	|- source1.cpp
 	|	|- source2.cpp
 	|	  ....
 	|- Makefile
```

Makefile for this purpose:

```makefile
# Compiler and flags
CXX           := g++
DEPFLAGS      := -MMD
CLAGS         := -Wall -Werror
DEBUG_FLAGS   := -O0 -g -DDEBUG
RELEASE_FLAGS := -O2 -DNDEBUG

# Folders
SRCDIR         := src
INCDIR         := inc
OBJDIR         := obj
BINDIR         := bin
DEBUG_OBJDIR   := $(OBJDIR)/debug
DEBUG_BINDIR   := $(BINDIR)/debug
RELEASE_OBJDIR := $(OBJDIR)/release
RELEASE_BINDIR := $(BINDIR)/release

# Executable binaries
DEBUG_TARGET   := $(DEBUG_BINDIR)/myapp
RELEASE_TARGET := $(RELEASE_BINDIR)/myapp

# File names
SOURCES      := $(wildcard $(SRCDIR)/*.cpp)
DEBUG_OBJS   := $(patsubst $(SRCDIR)/%.cpp, $(DEBUG_OBJDIR)/%.o, $(SOURCES))
DEBUG_DEPS   := $(patsubst %.o, %.d, $(DEBUG_OBJS))
RELEASE_OBJS := $(patsubst $(SRCDIR)/%.cpp, $(RELEASE_OBJDIR)/%.o, $(SOURCES))
RELEASE_DEPS := $(patsubst %.o, %.d, $(RELEASE_OBJS))

# Default target
debug: $(DEBUG_TARGET)

release: $(RELEASE_TARGET)

all: debug release

# Build debug target
$(DEBUG_TARGET): $(DEBUG_OBJS)
	@mkdir -p $(DEBUG_BINDIR)
	g++ $^ -o $@

# Compile source files (into debug objs)
$(DEBUG_OBJDIR)/%.o: $(SRCDIR)/%.cpp
	@mkdir -p $(DEBUG_OBJDIR)
	g++ -c $(CFLAGS) -I$(INCDIR) $(DEPFLAGS) $(DEBUG_FLAGS) $< -o $@

# Build release target
$(RELEASE_TARGET): $(RELEASE_OBJS)
	@mkdir -p $(RELEASE_BINDIR)
	g++ $^ -o $@

# Compile source files (into release objs)
$(RELEASE_OBJDIR)/%.o: $(SRCDIR)/%.cpp
	@mkdir -p $(RELEASE_OBJDIR)
	g++ -c $(CFLAGS) -I$(INCDIR) $(DEPFLAGS) $(RELEASE_FLAGS) $< -o $@

# Misc phony targets
clean:
	rm -rf $(DEBUG_TARGET) $(DEBUG_OBJDIR)/*
	rm -rf $(RELEASE_TARGET) $(RELEASE_OBJDIR)/*

cleanall: 
	rm -rf $(BINDIR) $(OBJDIR)

rundebug:
	./$(DEBUG_TARGET)

runrelease:
	./$(RELEASE_TARGET)

# Include dependency files
-include $(DEBUG_DEPS)
-include $(RELEASE_DEPS)

.PHONY: all clean cleanall rundebug runrelease debug release
```
