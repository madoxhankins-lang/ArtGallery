# Custom Printf Function - Explanations

## Layman's Terms Explanation

### What is this program?
Think of this program like a smart printer that can print different types of information. Just like how a regular printer can print text, numbers, and pictures, our custom `printf` function can print characters, words, numbers, and even binary code (the 1s and 0s that computers use).

### How does it work?
Imagine you're writing a letter with blanks that need to be filled in. For example: "Hello, my name is ____ and I am ____ years old." Our program works similarly:

1. **You give it a template** (like "Character: %c") where `%c` is a placeholder
2. **You tell it what to fill in** (like the letter 'A')
3. **It prints the result** ("Character: A")

### What can it do?
- **%c** - Print a single character (like 'A' or 'Z')
- **%s** - Print a whole word or sentence (like "Hello, World!")
- **%d or %i** - Print regular numbers (like 42 or -123)
- **%b** - Print numbers in binary (like 5 becomes "101")
- **%%** - Print an actual percent sign
- **Regular text** - Print words without any special formatting

### The Helper Functions
The program uses smaller helper functions, like having different workers for different jobs:
- `printChar()` - The worker who prints single letters
- `printString()` - The worker who prints whole sentences
- `printInt()` - The worker who converts numbers to text and prints them
- `printBinary()` - The worker who converts numbers to binary (1s and 0s) and prints them

### Error Handling
If you forget to give the program a template (pass NULL), it politely tells you there's an error instead of crashing.

---

## Technical Explanation

### Architecture Overview
This implementation creates a custom variadic function `customPrintf()` that mimics the behavior of the standard C library's `printf()` function. The program is structured using modular helper functions that handle specific data type conversions and output operations.

### Core Components

#### 1. Helper Functions

**`printChar(char c)`**
- **Purpose**: Outputs a single character to stdout
- **Implementation**: Uses the `write()` system call to write 1 byte to file descriptor 1 (stdout)
- **Return**: Number of characters written (always 1)

**`printString(const char *str)`**
- **Purpose**: Outputs a null-terminated string to stdout
- **Implementation**: 
  - Handles NULL pointer by defaulting to "(null)"
  - Iterates through string character by character using `write()`
  - Counts and returns total characters written
- **Return**: Total number of characters printed

**`printInt(int num)`**
- **Purpose**: Converts an integer to its decimal string representation and prints it
- **Implementation**:
  - Uses a local buffer (12 bytes) to store the string representation
  - Handles edge cases: zero, negative numbers
  - Converts integer to string using modulo arithmetic (extracts digits from right to left)
  - Reverses the digit string since digits are collected in reverse order
  - Delegates to `printString()` for actual output
- **Algorithm**: 
  - For negative numbers: sets flag, converts to positive
  - Extracts digits: `digit = num % 10`, then `num = num / 10`
  - Reverses array using two-pointer technique
- **Return**: Number of characters in the printed number

**`printBinary(int num)`**
- **Purpose**: Converts an integer to its binary (base-2) string representation
- **Implementation**:
  - Uses unsigned integer conversion to handle bit manipulation correctly
  - Local buffer (33 bytes) accommodates 32-bit integers plus null terminator
  - Converts to binary using division by 2 (extracts bits from right to left)
  - Reverses the binary string before printing
  - Handles zero case explicitly
- **Algorithm**: Similar to `printInt()` but uses base-2 division instead of base-10
- **Return**: Number of characters in the binary representation

#### 2. Main Function: `customPrintf()`

**Function Signature**
```c
int customPrintf(const char *format, ...)
```
- **Variadic Function**: Uses `...` to accept variable number of arguments
- **Format String**: First parameter is the format template containing literal text and format specifiers

**Implementation Details**

1. **NULL Check**: Validates format string pointer before processing
   - Returns -1 and prints error message if NULL

2. **Variadic Argument Handling**:
   - `va_list args`: Type for accessing variadic arguments
   - `va_start(args, format)`: Initializes argument list (must be called before accessing arguments)
   - `va_arg(args, type)`: Retrieves next argument of specified type
   - `va_end(args)`: Cleans up variadic argument list (must be called before function returns)

3. **Format String Parsing**:
   - Iterates through format string character by character
   - When `%` is encountered:
     - Advances past the `%`
     - Checks for valid format specifiers: `c`, `s`, `d`, `i`, `b`, `%`
     - Invalid specifiers are printed literally (both `%` and the character)
   - Regular characters are printed directly

4. **Format Specifier Handling** (using if-else chains, no switch-case):
   - `%c`: Extracts `char` (promoted to `int` in variadic context)
   - `%s`: Extracts `char*` pointer
   - `%d`/`%i`: Extracts `int`, calls `printInt()`
   - `%b`: Extracts `int`, calls `printBinary()`
   - `%%`: Prints literal `%` character
   - Invalid specifiers: Prints `%` followed by the invalid character

5. **Character Counting**:
   - Maintains `totalChars` counter
   - Accumulates return values from helper functions
   - Returns total characters printed

### Memory Management
- **Stack Allocation**: Helper functions use local buffers (stack-allocated arrays)
- **No Dynamic Allocation**: Current implementation uses fixed-size buffers sufficient for all data types
- **Buffer Sizing**: 
  - `printInt()`: 12 bytes (handles -2,147,483,648 to 2,147,483,647)
  - `printBinary()`: 33 bytes (32 bits + null terminator)

### System Calls
- **`write(int fd, const void *buf, size_t count)`**: Low-level I/O function from `<unistd.h>`
  - File descriptor 1 = stdout (standard output)
  - Direct system call, bypasses buffering (unlike `printf()`)

### Compilation Requirements
- **Flags**: `-Wall -Werror -Wextra -pedantic -std=c11`
  - `-Wall`: Enable all warnings
  - `-Werror`: Treat warnings as errors
  - `-Wextra`: Extra warning flags
  - `-pedantic`: Strict ISO C compliance
  - `-std=c11`: Use C11 standard

### Design Decisions

1. **No Switch-Case**: Requirement specifies if-else chains only
2. **Modular Design**: Separate functions for each data type improves maintainability
3. **String Reversal**: Digits/bits collected in reverse order, requiring reversal before output
4. **Error Handling**: Graceful NULL handling prevents segmentation faults
5. **Return Values**: All functions return character count for consistency with standard `printf()`

### Time Complexity
- **Format String Parsing**: O(n) where n is length of format string
- **Integer Conversion**: O(log₁₀ n) for decimal, O(log₂ n) for binary
- **Overall**: O(n + m) where n is format string length, m is total output length

### Space Complexity
- **Auxiliary Space**: O(1) - fixed-size buffers regardless of input size
- **Stack Space**: O(1) - constant number of local variables
