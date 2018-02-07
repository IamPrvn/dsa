#### Rolling hash Technique

##### Introduction

Every sub-string s[] of length `m` can be seen as a number H written in a positional numeral system in base B (B >= size of the alphabet used in the string):
$$
H = s[0] \times B^{m-1} + s[1] \times B^{m-2} + … + s[m - 2] \times B^1 + s[m - 1] \times B^0
$$
Now for a string `n` with all possible substrings of length `m`  we can calculate   H~0~, H~1~ ... H~n-m-1~ in  `O(n)` 
$$
H_{i+1} =  ( H_i  -  s[i]  \times B^{m-1} ) \times m + s[i+m] \times B^0
\\ or \\
H_{i} =  ( H_{i-1}  -  s[i-1]  \times B^{m-1} ) \times m + s[i+m-1] \times B^0
$$
Doing it programmatically :-

A problem arises when m and B are big enough and the number H becomes too large to fit into the standard integer types. To overcome this, instead of the number H itself we use its remainder when divided by some other number M. 

A + B = C => (A % M + B % M) % M = C % M
A * B = C => ((A % M) * (B % M)) % M = C % M

res = (n % m + m ) % m   // if n < 0

Applying this modulo to above arithmethic expression :-

```c++
H % M = (((s[0] % M) * (B(m – 1) % M)) % M + ((s[1] % M) * (B(m – 2) % M)) % M +…
…+ ((s[m - 2] % M) * (B1 % M)) % M + ((s[m - 1] % M) * (B0 % M)) % M) % M
```

Finally the general formula begins :-
$$
H_i \mod M = (((( H_{i – 1} \mod M – ((s[i- 1] \mod M) \times (B^{m – 1} \mod M)) \mod M ) \mod M) \\ \times (B \mod M)) \mod M + s[i + m - 1] \mod M) \mod M
$$
Obviously the value of $$(H_{i – 1} – s[i - 1] \times B^{m - 1})$$  may be negative. Again, the rules of modular arithmetic come into play. 

##### Applications

1. Rabin-karp alogorithm for a fixed length pattern search in a string `o()`

   ```C++
   // correctly calculates a mod b even if a < 0
   function int_mod(int a, int b)
   {
     return (a % b + b) % b;
   }

   function Rabin_Karp(text[], pattern[])
   {
     // let n be the size of the text, m the size of the
     // pattern, B - the base of the numeral system,
     // and M - a big enough prime number

     if(n < m) return; // no match is possible

     // calculate the hash value of the pattern
     hp = 0;
     for(i = 0; i < m; i++) 
       hp = int_mod(hp * B + pattern[i], M);

     // calculate the hash value of the first segment 
     // of the text of length m
     ht = 0;
     for(i = 0; i < m; i++) 
       ht = int_mod(ht * B + text[i], M);

     if(ht == hp) check character by character if the first
                  segment of the text matches the pattern;

     // start the "rolling hash" - for every next character in
     // the text calculate the hash value of the new segment
     // of length m; E = (Bm-1) modulo M            
     for(i = m; i < n; i++) {
       ht = int_mod(ht - int_mod(text[i - m] * E, M), M);
       ht = int_mod(ht * B, M);
       ht = int_mod(ht + text[i], M);

       if(ht == hp) check character by character if the
                    current segment of the text matches
                    the pattern; 
     }
   }
   ```

2.  the “rolling hash” technique used in RK is an important weapon. It is especially useful in problems where we have to look at all substrings of fixed length of a given text. An example is “the longest common substring problem”: given two strings find the longest string that is a substring of both. In this case, the combination of binary search (BS) and “rolling hash” works quite well. The important point that allows us to use BS is the fact that if the given strings have a common substring of length n, they also have at least one common substring of any length m < n. And if the two strings do not have a common substring of length n they do not have a common substring of any length m > n. So all we need is to run a BS on the length of the string we are looking for. For every substring of the first string of the length fixed in the BS we insert it in a hash table using one hash value as an index and a second hash value (“double hash”) is inserted in the table. For every substring of the fixed length of the second string, we calculate the corresponding two hash values and check in the table to see if they have been already seen in the first string. A hash table based on open addressing is very suitable for this task.

   Of course in “real life” (real challenges) the number of the given strings may be greater than two, and the longest substring we are looking for should not necessarily be present in all the given strings. This does not change the general approach.

   _BS for Longest common substring_

   ``` c++
       l = 0 , r= min(s1.length(),s2.length())
       while (l<=r){
       	mid = l + (r-l)/2
       	if(p(s1,s2,mid)) // p(s1,s2,len) checks for common substring 
       		l = mid + 1
       	else
       		r = mid - 1
       }
       return l-1
   ```

   _rolling hash_

   ```C++
       p(s1,s2,x){ // s1:string 1 s2:string 2 x:length of substring
       	// To find hash I'm using B as prime.
       	//calculate hash(0) = hash(s1[0...x-1]) s1[0] + s1[1]*B + ....
            //  + s1[x-1]*pow(B,x-1) 
       	//To calculate all the other hashes use this equation
       	//  hash(i) = hash(s1[i...i+x-1]) = (hash(i-1) - s[i-1])/B 
            //   + s1[i+x-1]*pow(B,x-1)
       	// now stores all the hashes in hash table
       	// similarly calculates all the hashes for string 2 and compare them
       	// with existing hashes
       	if (match found)
       		return true
       	else
       		return false
       }
   ```

3.  The longest common substring (LCS) of two input strings s,t is a common substring (in both of them) of maximum length. We can relax the constraints to generalize the problem: find a common substring of length k. We can then use binary search to find the maximum k. This takes time O(nlgn)

   provided that solving the relaxed problem takes linear time.

   Finding a _k_-common substring can be solved using a rolling hash:

   - Compute hash values of all k-length substrings of s and t.
   - If a hash of s coincides with a hash of t, then we've found a k-length common substring.


   ​

   ​

   ​







