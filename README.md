# Bitset-Based Search Algorithm

## Overview

You are not allowed to use any PHP standard library functions (e.g. array_XXX) for this task.

You can only use loops (while, do..while, for, foreach) and conditional
statements (if, else if, else, switch) to solve this question.

To get the code template from question.php in the same folder.

There are 2 parts:

1. Write code to populate $array with 100,000,000 random numbers.
2. Write code to find the number $match within $array. If number is found, set
   variable $found to TRUE, else, $found should be FALSE.

Your code must run < 10ms and do not consume memory > 2000 MB. Every time you run
your code, there will be messages printed to show you your time and memory usage.
Refer to question.php file.

## Solution

This project uses a **Bitset (Bit Array)** to track numbers and detect duplicates efficiently.

Instead of storing every number as a separate array element, each bit represents whether a particular number has appeared. This reduces memory usage compared with storing millions of individual numbers in a conventional PHP array.

## Requirements

- PHP CLI installed
- A terminal or command prompt

Check your PHP installation:

```bash
php -v
```

## Results

```
Populating array with 100000000 random numbers..

Checkpoint 1: Memory Usage 484696.000000 bytes
Checkpoint 2: 1791902006.757256ms.
Checkpoint 3: 1791902006.793761ms. Memory Usage 8877480.000000 bytes

Time used: 36.504984ms
Memory used: 8.004000 MB

Match found: Y
```

## How to Run

Navigate to the directory containing `bitset-search.php`, then run:

```bash
php bitset-search.php
```

The script populates the bitset with random numbers, searches for a target number, and reports the execution time and memory usage.

## How It Works

### 1. Map each number to a bit

The input numbers range from `0` to `10,000,000`.

Each integer in the PHP array represents **32 numbers**, with each bit indicating whether a number has appeared.

We calculate two values:

```php
$index = $num >> 5;
$mask = 1 << ($num & 31);
```

- `$index = $num >> 5`: Divides the number by 32 using a right bit shift, giving the group index.
- `$mask = 1 << ($num & 31)`: Creates a bitmask with exactly one bit turned on, representing the number's position within its group.

### 2. Example: Numbers 33 and 61

Both numbers belong to `$array[1]`, because both fall within the range `32–63`.

| Number | Array index | Bit position |
| -----: | ----------: | -----------: |
|     33 |           1 |            1 |
|     61 |           1 |           29 |

The corresponding bitmasks are:

```text
33 → 00000000 00000000 00000000 00000010
61 → 00100000 00000000 00000000 00000000
```

After both numbers are inserted, `$array[1]` contains:

```text
00100000 00000000 00000000 00000010
```

The two set bits indicate that both numbers have appeared. The integer does not store `33` and `61` directly; it stores their membership information.

### 3. Mark a number as present

```php
if (($array[$index] & $mask) == 0) {
    $array[$index] = $array[$index] | $mask;
}
```

The bitwise AND (`&`) checks whether the number's bit is already set.

- If the result is `0`, the number has not appeared in the bitset, so we turn its bit on using bitwise OR (`|`).
- If the result is nonzero, the bit is already set, so the number has appeared before.

Setting a bit that is already on does not change the bitset, so duplicates are naturally represented only once.

### 4. Search for a number

To check whether a target number exists:

```php
$index = $match >> 5;
$mask = 1 << ($match & 31);

if (($array[$index] & $mask) != 0) {
    $found = TRUE;
} else {
    $found = FALSE;
}
```

The algorithm calculates the target's group and bit position, then checks that bit directly.

It does not need to scan the entire collection.

## Why Use a Bitset?

- **Fast lookup:** Checking whether a number exists takes constant time, \(O(1)\).
- **Compact representation:** Each number requires one bit of logical storage.
- **Duplicate handling:** Repeated numbers do not require additional bits.
- **No hashing required:** Numbers map directly to bit positions.

## Time and Space Complexity

Let \(N\) be the number of generated inputs and \(M\) be the size of the number range.

| Operation             | Complexity                |
| --------------------- | ------------------------- |
| Populate the bitset   | \(O(N)\)                  |
| Search for one number | \(O(1)\)                  |
| Initialize the bitset | \(O(M/32)\) integer slots |
| Bitset storage        | \(O(M)\) bits             |

For the range `0–10,000,000`, the bitset needs 10,000,001 logical bits, or approximately **1.19 MiB** of raw bit storage. The actual PHP memory usage will be higher because each integer stored in a PHP array has additional overhead.

**Conclusion:** This implementation is a bitset-based membership tracking algorithm. It is a good fit when the possible integer range is known and reasonably bounded.
