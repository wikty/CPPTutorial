# Input/output with files

C++ provides the following classes to perform output and input of characters to/from files: 

- **ofstream:** Stream class to write on files
- **ifstream:** Stream class to read from files
- **fstream:** Stream class to both read and write from/to files.

These classes are derived directly or indirectly from the classes `istream` and `ostream`. We have already used objects whose types were these classes: `cin` is an object of class `istream` and `cout` is an object of class `ostream`. Therefore, we have already been using classes that are related to our file streams. And in fact, we can use our file streams the same way we are already used to use `cin` and `cout`, with the only difference that we have to associate these streams with physical files.

## Open file

The first operation generally performed on an object of one of these classes is to associate it to a real file. This procedure is known as to *open a file*. An open file is represented within a program by a *stream* and any input or output operation performed on this stream object will be applied to the physical file associated to it.

For example:

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main () {
  ofstream myfile;
  myfile.open ("example.txt");  // open file
  myfile.close();				// close file
  return 0;
}
```

In order to open a file with a stream object we use its member function `open`:

```
file_stream.open(filename, mode);
```

where `filename` is a string representing the name of the file to be opened, and `mode` is an optional parameter with a combination of the following flags:

| `ios::in`     | Open for input operations.                                   |
| ------------- | ------------------------------------------------------------ |
| `ios::out`    | Open for output operations.                                  |
| `ios::binary` | Open in binary mode.                                         |
| `ios::ate`    | Set the initial position at the end of the file. If this flag is not set, the initial position is the beginning of the file. |
| `ios::app`    | All output operations are performed at the end of the file, appending the content to the current content of the file. |
| `ios::trunc`  | If the file is opened for output operations and it already existed, its previous content is deleted and replaced by the new one. |

All these flags can be combined using the bitwise operator OR (`|`). 

File streams opened in **binary mode** perform input and output operations independently of any format considerations. Non-binary files are known as **text files**, and some translations may occur due to formatting of some special characters (like newline and carriage return characters).

For example:

```c++
ofstream myfile;
// output to a binary file and append content to the end of file
myfile.open ("example.bin", ios::out | ios::app | ios::binary);
```

Each of the `open` member functions of classes `ofstream`,  `ifstream` and `fstream` has a default mode that is used if the file is opened without a second argument:

| class      | default mode parameter |
| ---------- | ---------------------- |
| `ofstream` | `ios::out`             |
| `ifstream` | `ios::in`              |
| `fstream`  | `ios::in | ios::out`   |

Since the first task that is performed on a file stream is generally to open a file, these three classes include a constructor that automatically calls the `open` member function and has the exact same parameters as this member.  Thus we can re-new the above example as follows:

```c++
ofstream myfile("example.bin", ios::out | ios::app | ios::binary);
```

To check if a file stream was successful opening a file, you can do it by calling to member `is_open`. This member function returns a `bool` value of `true` in the case that indeed the stream object is associated with an open file, or `false`otherwise:

```c++
if(myfile.is_open()) {/* checked file has been opened */}
```

## Close file

When we are finished with our input and output operations on a file we shall close it so that the operating system is notified and its resources become available again. For that, we call the stream's member function `close`. This member function *takes flushes the associated buffers and closes the file*:

```
myfile.close()
```

Once this member function is called, the stream object can be re-used to open another file, and the file is available again to be opened by other processes.

In case that an object is destroyed while still associated with an open file, the destructor automatically calls the member function `close`.  

## Checking state flags

The following member functions exist to check for specific states of a stream (all of them return a boolean value): 

- `bad()`

  Returns `true` if a reading or writing operation fails. For example, in the case that we try to write to a file that is not open for writing or if the device where we try to write has no space left.

- `fail()`

  Returns `true` in the same cases as `bad()`, but also in the case that a format error happens, like when an alphabetical character is extracted when we are trying to read an integer number.

- `eof()`

  Returns `true` if a file open for reading has reached the end.

- `good()`

  It is the most generic state flag: it returns `false` in the same cases in which calling any of the previous functions would return `true`. Note that `good` and `bad` are not exact opposites (`good` checks more state flags at once).

The member function `clear()` can be used to reset the state flags.

## Get and put stream positioning

 All i/o streams objects keep internally -at least- one internal position:

* `ifstream`, like `istream`, keeps an internal **get position** with the location of the element to be read in the next input operation.

* `ofstream`, like `ostream`, keeps an internal **put position** with the location where the next element has to be written.  
* Finally, `fstream`, keeps both, the *get* and the *put position*, like `iostream`.

These internal stream positions point to the locations within the stream where the next reading or writing operation is performed. These positions can be observed and modified using the following member functions: 

**Tell positions**

The `tellg()` and `tellp()` functions.

These two member functions with no parameters return a value of the member type `streampos`,  which is a type representing the current get position or the put position.

**Seek positions**

The `seekg()` and `seekp()` functions.

These functions allow to change the location of the get and put positions. Both functions are overloaded with two different prototypes:

`seekg(position);` and `seekp(position);`, the stream pointer is changed to the absolute position `position` (counting from the beginning of the file). The type for this parameter is `streampos`

`seekg(offset, direction);` and `seekp(offset, direction);`, the *get* or *put position* is set to an offset value relative to some specific point determined by the parameter `direction`. `offset` is of type `streamoff`. And `direction` is of type `seekdir`, which is an *enumerated type* that determines the point from where offset is counted from, and that can take any of the following values: `ios::beg`, `ios::cur`, `ios::end`.

These stream positioning functions use two particular types: `streampos` and `streamoff`. These types are also defined as member types of the stream class:

| Type        | Member type     | Description                                                  |
| ----------- | --------------- | ------------------------------------------------------------ |
| `streampos` | `ios::pos_type` | Defined as `fpos<mbstate_t>`. It can be converted to/from `streamoff` and can be added or subtracted values of these types. |
| `streamoff` | `ios::off_type` | It is an alias of one of the fundamental integral types (such as `int` or `long long`). |

For example, to get the size of a file:

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main () {
  streampos begin,end;
  ifstream myfile ("example.bin", ios::binary);
  begin = myfile.tellg();
  myfile.seekg (0, ios::end);
  end = myfile.tellg();
  myfile.close();
  cout << "size is: " << (end-begin) << " bytes.\n";
  return 0;
}
```

## Text files

Text file streams are those where the `ios::binary` flag is not included in their opening mode. These files are designed to store text and thus all values that are input or output from/to them can suffer some **formatting transformations**, which do not necessarily correspond to their literal binary value.

Example:

```c++
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

void write_to_file(const char* filename) {
  // write to file
  ofstream myfile (filename);
  if (myfile.is_open()) {
    myfile << "This is a line.\n";
    myfile << "This is another line.\n";
    myfile.close();
  }
  else cout << "Unable to open file";
}

void read_from_file(const char* filename) {
    string line;
    ifstream myfile(filename);
    if (myfile.is_open()) {
        while (getline(myfile, line)) {
            cout << line << '\n';
        }
        myfile.close();
    }
    else cout << "Unable to open file";
}

int main () {
  write_to_file("output.txt");	
  read_from_file("output.txt");
  return 0;
}
```

**Note:** The value returned by [getline](http://www.cplusplus.com/getline) is a reference to the stream object itself, which when evaluated as a boolean expression (as in this while-loop) is `true` if the stream is ready for more operations, and `false` if either the end of the file has been reached or if some other error occurred.

## Binary files

For binary files, reading and writing data with the extraction and insertion operators (`<<` and `>>`) and functions like `getline` is **not efficient**, since we do not need to format any data and data is likely not formatted in lines.

File streams include two member functions specifically designed to read and write binary data sequentially: `write` and `read`. The first one (`write`) is a member function of `ostream` (inherited by `ofstream`). And `read` is a member function of `istream` (inherited by `ifstream`). Objects of class `fstream` have both. Their prototypes are:  

```
write ( memory_block, size );
read ( memory_block, size );
```

where `memory_block` is of type `char*` (pointer to `char`), and represents the address of an array of bytes where the read data elements are stored or from where the data elements to be written are taken. The `size` parameter is an integer value that specifies the number of characters to be read or written from/to the memory block. For example:

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main() {
    streampos size;
    char* memblock;
    // open file and set the pointer at the end of file
    ifstream myfile("data.bin", ios::binary | ios::ate);
    
    if (myfile.is_open()) {
    	size = myfile.tellg();  // get the position of current file pointer, i.e., obtain the size of the file
    	memblock = new char[size];
    	myfile.seekg (0, ios::beg);  // reset file pointer
    	myfile.read(memblock, size);
    	myfile.close();
    	
    	cout << "File content in memory now";
    	
    	delete[] memblock;
	}
	else cout << "Unable to open file";
    
    return 0;
}
```

## Buffers and Synchronization

When we operate with file streams, these are associated to an internal **buffer object** of type `streambuf`. *This buffer object may represent a memory block that acts as an intermediary between the stream and the physical file*. For example, with an `ofstream`, each time the member function `put` (which writes a single character) is called, the character may be inserted in this intermediate buffer instead of being written directly to the physical file with which the stream is associated.

The operating system may also define other layers of buffering for reading and writing to files.

When the buffer is flushed, all the data contained in it is written to the physical medium (if it is an output stream). This process is called **synchronization** and takes place under any of the following circumstances: 

- **When the file is closed:** before closing a file, all buffers that have not yet been flushed are synchronized and all pending data is written or read to the physical medium.
- **When the buffer is full:** Buffers have a certain size. When the buffer is full it is automatically synchronized.
- **Explicitly, with manipulators:** When certain manipulators are used on streams, an explicit synchronization takes place. These manipulators are: `flush` and `endl`.
- **Explicitly, with member function sync():** Calling the stream's member function `sync()` causes an immediate synchronization. This function returns an `int` value equal to `-1` if the stream has no associated buffer or in case of failure. Otherwise (if the stream buffer was successfully synchronized) it returns `0`.

