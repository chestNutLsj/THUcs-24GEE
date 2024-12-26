
## tldr
```shell
❯ tldr gcc

  gcc

  Preprocess and compile C and C++ source files, then assemble and link them together.
  More information: https://gcc.gnu.org.

  - Compile multiple source files into an executable:
    gcc path/to/source1.c path/to/source2.c ... -o path/to/output_executable

  - Show common warnings, debug symbols in output, and optimize without affecting debugging:
    gcc path/to/source.c -Wall -g -Og -o path/to/output_executable

  - Include libraries from a different path:
    gcc path/to/source.c -o path/to/output_executable -Ipath/to/header -Lpath/to/library -llibrary_name

  - Compile source code into Assembler instructions:
    gcc -S path/to/source.c

  - Compile source code into an object file without linking:
    gcc -c path/to/source.c

```

## 最简单的编译可执行文件
```shell
gcc -o hello hello.c
```

参数解释：
- `-o`：输出可执行文件，后紧跟的是可执行文件的文件名。

## 优化级别
```
gcc -Og -o p p1.c p2.c
```

