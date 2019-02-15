[TOC]

## Bitwise Hacks

* If we subtract 1 from a number, then all the bits from right and upto _lowest set bit_ (including thislowest set bit) are reversed.

  ```c++
  x = 10    # 1010
  y = x-1   # 1001 (startig from 1 at last 2nd pos to last everything is reversed)
  ```

   Thus, `x & ~(x - 1)`  consists of only the lowest set bit of x. However, this only tells us the bit value, not the index of the bit.  `x ^ (x & x-1)` also does the same. The 386 introduced CPU instructions for bit scanning: BSF (bit scan forward) and BSR (bit scan reverse). GCC exposes these instructions through the built-in functions  **__builtin_ctz (count trailing zeros) and _builtin_clz (count leading zeros)**. These are the most convenient way to find bit indices for C++ programmers in TopCoder. Be warned though: the return value is undefined for an argument of zero.

* `x = x & (x-1)` is used for traverssing through the set bits. Each iteration clears the last set bit. Another way is `x = x - (x & ~(x-1))`. Another way is `x = x - (x & -x)`.

* `~(x-1) = -x` . Let num be the integer whose last digit we want to isolate. In binary notation num can be represented as a1b, where a represents binary digits before the last digit and b represents zeroes after the last digit. 

  Integer -num is equal to (a1b)¯ + 1 = a¯0b¯ + 1. b consists of all zeroes, so b¯ consists of all ones. Finally we have

  -num = (a1b)¯ + 1 = a¯0b¯ + 1 = a¯0(0…0)¯ + 1 = a¯0(1…1) + 1 = a¯1(0…0) = a¯1b.

* **a << b** and **a >> b** :   You can think of left-shifting by b as multiplication by $2^b$ and right-shifting as integer division by $2^b$.

* Set union

  `A | B`

* Set intersection

  `A & B`

* Set subtraction

  `A & ~B`

* Set negation 

  `ALL_BITS ^ A            #A = ALL_BITS is a number with 1′s for all bits corresponding to the elements of the domain`

* Set bit

  `A |= 1 << bit`

* Clear bit

  `A &= ~(1 << bit)`

* Test bit

  `(A & 1 << bit) != 0` 

#### Compute the sign of integer

##### -1 if -ve 0 if +ve 

```c++
// CHAR_BIT is the number of bits per byte (normally 8).
sign = -(v < 0);  // if v < 0 then -1, else 0. 
// or, to avoid branching on CPUs with flag registers (IA32):
sign = -(int)((unsigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));
// or, for one less instruction (but not portable):
sign = v >> (sizeof(int) * CHAR_BIT - 1); 
```

when signed integers are shifted right, the value of the far left bit is copied to the other bits. `10000000 >>7 = 11111111 `.  when unsigned integers are sifted right,  0 is copied to the other bits. `10000000 >>7 = 00000001`.

##### -1 if -ve +1 if +ve

```c++
sign = +1 | (v >> (sizeof(int) * CHAR_BIT - 1));  // if v < 0 then -1, else +1
```

##### -1 if -ve, 0, +1 if +ve 

```C++
sign = (v != 0) | -(int)((unsigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));
// Or, for more speed but less portability:
sign = (v != 0) | (v >> (sizeof(int) * CHAR_BIT - 1));  // -1, 0, or +1
// Or, for portability, brevity, and (perhaps) speed:
sign = (v > 0) - (v < 0); // -1, 0, or +1
```

##### +1 if +ve else 0

```C++
sign = 1 ^ ((unsigned int)v >> (sizeof(int) * CHAR_BIT - 1)); // if v < 0 then 0, else 1
```

---

#### Compute the integer absolute value (abs) without branching

```C++
int v;           // we want to find the absolute value of v
unsigned int r;  // the result goes here 
int const mask = v >> sizeof(int) * CHAR_BIT - 1;   # gives all 1's
r = (v ^ mask) - mask;  # does 2's complement
```

---

#### Compute minimum (min) or maximum (max) of two integers without branching

```C++
int x;  // we want to find the minimum of x and y
int y;   
int r;  // the result goes here 
r = y ^ ((x ^ y) & -(x < y)); // min(x, y)
r = x ^ ((x ^ y) & -(x < y)); // max(x, y)
```

It works because `if x < y, then -(x < y)` will be all ones, so `r = y ^ (x ^ y) & ~0 = y ^ x ^ y = x`. Otherwise, if x >= y, then -(x < y) will be all zeros, so ` r = y ^ ((x ^ y) & 0) = y`. On some machines, evaluating (x < y) as 0 or 1 requires a branch instruction, so there may be no advantage. 

---

#### Determining if an integer is a power of 2

```C++
f = (v & (v - 1)) == 0;
Note that 0 is incorrectly considered a power of 2 here. To remedy this, use:
f = v && !(v & (v - 1));
```

---

#### Conditionally set or clear bits without branching 

```C++
bool f;         // conditional flag
unsigned int m; // the bit mask
unsigned int w; // the word to modify:  if (f) w |= m; else w &= ~m; 

w ^= (-f ^ w) & m;

// OR, for superscalar CPUs:
w = (w & ~m) | (-f & m);
```

---

#### Conditionally negate a value without branching

```C++
bool fDontNegate;  // Flag indicating we should not negate v.
int v;             // Input value to negate if fDontNegate is false.
int r;             // result = fDontNegate ? v : -v;

r = (fDontNegate ^ (fDontNegate - 1)) * v;

If you need to negate only when a flag is true, then use this:

bool fNegate;  // Flag indicating if we should negate v.
int v;         // Input value to negate if fNegate is true.
int r;         // result = fNegate ? -v : v;

r = (v ^ -fNegate) + fNegate;
```

---

#### Merge bits from two values according to a mask 

```C++
unsigned int a;    // value to merge in non-masked bits
unsigned int b;    // value to merge in masked bits
unsigned int mask; // 1 where bits from b should be selected; 0 where from a.
unsigned int r;    // result of (a & ~mask) | (b & mask) goes here

r = a ^ ((a ^ b) & mask); 
```

---

#### Counting bits set

##### naive way

```C++
for (c = 0; v; v >>= 1)
{
  c += v & 1;
}
```

##### Brian Kernighan's way

```C++
for (c = 0; v; c++)
{
  v &= v - 1; // clear the least significant bit set
}
```

##### lookup table way

```C++
static const unsigned char BitsSetTable256[256] = 
{
#   define B2(n) n,     n+1,     n+1,     n+2
#   define B4(n) B2(n), B2(n+1), B2(n+1), B2(n+2)
#   define B6(n) B4(n), B4(n+1), B4(n+1), B4(n+2)
    B6(0), B6(1), B6(1), B6(2)
};

unsigned int v; // count the number of bits set in 32-bit value v
unsigned int c; // c is the total bits set in v

// Option 1:
c = BitsSetTable256[v & 0xff] + 
    BitsSetTable256[(v >> 8) & 0xff] + 
    BitsSetTable256[(v >> 16) & 0xff] + 
    BitsSetTable256[v >> 24]; 

// Option 2:
unsigned char * p = (unsigned char *) &v;
c = BitsSetTable256[p[0]] + 
    BitsSetTable256[p[1]] + 
    BitsSetTable256[p[2]] +	
    BitsSetTable256[p[3]];


// To initially generate the table algorithmically:
BitsSetTable256[0] = 0;
for (int i = 0; i < 256; i++)
{
  BitsSetTable256[i] = (i & 1) + BitsSetTable256[i / 2];
}
```





