## Assignment-3

#### ***General Assumptions:***
- All the programs uses the below MIPS instructions:
    - `.half` - Define Halfword --> Define a halfword (16 bits) in the `.data` section (pseudo-instruction)
    - `.word` - Define Word --> Define a word (32 bits) in the `.data` section
    - `lw` - Load Word --> Load a word (32 bits) from memory into a register
    - `sw` - Store Word --> Store a word (32 bits) from a register into memory
    - `la` - Load Address (pseudo-instruction) --> Load the address of a label into a register
    - `lh` - Load Halfword --> Load a halfword (16 bits) from memory into a register
    - `sh` - Store Halfword --> Store a halfword (16 bits) from a register into memory
    - `addu` - Add Unsigned --> Add two registers without overflow
    - `addi` - Add Immediate --> Add a constant value to a register 
    - `subi` - Subtract Immediate --> Subtract a constant value from a register
    - `add` - Add --> Add two registers
    - `sub` - Subtract --> Subtract one register from another
    - `mul` - Multiply --> Multiply two registers
    - `div` - Divide --> Divide one register by another
    - `mflo` - Move From LO --> Move the contents of the LO register to a register
    - `mfhi` - Move From HI --> Move the contents of the HI register to a register
    - `beq` - Branch if Equal --> Branch to a target address if two registers are equal
    - `bne` - Branch if Not Equal --> Branch to a target address if two registers are not equal
    - `slt` - Set Less Than --> Set a register to 1 if one register is less than another, 0 otherwise
    - `j` - Jump --> Jump to a target address
    - `loop` - Label for loop iteration
- In the above list, in whichever instruction if it is not a real MIPS instruction, then it is assumed to be a pseudo-instruction which works according to the description given in the list.
- The memory locations are assumed to be in the `.data` section of the MIPS program.
- The result of the program is stored in a memory location.
- In the assembly programs, it is assumed that the exact implementation details for terminating the program are not known. Therefore, a pseudo command `exit` is utilized to signify the program's termination. This command is intended to represent the program's exit functionality without specifying the precise assembly instructions that would normally be required for this operation.


### Q1. Write a program in assembly language to subtract two 16 bit numbers without using the subtraction instruction. Note: the numbers have to be fetched from the memory.

***Assumptions:***
- Numbers are assumed to be in integer format.
- The two 16-bit numbers are stored in memory locations `num1` and `num2`.
- The result of the subtraction will be stored in memory location `result`.

#### C Program
```c
int main() {
    // Define 16-bit integers
    int16_t num1 = 70;  // First number
    int16_t num2 = 18;  // Second number
    int16_t result;     // Variable to hold the result

    // Perform subtraction using two's complement
    // result = num1 - num2
    result = num1 + (~num2 + 1); // Two's complement of num2 is obtained by inverting bits and adding 1

    return 0; // Exit the program
}
```
#### Assembly Program
```mips
.data                   # Data section
num1:  .half   70        # First 16-bit number 
num2:  .half   18        # Second 16-bit number 
result: .half   0       # Result of the subtraction

    .text                   # Code section
    .globl  main

main:    
    # Load 16-bit numbers from memory
    lh     $t0, num1        # Load halfword (16-bit) from memory location 'num1' into register $t0
    lh     $t1, num2        # Load halfword (16-bit) from memory location 'num2' into register $t1
    
    # Convert $t1 to its two's complement to negate it
    # Two's complement = invert bits + 1
    nor     $t1, $t1         # Invert all bits of $t1
    addi    $t1, $t1, 1      # Add 1 to the inverted value to get the two's complement

    # Perform addition (which is effectively subtraction now)
    addu    $t2, $t0, $t1    # Add the value in $t0 and $t1 (two's complement of num2), result in $t2

    # Store the result back to memory
    sh      $t2, result      # Store halfword (16-bit) from $t2 into memory location 'result'

    j   exit                    # Exit the program
```


### Q2. Write an assembly language program to find an average of 15 numbers stored at consecutive locations in memory.

***Assumptions:***
- All numbers are integers.
- The 15 numbers are stored in consecutive memory locations starting from `base_address`.
- Assume that we don't know the `mul` and `div` instruction for this specific problem.

#### C Program
```c

int main() {
    // Define the array of 15 integers
    int base_address[15] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14};
    int result = 0;      // Variable to store the result (average)
    int sum = 0;         // Variable to store the sum
    int count = 14;      // Number of elements in the array
    
    // Calculate the sum of the array
    for (int i = count; i >= 0; i--) {
        sum += base_address[i]; // Add each element to the sum
    }

    // Manual division of sum by 15 using repeated subtraction
    int quotient = 0;  // Variable to store the quotient (average)
    int divisor = count; // Divisor is 15 (number of elements)
    int temp_sum = sum;  // Copy of sum for division process

    // Repeated subtraction to calculate sum / 15
    while (temp_sum >= divisor) {
        temp_sum -= divisor; // Subtract divisor from temp_sum
        quotient++;          // Increment quotient each time
    }

    // Store the final quotient as the result
    result = quotient;
    return 0; // Exit the program
}
```

#### Assembly Program
```mips
.data
# Define the array of 15 integers
base_adress: .word 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14  # Base address of the array
result:      .word 0          # Memory location to store the average

.text
.globl main
main:
    lw $t0, base_adress  # Load the base address of the array into $t0
    lw $t1, 0            # Initialize sum to 0
    lw $t2, 14           # Number of integers (15 numbers)
    lw $t3, 0            # Initialize sum to 0

loop:
    lw $t4, 0($t0)       # Load the current number from memory into $t4 (dereference the value stored at adress $t0)
    add $t1, $t1, $t3    # Add the current number to the sum in $t1
    addi $t0, $t0, 4     # Increment the address in $t0 to point to the next number
    subi $t2, $t2, 1     # Decrease the counter $t0 by 1

    lw $t5, 0            # Set $t5 to 0 to use for comparison
    bne $t0, $t5, loop   # If $t0 != 0, branch back to the loop

    # Calculate the average (manual division by shifting)
    lw $t6, 15           # Reload the counter (number of elements, 15)
    
    # Use a loop to calculate sum / 15 (integer division by repeated subtraction)
    move $t5, $t1        # Move sum to $t5 for division
    lw $t7, 0            # Initialize quotient (average) to 0

div_loop:
    sub $t5, $t5, $t6    # Subtract divisor (15) from the sum
    slt $t8, $t5, $zero  # Set $t8 to 1 if result is negative
    bne $t8, $zero, end_div # If $t5 < 0, exit the loop
    addi $t7, $t7, 1     # Increment quotient
    j div_loop           # Jump back to continue division

end_div:
    # Store the result (quotient) in memory
    sw $t7, average      # Store the average value in memory at the 'average' location

    j   exit                 # Exit the program
```

### Question 3: Write an assembly language program to find an LCM of two numbers stored at consecutive locations in memory.

***Assumptions:***
- The two numbers are stored in memory locations `num1` and `num2`.
- The LCM of the two numbers will be stored in memory location `lcm`.

#### C Program
```c
#include <stdio.h>

// Function to calculate the GCD using the Euclidean algorithm
int gcd(int a, int b) {
    while (b != 0) {
        int remainder = a % b; // Calculate the remainder
        a = b;                 // Set a to b
        b = remainder;         // Set b to the remainder
    }
    return a; // The GCD is stored in a
}

// Function to calculate the LCM using the formula: LCM(a, b) = (a * b) / GCD(a, b)
int lcm(int a, int b) {
    int gcd_value = gcd(a, b);  // Find the GCD of a and b
    return (a * b) / gcd_value; // Calculate LCM
}

int main() {
    // Define two numbers to find the LCM of
    int num1 = 15;
    int num2 = 20;

    // Calculate the LCM of num1 and num2
    int result = lcm(num1, num2);
    return 0;
}
```

#### Assembly Program
```mips
.data
number: .word 15, 20    # Two numbers to find the LCM of
lcm:       .word 0         # Memory location to store the LCM

.text
.globl main
main:
    # Load the two numbers into registers
    la $t0, numbers       # Load address of the numbers array
    lw $t1, 0($t0)        # Load the first number into $t1 (a)
    lw $t2, 4($t0)        # Load the second number into $t2 (b)
    
    # Save the original values for LCM calculation later
    move $t3, $t1         # Copy first number to $t3 (for LCM)
    move $t4, $t2         # Copy second number to $t4 (for LCM)

    # Find the GCD using the Euclidean algorithm
gcd_loop:
    beq $t2, $zero, gcd_done  # If b (second number) is 0, GCD is in $t1
    beq $t1, $t2, gcd_done    # If a == b, GCD is found

    div $t1, $t1, $t2         # Divide a by b
    mfhi $t1                  # Move the remainder to $t1
    move $t5, $t1             # Copy the remainder to $t5 (for LCM)
    move $t1, $t2             # Move b to a
    move $t2, $t5             # Move the remainder to b
    j gcd_loop                # Repeat until GCD is found

gcd_done:
    # Store the GCD in memory
    sw $t1, gcd               # Store GCD (in $t1) in memory
    
    # Calculate LCM using the formula: LCM = (a * b) / GCD
    # t3 = original a, t4 = original b, t1 = GCD
    
    mul $t5, $t3, $t4         # Multiply the two original numbers (t5 = a * b)
    div $t5, $t5, $t1         # Divide the product by the GCD (t5 = (a * b) / GCD)
    
    # Store the result (LCM) in memory
    sw $t5, lcm               # Store the LCM in memory at 'lcm'

    j   exit                      # Exit the program
```

### Question 4: Write an assembly language program to calculate multiplication of two numbers without using MUL commands.

***Assumptions:***
- The two numbers are stored in memory locations `num1` and `num2`.
- The result of the multiplication will be stored in memory location `result`.
- We don't know the `mul` instruction for this specific problem.

#### C Program
```c
int multiply(int a, int b) {
    int product = 0; // Initialize product to 0

    // Check if either number is zero
    if (a == 0 || b == 0) {
        return 0;
    }

    // Multiply by repeated addition
    while (b > 0) {
        product += a; // Add a to the product
        b--;          // Decrement b by 1
    }

    return product; // Return the final product
}

int main() {
    // Define two numbers to multiply
    int num1 = 6;
    int num2 = 7;

    // Call the multiply function and store the result
    int result = multiply(num1, num2);
    return 0;
}
```

#### Assembly Program
```mips
.data
numbers:    .word 6, 7    # Two numbers to multiply, e.g., 6 * 7
product:    .word 0       # Memory location to store the result (product)

.text
.globl main
main:
    # Load the two numbers into registers
    la   $t0, numbers       # Load address of the numbers array
    lw   $t1, 0($t0)        # Load the first number into $t1 (multiplicand)
    lw   $t2, 4($t0)        # Load the second number into $t2 (multiplier)

    # Initialize result to 0
    lw   $t3, 0             # $t3 will hold the product (initially 0)

    # Check if either number is zero (result is zero)
    beq  $t1, $zero, done   # If multiplicand is 0, skip to done
    beq  $t2, $zero, done   # If multiplier is 0, skip to done

    # Multiply by repeated addition
multiply_loop:
    beq  $t2, $zero, done   # If multiplier reaches 0, we're done
    add  $t3, $t3, $t1      # Add multiplicand to result (product)
    subi $t2, $t2, 1        # Decrease multiplier by 1
    j    multiply_loop      # Repeat until multiplier becomes 0

done:
    # Store the result (product) in memory
    sw   $t3, product       # Store the product in memory at 'product'

    exit                    # Exit the program
```

### Question 5: Write an assembly language program to find a given number in the list of 10 numbers (assuming the numbers are sorted). If found store 1 in output, else store 2 in output.The given number has been loaded from X location in memory, the output has to be stored at the next location and if found store the number of iterations and the index of the element at the next at the next consecutive locations, if found.

***Assumptions:***
- The sorted list of 10 numbers is stored in memory locations `numbers`.
- If we don't find the target number we don't need to store the number of iterations or the index in the respective memory locations.

#### C Program
```c
// Binary search function
int binary_search(int arr[], int size, int target, int* iterations) {
    int low = 0;
    int high = size - 1;
    *iterations = 0;

    while (low <= high) {
        (*iterations)++; // Increment iteration count

        int mid = (low + high) / 2; // Calculate mid index

        if (arr[mid] == target) {
            return mid; // Return the index where the target is found
        }

        if (arr[mid] < target) {
            low = mid + 1; // Search in the right half
        } else {
            high = mid - 1; // Search in the left half
        }
    }

    return -1; // Target not found
}

int main() {
    int numbers[] = {2, 5, 7, 10, 12, 15, 18, 21, 23, 30}; // Sorted list of numbers
    int target = 15; // Target number to search for
    int iterations = 0; // To store the number of iterations
    int index; // To store the result of binary search

    int output[3]; // Array to store results: output[0] = found/not found, output[1] = iterations, output[2] = index

    // Perform binary search
    index = binary_search(numbers, 10, target, &iterations);

    // Store the result (found/not found) in output[0]
    if (index != -1) {
        output[0] = 1;    // Found: output[0] = 1
        output[1] = iterations; // Store the number of iterations in output[1]
        output[2] = index;    // Store index in output[2]
    } else {
        output[0] = 2; // Not found: output[0] = 2
    }
    return 0;
}
```

#### Assembly Program
```mips
.data
numbers:    .word 2, 5, 7, 10, 12, 15, 18, 21, 23, 30  # Sorted list of 10 numbers
target:     .word 15      # The target number we want to search for

.text
.globl main
main:
    # Load target number into $t0
    la   $t0, target      # Load address of the target number
    lw   $t1, 0($t0)      # Load target number into $t1

    # Initialize variables for binary search
    lw   $t2, 0           # Low index (start of the list) -> $t1
    lw   $t3, 9           # High index (end of the list) -> $t2
    lw   $t4, 0           # Iteration counter -> $t3
    lw   $t5, -1          # Mid index, used in calculations

binary_search:
    beq  $t2, $t3, not_found  # If low index > high index, not found

    # Increment iteration counter
    addi $t4, $t4, 1          # Increment $t3 (iterations)

    # Calculate mid index: mid = (low + high) / 2
    add  $t5, $t2, $t3        # Add low + high
    div $t5, $t5, 2          # Divide by 2
    mflo $t5                  # Move the result to $t5

    # Load the number at mid index into $t5
    la   $t6, numbers         # Load base address of the array
    mul  $t7, $t5, 4          # Multiply mid index by 4 (word address)
    add $t7, $t7, $t6        # Add the base address to get the address of numbers[mid]
    lw   $t8, 0($t7)     # Load numbers[mid] into $t8

    # Compare numbers[mid] with target
    beq  $t0, $t8, found      # If equal, found the number
    slt $t9, $t0, $t8        # Set $t9 to 1 if target < numbers[mid]
    beq $t9, $zero, search_right  # If target < numbers[mid], search right
    beq $t9, $zero, search_left   # If target > numbers[mid], search left

search_left:
    subi $t5,$t5, 1          # Set mid = mid - 1
    lw  $t3, $t5            # Set high = mid
    j    binary_search    # Jump back to binary_search

search_right:
    addi $t2, $t2, 1          # Set mid = mid + 1
    lw  $t2, $t5            # Set low = mid
    j    binary_search         # Jump back to binary_search

found:
    # Store 1 (found) in output
    lw   $t10, $t0          # Load address of the target
    addi $t10, $t10, 4      # Move to the next memory location
    la   $t10, output         # Load address of the output
    li   $t11, 1              # Load 1 (found)
    sw   $t11, 0($t10)        # Store the result in the output

    # Store the number of iterations in the next memory location
    lw   $t12, $10     # Load the address of output
    addi $t12, $t12, 4      # Move to the next memory location
    sw   $t4, 0($t12)         # Store the number of iterations

    # Store the index in the next memory location
    lw   $t13, $t12     # Load the address of iterations
    addi $t13, $t13, 4      # Move to the next memory location
    sw   $t5, 0($t13)         # Store the index

    # Exit the program
    j   exit

not_found:
    # Store 2 (not found) in output
    lw   $t10, $t0          # Load address of the target
    addi $t10, $t10, 4      # Move to the next memory location
    li   $t11, 2              # Load 2 (not found)
    sw   $t11, 0($t10)        # Store the result in the output

    # Exit the program
    j    exit
```

### Question 6: Write an assembly language program to find a character in a string.

***Assumptions:***
- The string is stored in memory location `str`.
- If we find the character, we store 1 in the output memory location and the index of the character in the next memory location.
- Output memory location is `output`.

#### C Program
```c

int main() {
    // Define the string, target character, and output memory locations
    char str[] = "Hello World";   // The string we are searching in
    char target = 'W';            // The character we want to find
    int output = 0;               // Output memory location, 1 if found, 2 if not found
    int index = -1;               // Index of the found character (if found)

    // Initialize the index counter
    int i = 0;

    // Loop to search for the target character
    while (str[i] != '\0') {
        if (str[i] == target) {   // If the target character is found
            output = 1;           // Store 1 in the output memory location (found)
            index = i;            // Store the index of the found character
            break;
        }
        i++;                      // Increment the index
    }

    // If the target character is not found, store 2 in the output memory location
    if (output == 0) {
        output = 2;               // Not found
    }
    return 0;
}
```

#### Assembly Program
```mips
.data
str:       .asciiz "Hello World"   # The string we are searching in
target:    .byte 'W'               # The character we want to find
output:    .word 0                 # Result: 1 if found, 2 if not found
index:     .word 0                 # Index of the found character (if found)

.text
.globl main
main:
    # Load the address of the string and target character
    la   $t0, str         # Load address of the string into $t0
    lb   $t1, target      # Load the target character into $t1

    # Initialize index counter to 0
    lw   $t2, 0           # $t2 will keep track of the index

find_char:
    lb   $t3, 0($t0)      # Load the current character of the string into $t3
    beq  $t3, $zero, not_found  # If current character is null, end of string reached

    # Compare the current character with the target character
    beq  $t3, $t1, found  # If equal, found the character

    # Move to the next character in the string
    addi $t0, $t0, 1      # Increment the string pointer
    addi $t2, $t2, 1      # Increment the index
    j    find_char        # Repeat the process

not_found:
    # Store 2 (not found) in output
    la   $t4, output      # Load address of the output
    lw   $t5, 2           # Load 2 (not found)
    sw   $t5, 0($t4)      # Store the result in the output

    # Exit the program
    j    exit

found:
    # Store 1 (found) in output
    la   $t4, output      # Load address of the output
    lw   $t5, 1           # Load 1 (found)
    sw   $t5, 0($t4)      # Store the result in the output

    # Store the index in the next memory location
    la   $t6, index       # Load the address of index
    sw   $t2, 0($t6)      # Store the index

    # Exit the program
    j    exit
```