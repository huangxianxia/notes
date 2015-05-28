## Chapter 5. Standard I/O Library

The standard I/O library handles such details as buffer allocation and performing I/O in optimal-sized chunks.

### Streams and `FILE` Objects

Standard I/O file streams can be used with both **single-byte** and **multibyte** ("wide") character sets. A stream’s orientation determines whether the characters that are read and written are single byte or multibyte. 

* This book deals only with **byte-oriented** (single byte) streams.
* This book refers to a pointer to a `FILE` object, the type `FILE *`, as a *file pointer*.

### Standard Input, Standard Output, and Standard Error

Three streams are predefined and automatically available to a process. They refer to file descriptors `STDIN_FILENO`, `STDOUT_FILENO`, and `STDERR_FILENO` (defined in `<unistd.h>`) [p9]. These three standard I/O streams are referenced through the predefined file pointers `stdin`, `stdout`,and `stderr`(defined in `<stdio.h>`).

### Buffering

1. **Fully buffered**
2. **Line buffered**
3. **Unbuffered**

Most implementations default to the following types of buffering:

* Standard error is always unbuffered.
* All other streams are line buffered if they refer to a terminal device; otherwise, they are fully buffered.

<script src="https://gist.github.com/shichao-an/70e28ba25f1b7276e834.js"></script>

* `setbuf`: *buf* must point to a buffer of length `BUFSIZ`, a constant defined in `<stdio.h>`
* `setvbuf`: type of buffering is specified with `_IOFBF`, `_IOLBF`, `_IONBF`.

The GNU C librarys use the value from the `st_blksize` member of the `stat` structure to determine the optimal standard I/O buffer size.

The `fflush` function causes any unwritten data for the stream to be passed to the kernel. If **fp** is `NULL`, `fflush` causes all output streams to be flushed.

### Opening a Stream

<script src="https://gist.github.com/shichao-an/3fea32272cd9e1b574c6.js"></script>

* `fdopen` function is often used with descriptors returned by the functions that create pipes and network communication channels, because these special types of files cannot be opened with the `fopen` function.

**type** argument has 15 values as specifed by ISO C:

**type** | Description | `open`(2) Flags
---- | ----------- | ---------------
`r`, `rb` | open for reading | <code>O_RDONLY</code>
`w`, `wb` | truncate to 0 length or create for writing | <code>O_WRONLY&#124;O_CREAT&#124;O_TRUNC</code>
`a`, `ab` | append; open for writing at end of file, or create for writing | <code>O_WRONLY&#124;O_CREAT&#124;O_APPEND</code>
`r+`, `r+b`, `rb+` | open for reading and writing | <code>O_RDWR</code>
`w+`, `w+b`, `wb+` | truncate to 0 length or create for reading and writing | <code>O_RDWR&#124;O_CREAT&#124;O_TRUNC</code>
`a+`, `a+b`, `ab+` | open or create for reading and writing at end of file | <code>O_RDWR&#124;O_CREAT&#124;O_APPEND</code>

Character `b` allows the standard I/O system to differentiate between a text file and a binary file. The UNIX kernel doesn’t differentiate between these types of files, thus character `b` has no effect.

* **Write**: The `fdopen` function cannot truncate any file it opens for writing
* **Append**: each write will take place at the then current end of file. If multiple processes open the same file with the standard I/O append mode, the data from each process will be correctly written to the file
* **Read and write** (`+` sign in type): Output cannot be directly followed by input without an intervening `fflush`, `fseek`, `fsetpos`, or `rewind`. Input cannot be directly followed by output without an intervening `fseek`, `fsetpos`, or `rewind`, or an input operation that encounters an end of file.


<script src="https://gist.github.com/shichao-an/3c8458270dc1f08325f4.js"></script>

An open stream is closed by calling `fclose`:

* Any buffered output data is flushed before the file is closed
* Any input data that may be buffered is discarded

When a process terminates normally, either by calling the exit function directly or by returning from the main function, all standard I/O streams with unwritten buffered data are flushed and all open standard I/O streams are closed.

### Reading and Writing a Stream

Unformatted I/O:

* Character-at-a-time I/O
* Line-at-a-time I/O: `fgets` and `fputs`. Each line is terminated with a newline character.
* Direct I/O (binary I/O, object-at-a-time I/O, record-oriented I/O, or structure-oriented I/O): `fread` and `fwrite`. For each I/O operation, we read or write some number of objects, where each object is of a specified size

#### Input Functions

<script src="https://gist.github.com/shichao-an/afcad88f5484e55327c4.js"></script>

* The function `getchar` is defined to be equivalent to `getc(stdin)`. 
* `getc` can be implemented as a macro, whereas `fgetc` cannot be implemented as a macro.
* These three functions return the next character as an `unsigned char` converted to an `int`. Thus, all possible character values can be returned, along with an indication that either an error occurred or the end of file has been encountered. The constant EOF in `<stdio.h>` is required to be a negative value. Its value is often −1.

These functions return the same value whether an error occurs or the end of file is reached. To distinguish between the two, we must call either `ferror` or `feof`:

<script src="https://gist.github.com/shichao-an/fe81c6d98b62e487aaf3.js"></script>

In most implementations, two flags are maintained for each stream in the `FILE` object:

* An error flag
* An end-of-file flag

Both flags are cleared by calling `clearerr`.

##### **Pushback**

After reading from a stream, we can push back characters by calling `ungetc`.

<script src="https://gist.github.com/shichao-an/1de26637c53e6de17b25.js"></script>

* The characters that are pushed back are returned by subsequent reads on the stream in reverse order of their pushing.
* The character that is pushed back does not have to be the same character that was read.
* When characters are pushed back with `ungetc`, they are not written back to the underlying file or device. Instead, they are kept incore in the standard I/O library’s buffer for the stream. EOF cannot be pushed back.
* Used for peeking characters.

#### Output Functions

<script src="https://gist.github.com/shichao-an/3aaa0731d3fbbe68da0c.js"></script>

* `putchar(c)` is equivalent to `putc(c, stdout)`
* `putc` can be implemented as a macro, whereas `fputc` cannot be implemented as a macro.

### Line-at-a-Time I/O

<script src="https://gist.github.com/shichao-an/2c9049288e8634b92734.js"></script>

* `gets` function reads from standard input, whereas `fgets` reads from the specified stream.
* `fgets`: reads *n - 1* characters (including the newline) or partial line if longer than *n - 1* into the buffer, then the buffer is (always) null terminated.
* `gets`: should never be used. Without specifying buffer size, this may cause buffer to overflow if the line is longer than the buffer, writing over whatever happens to follow the buffer in memory. `gets` is marked as an obsolescent interface in SUSv4 and has been omitted from the latest version of the ISO C standard

<script src="https://gist.github.com/shichao-an/5a2fdd9294a2e4c37aad.js"></script>

* `fputs`: writes the null-terminated string to the specified stream without writing the null byte
* `puts`: writes the null-terminated string to the standard output without writing the null byte, and then writes a newline character to the standard output. `puts` should be avoided being used to prevent having to remember whether it appends a newline.

### Standard I/O Efficiency

Function | User CPU (seconds) | System CPU (seconds) | Clock time (seconds) | Bytes of program text
-------- | ------------------ | -------------------- | -------------------- | ---------------------
best time from Figure 3.6 | 0.05 | 0.29 | 3.18 | 
fgets, fputs | 2.27 | 0.30 | 3.49 | 143
getc, putc | 8.45 | 0.29 | 10.33 | 114
fgetc, fputc | 8.16 | 0.40 | 10.18 | 114
single byte time from Figure 3.6 | 134.61 | 249.94 | 394.95 | 

* One advantage of using the standard I/O routines is that we don’t have to worry about buffering or choosing the optimal I/O size.
* Usually, `getc` and `putc` are implemented as macros, but in the GNU C library implementation the macro simply expands to a function call.
* The line-at-a-time functions are implemented using memccpy(3). Often, the memccpy function is implemented in assembly language instead of C, for efficiency.

### Binary I/O

If doing binary I/O, we often want to read or write an entire structure at a time. There are problems with the previous functions:

* `getc`, `putc`: we have to loop through the entire structure one byte a time
* `fputs`: stops writing when it hits a null byte
* `fgets`: won't work correctly on input if any data bytes are null or newlines

<script src="https://gist.github.com/shichao-an/a90609694cb97c765ca2.js"></script>

These functions have two common uses:

Read or write a binary array (e.g write elements 2 through 5 of a floating-point array):

```c
float   data[10];

if (fwrite(&data[2], sizeof(float), 4, fp) != 4)
    err_sys("fwrite error");
```

Read or write a structure:

```c
struct {
    short  count;
    long   total;
    char   name[NAMESIZE];
} item;

if (fwrite(&item, sizeof(item), 1, fp) != 1)
    err_sys("fwrite error");
```

* `fread`: return value can be less than *nobj* if an error occurs or if the end of file is encountered
* `fwrite`: if the return value is less than the requested `nobj`, an error has occurred

These two functions won't work on different systems (sometimes even on the same system):

1. The offset of a member within a structure can differ between compilers and systems because of different [alignment requirements](http://en.wikipedia.org/wiki/Data_structure_alignment#Typical_alignment_of_C_structs_on_x86). Even on a single system, the binary layout of a structure can differ, depending on compiler options. [p157]
2. The binary formats used to store multibyte integers and floating-point values differ among machine architectures


### Positioning a Stream

<script src="https://gist.github.com/shichao-an/18d258b1815658a84cf7.js"></script>

* `ftell`: return file's position indicator (bytes from the beginning of the file)
* `fseek`
    * Binary file: *whence* can be `SEEK_SET`, `SEEK_CUR`, and `SEEK_END`
    * Text file: *whence* has to be `SEEK_SET`; *offset* can only be 0 (rewind the file to its beginning) or a value that was returned by `ftell` for that file.
* `rewind`: set the stream to the beginning of the file

<script src="https://gist.github.com/shichao-an/530b106fa164cb197077.js"></script>
<script src="https://gist.github.com/shichao-an/e37050a7db81810e78cc.js"></script>

### Formatted I/O

<script src="https://gist.github.com/shichao-an/558af328d4915c8d77b8.js"></script>

* `sprintf`: automatically appends a null byte at the end of the array, but this null byte is not included in the return value. `sprintf` is possible to overflow the buffer.
* `snprintf`: returns the number of characters that would have been written to the buffer had it been big enough. If `snprintf` returns a positive value less than the buffer size n, then the output was not truncated.

#### Conversion specification

```
%[flags][fldwidth][precision][lenmodifier]convtype
```

* Flag

    Flag | Description
    ---- | -----------
    `’` | (apostrophe) format integer with thousands grouping characters
    `-` | left-justify the output in the field
    `+` | always display sign of a signed conversion
    (space) | prefix by a space if no sign is generated
    `#` | convert using alternative form (include 0x prefix for hexadecimal format, for example)
    `0` | prefix with leading zeros instead of padding with spaces

* `fldwidth` specifies a minimum field width for the conversion
* `precision` specifies the minimum number of digits to appear for integer conversions, the minimum number of digits to appear to the right of the decimal point for floating-point conversions, or the maximum number of bytes for string conversions
* `lenmodifier` pecifies the size of the argument

    Length modifier | Description
    --------------- | -----------
    `hh` | signed or unsigned `char`
    `h` | signed or unsigned `short`
    `l` | signed or unsigned `long` or wide character
    `ll` | signed or unsigned `long` `long`
    `j` | `intmax_t` or `uintmax_t`
    `z` | `size_t`
    `t` | `ptrdiff_t`
    `L` | `long double`

### Doubts and Solutions
#### Verbatim

Section 5.4 on line buffering [p145]

> Second, whenever input is requested through the standard I/O library from either (a) an unbuffered stream or (b) a line-buffered stream (that requires data to be requested from the kernel), all line-buffered output streams are flushed. The reason for the qualifier on (b) is that the requested data may already be in the buffer, which doesn’t require data to be read from the kernel. Obviously, any input from an unbuffered stream, item (a), requires data to be obtained from the kernel.

Section 5.8 Standard I/O Efficiency [p155]

> The version using line-at-a-time I/O is almost twice as fast as the version using character-at-a-time I/O. If the fgets and fputs functions are implemented using getc and putc, then we would expect the timing to be similar to the getc version. Actually, we might expect the line-at-a-time version to take longer, since we would be adding the overhead of 200 million extra function calls to the existing 6 million ones.