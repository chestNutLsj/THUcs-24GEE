A build system is a collection of tools and utilities that automate the process of compiling, linking, and executing source code files in a project. The primary goal of build systems is to manage the complexity of the compilation process and produce a build (executable or binary files) in the end. In C++ (cpp), some common build systems are:

- **GNU Make**: It is a popular build system that uses `Makefile` to define the build process. It checks the dependencies and timestamps of source files to determine which files need to be compiled and linked.
    
    Code example:
    
    ```
    # Makefile
    CXX = g++
    CPPFLAGS = -Wall -std=c++11
    TARGET = HelloWorld
    
    all: $(TARGET)
    
    $(TARGET): main.cpp
        $(CXX) $(CPPFLAGS)main.cpp -o $(TARGET)
    
    clean:
        rm $(TARGET)
    ```
    
- **CMake**: It is a cross-platform build system that focuses on defining project dependencies and managing build environments. CMake generates build files (like Makefiles) for different platforms and allows developers to write source code once and then compile it for different target platforms.
    
    Code example:
    
    ```
    # CMakeLists.txt
    cmake_minimum_required(VERSION 3.10)
    project(HelloWorld)
    
    set(CMAKE_CXX_STANDARD 11)
    
    add_executable(HelloWorld main.cpp)
    ```
    
- **Autotools**: Also known as GNU Build System, consists of the GNU Autoconf, Automake, and Libtool tools that enable developers to create portable software across different Unix-based systems. For a C++ project, you will need to create `configure.ac`, `Makefile.am` files with specific rules, and then run the following commands in the terminal to build the project:
    
    ```
    autoreconf --install
    ./configure
    make
    make install
    ```
    
- **SCons**: This build system uses Python for build scripts, making it more expressive than GNU Make. It can also build for multiple platforms and configurations simultaneously.
    
    Code example:
    
    ```
    # SConstruct
    env = Environment()
    env.Program(target="HelloWorld", source=["main.cpp"])
    ```
    
- **Ninja**: A small and focused build system that takes a list of build targets specified in a human-readable text file and builds them as fast as possible.
    
    Code example:
    
    ```
    # build.ninja
    rule cc
      command = g++ -c $in -o $out
    
    rule link
      command = g++ $in -o $out
    
    build main.o: cc main.cpp
    build HelloWorld: link main.o
      default HelloWorld
    ```
    

These are some of the popular build systems in C++, each with their own syntax and capabilities. While Make is widely used, CMake is a cross-platform build system that generates build files for other build systems like Make or Ninja. Autotools is suitable for creating portable software, SCons leverages Python for its build scripts, and Ninja focuses on fast build times.

## CMake
CMake is a powerful cross-platform build system that generates build files, Makefiles, or workspaces for various platforms and compilers. Unlike the others build systems, CMake does not actually build the project, it only generates the files needed by build tools. CMake is widely used, particularly in C++ projects, for its ease of use and flexibility.

## CMakeLists.txt

CMake uses a file called `CMakeLists.txt` to define settings, source files, libraries, and other configurations. A typical `CMakeLists.txt` for a simple project would look like:

```
cmake_minimum_required(VERSION 3.0)

project(MyProject)

set(SRC_DIR "${CMAKE_CURRENT_LIST_DIR}/src")
set(SOURCES "${SRC_DIR}/main.cpp" "${SRC_DIR}/file1.cpp" "${SRC_DIR}/file2.cpp")

add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_LIST_DIR}/include")

set_target_properties(${PROJECT_NAME} PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
)
```

## Building with CMake

Here is an example of a simple build process using CMake:

- Create a new directory for the build.

```
mkdir build
cd build
```

- Generate build files using CMake.

```
cmake ..
```

In this example, `..` indicates the parent directory where `CMakeLists.txt` is located. The build files will be generated in the `build` directory.

- Build the project using the generated build files.

```
make
```

Or, on Windows with Visual Studio, you may use:

```
msbuild MyProject.sln
```

CMake makes it easy to manage large projects, define custom build configurations, and work with many different compilers and operating systems. Making it a widely chosen tool for managing build systems in C++ projects.

## Makefile
A Makefile is a configuration file used by the `make` utility to automate the process of compiling and linking code in a C++ project. It consists of a set of rules and dependencies that help in building the target executable or library from source code files.

Makefiles help developers save time, reduce errors, and ensure consistency in the build process. They achieve this by specifying the dependencies between different source files, and providing commands that generate output files (such as object files and executables) from input files (such as source code and headers).

## Structure of a Makefile

A typical Makefile has the following structure:

- **Variables**: Define variables to store commonly used values, such as compiler flags, directories, or target names.
- **Rules**: Define how to generate output files from input files using a set of commands. Each rule has a _target_, a set of _prerequisites_, and a _recipe_.
- **Phony targets**: Targets that do not represent actual files in the project but serve as a way to group related rules and invoke them using a single command.

## Example

Consider a basic C++ project with the following directory structure:

```
project/
|-- include/
|   |-- header.h
|-- src/
|   |-- main.cpp
|-- Makefile
```

A simple Makefile for this project could be as follows:

```
# Variables
CXX = g++
CXXFLAGS = -Wall -Iinclude
SRC = src/main.cpp
OBJ = main.o
EXE = my_program

# Rules
$(EXE): $(OBJ)
	$(CXX) $(CXXFLAGS) -o $(EXE) $(OBJ)

$(OBJ): $(SRC)
	$(CXX) $(CXXFLAGS) -c $(SRC)

# Phony targets
.PHONY: clean
clean:
	rm -f $(OBJ) $(EXE)
```

With this Makefile, you can simply run `make` in the terminal to build the project, and `make clean` to remove the output files. The Makefile specifies the dependencies between the source code, object files, and the final executable, as well as the commands to compile and link them.

## Summary

Makefiles provide a powerful way to automate building C++ projects using the `make` utility. They describe the dependencies and commands required to generate output files from source code, saving time and ensuring consistency in the build process.

## Ninja
Ninja is a small build system with a focus on speed. It is designed to handle large projects by generating build files that implement the minimal amount of work necessary to build the code. This results in faster build times, especially for large codebases. Ninja is often used in conjunction with other build systems like CMake, which can generate Ninja build files for you.

Ninja build files are typically named `build.ninja` and contain rules, build statements, and variable declarations. Here’s a simple example of a Ninja build file for a C++ project:

```
# Variable declarations
cxx = g++
cflags = -Wall -Wextra -std=c++17

# Rule for compiling the C++ files
rule cxx_compile
  command = $cxx $cflags -c $in -o $out

# Build statements for the source files
build main.o: cxx_compile main.cpp
build foo.o: cxx_compile foo.cpp

# Rule for linking the object files
rule link
  command = $cxx $in -o $out

# Build statement for the final executable
build my_program: link main.o foo.o
```

To build the project using this `build.ninja` file, simply run `ninja` in the terminal:

```
$ ninja
```

This will build the `my_program` executable by first compiling the `main.cpp` and `foo.cpp` files into object files, and then linking them together.