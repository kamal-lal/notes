# **The MAKE Build System**

# Basics
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

## More realistic (manual) example - 1
Consider a C++ project having multiple source files and header files. 

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

## More realistic (manual) example - 2

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

## Patterns

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

## Functions

### 1. Pattern substitution

Basic format: 
```makefile
$(patsubst pattern,replacement,text)
```

An example:
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

### 2. Add Prefix

Example:
```makefile
SRC_FILES=main.cpp file1.cpp file2.cpp
SRC_FILES_WITH_PATH=$(addprefix src/,$(SRC_FILES))
```


