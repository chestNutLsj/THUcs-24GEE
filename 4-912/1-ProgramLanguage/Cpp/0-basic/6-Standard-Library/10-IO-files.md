
C++ provides the following classes to perform output and input of characters to/from files:

*   **[ofstream](https://cplusplus.com/ofstream):** Stream class to write on files
*   **[ifstream](https://cplusplus.com/ifstream):** Stream class to read from files
*   **[fstream](https://cplusplus.com/fstream):** Stream class to both read and write from/to files.

  
These classes are derived directly or indirectly from the classes `istream` and `ostream`. We have already used objects whose types were these classes: `cin` is an object of class `istream` and `cout` is an object of class `ostream`. Therefore, we have already been using classes that are related to our file streams. And in fact, we can use our file streams the same way we are already used to use `cin` and `cout`, with the only difference that we have to associate these streams with physical files. Let's see an example:

```cpp
// basic file operations
#include <iostream> 
#include <fstream> 
using namespace std;

int main () {
  ofstream myfile;
  myfile.open ("example. txt");
  myfile << "Writing this to a file.\n";
  myfile.close ();
  return 0;
}
```

This code creates a file called `example.txt` and inserts a sentence into it in the same way we are used to do with `cout`, but using the file stream `myfile` instead.

But let's go step by step:

## Open a file

The first operation generally performed on an object of one of these classes is to associate it to a real file. This procedure is known as to _open a file_. An open file is represented within a program by a _stream_ (i.e., an object of one of these classes; in the previous example, this was `myfile`) and any input or output operation performed on this stream object will be applied to the physical file associated to it. （文件 IO 类的对象首先要关联到实际文件，这一步称为*打开文件*，打开的文件在程序中以流的形式存在，并且对该流对象执行的任何输入或输出操作都将应用于与之关联的物理文件）

In order to open a file with a stream object we use its member function `open`:

`open (filename, mode);`

Where `filename` is a string representing the name of the file to be opened, and `mode` is an optional parameter with a combination of the following flags:

| mode         | description                                                                                                                      | 
| ------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| ios:: in     | Open for input operations.                                                                                                       |
| ios:: out    | Open for output operations.                                                                                                      |
| ios:: binary | Open in binary mode.                                                                                                             |
| ios:: ate    | Set the initial position at the end of the file. If this flag is not set, the initial position is the beginning of the file.     |
| ios:: app    | All output operations are performed at the end of the file, appending the content to the current content of the file.            |
| ios:: trunc  | If the file is opened for output operations and it already existed, its previous content is deleted and replaced by the new one. |

All these flags can be combined using the bitwise operator OR (`|`). For example, if we want to open the file `example.bin` in binary mode to add data we could do it by the following call to member function `open`:

```
ofstream myfile;
myfile.open ("example.bin", ios::out | ios::app | ios::binary);
```

Each of the `open` member functions of classes `ofstream`, `ifstream` and `fstream` has a default mode that is used if the file is opened without a second argument:

| class    | default mode parameter |
|----------|------------------------|
| ofstream | ios:: out               |
| ifstream | ios:: in                |
| fstream  | ios:: in | ios:: out    |

For `ifstream` and `ofstream` classes, `ios::in` and `ios::out` are automatically and respectively assumed, even if a mode that does not include them is passed as second argument to the `open` member function (the flags are combined). （对于 `ifstream` 和 `ofstream` 类而言，标记 `ios::in` 和 `ios:out` 时自动设定的，即使在 `open` 成员函数的第二个参数中不包含它们）

For `fstream`, the default value is only applied if the function is called without specifying any value for the mode parameter. If the function is called with any value in that parameter the default mode is overridden, not combined.（对于 `fstream` 对象，仅当调用函数的 mode 参数没有指定任何值时才使用默认值，否则就会被覆盖）

File streams opened in _binary mode_ perform input and output operations independently of any format considerations. Non-binary files are known as _text files_, and some translations may occur due to formatting of some special characters (like newline and carriage return characters).（以二进制模式打开的文件流执行输入和输出操作，不受任何格式因素的影响。以非二进制模式打开的文件又称文本文件，对于一些特殊字符如换行符和回车符有特殊的翻译操作）

Since the first task that is performed on a file stream is generally to open a file, these three classes include a constructor that automatically calls the `open` member function and has the exact same parameters as this member. Therefore, we could also have declared the previous `myfile` object and conduct the same opening operation in our previous example by writing:（由于对文件流执行的第一个任务通常是打开文件，因此这三个类都包括一个构造函数以自动调用 `open` 成员函数。因此，可以在声明这三个类的对象时，直接初始化打开的文件名和操作模式：）

```
ofstream myfile ("example.bin", ios::out | ios::app | ios::binary);
```

Combining object construction and stream opening in a single statement. Both forms to open a file are valid and equivalent.

To check if a file stream was successful opening a file, you can do it by calling to member `is_open`. This member function returns a `bool` value of `true` in the case that indeed the stream object is associated with an open file, or `false` otherwise:

```
if (myfile.is_open()) { /* ok, proceed with output */ }
```

## Closing a file

When we are finished with our input and output operations on a file we shall close it so that the operating system is notified and its resources become available again. For that, we call the stream's member function `close`. This member function takes flushes the associated buffers and closes the file:  （`close` 函数会刷新关联的缓冲区并关闭文件）

```cpp
myfile.close();
```

Once this member function is called, the stream object can be re-used to open another file, and the file is available again to be opened by other processes.

In case that an object is destroyed while still associated with an open file, the destructor automatically calls the member function `close`.（如果文件对象在仍与打开的文件相关联时被销毁，析构函数会自动调用 `close` 成员函数）

## Text files

Text file streams are those where the `ios::binary` flag is not included in their opening mode. These files are designed to store text and thus all values that are input or output from/to them can suffer some formatting transformations, which do not necessarily correspond to their literal binary value.

Writing operations on text files are performed in the same way we operated with `cout`:

```cpp
// writing on a text file
#include "iostream"
#include "fstream"

using namespace std;

int main() {
    ofstream myfile("example2.txt");
    if (myfile.is_open()) {
        myfile << "this is a line.\n";
        myfile << "this is another line.\n";
        myfile.close();
    } else cout << "Unable to open file";
    return 0;
}
```

Reading from a file can also be performed in the same way that we did with `cin`:

```cpp
// reading a text file
#include <iostream> 
#include <fstream> 
#include <string> 
using namespace std;

int main () {
  string line;
  ifstream myfile ("example2. txt");
  if (myfile.is_open ())
  {
    while ( getline (myfile, line) )
    {
      cout << line << '\n';
    }
    myfile.close ();
  }

  else cout << "Unable to open file"; 

  return 0;
}
```

This last example reads a text file and prints out its content on the screen. We have created a while loop that reads the file line by line, using [getline](https://cplusplus.com/getline). 

>[! note] getline ()
The value returned by [getline](https://cplusplus.com/getline) is a reference to the stream object itself, which when evaluated as a boolean expression (as in this while-loop) is `true` if the stream is ready for more operations, and `false` if either the end of the file has been reached or if some other error occurred.  

## Checking state flags

The following member functions exist to check for specific states of a stream (all of them return a `bool` value):

| function | description                                                                                                                                                                                                                              |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bad()`  | Returns `true` if a reading or writing operation fails. For example, in the case that we try to write to a file that is not open for writing or if the device where we try to write has no space left.                                   |
| `fail()` | Returns `true` in the same cases as `bad()`, but also in the case that a format error happens, like when an alphabetical character is extracted when we are trying to read an integer number.                                            |
| `eof()`  | Returns `true` if a file open for reading has reached the end.                                                                                                                                                                           |
| `good()` | It is the most generic state flag: it returns `false` in the same cases in which calling any of the previous functions would return `true`. Note that `good` and `bad` are not exact opposites (`good` checks more state flags at once). |
| `clear`  | Reset the state flags.                                                                                                                                                                                                                                         |
 
## get and put stream positioning

All i/o streams objects keep internally -at least- one internal position: （所有 IO 流对象在内部至少保留一个内部位置）
-  `ifstream`, like `istream`, keeps an internal _get position_ with the location of the element to be read in the next input operation.（`ifstream` 在下一个输入操作中保留要读取的元素位置的*内部获取位置*）
- `ofstream`, like `ostream`, keeps an internal _put position_ with the location where the next element has to be written.（`ofstream` 必须在写入下一个元素的位置上保留一个*内部放置位置*）
- Finally, `fstream`, keeps both, the _get_ and the _put position_, like `iostream`.（`fstream` 保留 *get* 和 *put* 这二者的内部位置）

These internal stream positions point to the locations within the stream where the next reading or writing operation is performed. These positions can be observed and modified using the following member functions: （这些内部流位置指向流中执行下一个读取或写入操作的位置。可以使用以下成员函数访问和修改这些位置）

- `tellg ()` and `tellp () `: These two member functions with no parameters return a value of the member type ` streampos `, which is a type representing the current _get position_ (in the case of ` tellg `) or the _put position_ (in the case of ` tellp `). （这两个无参成员函数返回成员 `streampos` 类型的值，该值分别表示当前获取位置和放置位置）
- `seekg ()` and `seekp ()`: These functions allow to change the location of the _get_ and _put positions_. Both functions are overloaded with two different prototypes. （这两个函数允许改变获取和放置位置的地点，且这两个函数都用两个不同的原型重载） ^9c9960
	- The two forms are:
		1. `seekg ( position );` ` seekp ( position ); ` : Using this prototype, the stream pointer is changed to the absolute position `position` (counting from the beginning of the file). The type for this parameter is `streampos`, which is the same type as returned by functions `tellg` and `tellp`.（使用此原型，将流指针更改为从文件开头计数的绝对位置，其参数类型为 `streampos`）
		2. `seekg ( offset, direction );` `seekp ( offset, direction );`: Using this prototype, the _get_ or _put position_ is set to an offset value relative to some specific point determined by the parameter `direction`. （使用此原型，将获取和放置位置更改为相对于参数 `direction` 确定的某个特定点的偏移值）` offset ` is of type ` streamoff `. And ` direction ` is of type ` seekdir `, which is an _enumerated type_ that determines the point from where offset is counted from, and that can take any of the following values:
			- `ios::cur`: offset counted from the current position
			- `ios::end`: offset counted from the end of the stream

The following example uses the member functions we have just seen to obtain the size of a file:

```cpp
// obtaining file size
#include <iostream> 
#include <fstream> 
using namespace std;

int main () {
  streampos begin, end;
  ifstream myfile ("example.bin", ios::binary);
  begin = myfile.tellg ();
  myfile. seekg (0, ios::end);
  end = myfile.tellg ();
  myfile.close ();
  cout << "size is: " << (end-begin) << " bytes.\n";
  return 0;
}
```

Notice the type we have used for variables `begin` and `end`:

```cpp
streampos begin, end;
```

`streampos` is a specific type used for buffer and file positioning and is the type returned by `file.tellg()`. Values of this type can safely be subtracted from other values of the same type, and can also be converted to an integer type large enough to contain the size of the file. （`streampos` 是用于缓冲区和文件定位的特定类型，可以安全地从相同类型的其它值中减去此类型的值，也可以转换为足够大的整数类型以包含文件大小）

These stream positioning functions use two particular types: `streampos` and `streamoff`. These types are also defined as member types of the stream class:（这些流定位函数中有两种特定类型：`streampos` 和 `streamoff`，这些类型也被定义为流类的成员类型）

| Type      | Member type   | Description                                                                                                                   |
|-----------|---------------|-------------------------------------------------------------------------------------------------------------------------------|
| streampos | ios:: pos_type | Defined as fpos&lt; mbstate_t&gt;. It can be converted to/from streamoff and can be added or subtracted values of these types. |
| streamoff | ios:: off_type | It is an alias of one of the fundamental integral types (such as int or long long).                                           |

Each of the member types above is an alias of its non-member equivalent (they are the exact same type). It does not matter which one is used. The member types are more generic, because they are the same on all stream objects (even on streams using exotic types of characters), but the non-member types are widely used in existing code for historical reasons.  （每个成员类型都是其非成员等效项的别名，使用哪个并不重要。成员类型更加通用，因为在所有流对象上都是相同的，历史原因中非成员类型也被广泛使用。）

## Binary files

For binary files, reading and writing data with the extraction and insertion operators (`<<` and `>>`) and functions like `getline` is not efficient, since we do not need to format any data and data is likely not formatted in lines.（对于二进制文件，使用提取符、插入符、类似 `getline` 等读取函数的效率不高，因为不需要对二进制数据进行格式化）

File streams include two member functions specifically designed to read and write binary data sequentially: `write` and `read`. The first one (`write`) is a member function of `ostream` (inherited by `ofstream`). And `read` is a member function of `istream` (inherited by `ifstream`). Objects of class `fstream` have both. Their prototypes are:（文件流包括两个专门设计用于顺序读取和写入二进制数据的成员函数——write 和 read。write 是 `ostream` 的成员函数，read 是 `istream` 的成员函数。`fstream` 类的对象包括二者，它们的原型是：）

```
write (memory_block, size);  
read (memory_block, size);  
```

Where `memory_block` is of type `char*` (pointer to `char`), and represents the address of an array of bytes where the read data elements are stored or from where the data elements to be written are taken. The `size` parameter is an integer value that specifies the number of characters to be read or written from/to the memory block. （`memory_block` 的类型是 `char*`，表示存储读取数据元素或从中获取要写入的数据元素的字节数组的地址。`size` 参数是一个整数值，指定要从内存块读取或写入的字符数）

```cpp
// reading an entire binary file
#include <iostream> 
#include <fstream> 
using namespace std;

int main () {
  streampos size;
  char * memblock;

  ifstream file ("example. bin", ios::in|ios::binary|ios::ate);
  if (file. is_open ())
  {
    size = file.tellg ();
    memblock = new char [size];
    file. seekg (0, ios::beg);
    file. read (memblock, size);
    file.close ();

    cout << "the entire file content is in memory";

    delete[] memblock;
  }
  else cout << "Unable to open file";
  return 0;
}
```

In this example, the entire file is read and stored in a memory block. Let's examine how this is done:

First, the file is open with the `ios::ate` flag, which means that the get pointer will be positioned at the end of the file. This way, when we call to member `tellg ()`, we will directly obtain the size of the file. （文件使用 `ios::ate` 标志打开，即 get 指针位于文件末尾，这样调用 `tellg()` 成员可以直接获取文件大小）

Once we have obtained the size of the file, we request the allocation of a memory block large enough to hold the entire file:

```
memblock = new char[size];
```

Right after that, we proceed to set the _get position_ at the beginning of the file (remember that we opened the file with this pointer at the end), then we read the entire file, and finally close it: （创建足够大内存块后，在文件的开头设置 get 位置，然后读取整个文件）

```
file.seekg (0, ios::beg);
file.read (memblock, size);
file.close ();
```

At this point we could operate with the data obtained from the file. But our program simply announces that the content of the file is in memory and then finishes.  

## Buffers and Synchronization

When we operate with file streams, these are associated to an internal buffer object of type `streambuf`. This buffer object may represent a memory block that acts as an intermediary between the stream and the physical file. For example, with an `ofstream`, each time the member function `put` (which writes a single character) is called, the character may be inserted in this intermediate buffer instead of being written directly to the physical file with which the stream is associated.（当操作文件流时，它们与 `streambuf` 类型的内部缓冲区对象相关联。该缓冲区对象可以表示充当流和物理文件之间的中介的内存块。例如，使用 `ofstream` 每次调用成员函数 `put` 写入单个字符时，可以将该字符插入到此中间缓冲区，而不是直接写入与流相关联的物理文件）

The operating system may also define other layers of buffering for reading and writing to files.

When the buffer is flushed, all the data contained in it is written to the physical medium (if it is an output stream). This process is called _synchronization_ and takes place under any of the following circumstances:（刷新缓冲区时，对于输出流而言其中所有数据都将写入物理介质，这一过程称作*同步*，可以在以下情况中发生）

*   **When the file is closed:** before closing a file, all buffers that have not yet been flushed are synchronized and all pending data is written or read to the physical medium.（关闭文件前，将同步所有尚未刷新的缓冲区，并将所有挂起的数据写入或读取到物理介质）
*   **When the buffer is full:** Buffers have a certain size. When the buffer is full it is automatically synchronized.（缓冲区已满时，会自动进行同步）
*   **Explicitly, with manipulators:** When certain manipulators are used on streams, an explicit synchronization takes place. These manipulators are: [flush](https://cplusplus.com/flush) and [endl](https://cplusplus.com/endl).（在流上使用某些操作如 `flush` 和 `endl` 将会显式同步）
*   **Explicitly, with member function sync ():** Calling the stream's member function `sync ()` causes an immediate synchronization. This function returns an `int` value equal to -1 if the stream has no associated buffer or in case of failure. Otherwise (if the stream buffer was successfully synchronized) it returns `0`.（调用流的成员函数 `sync()` 会立即同步，如果流没有关联的缓冲区或发生故障，则会返回 -1，否则将返回 0）