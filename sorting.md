[TOC]





#### template for sorting

![template for sorting](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0245-01.jpg)

> When studying sorting algorithms, we count *compares* and *exchanges.* For algorithms that do not use exchanges, we count *array accesses.*

- By convention, v.compareTo(w) throws an exception if v and w are incompatible types or either is null. Furthermore, compareTo() must implement a total order: it must be

  • Reflexive (for all v, v = v)

  • Antisymmetric (for all v and w, if v < w then w > v and if v = w then w = v)

  • Transitive (for all v, w, and x, if v <= w and w <= x then v <=x)



#### Selection sort

![Selection sort](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0249-01.jpg)

##### Proposition A

>  Selection sort uses ~$$N^2/2$$  compares and N exchanges to sort an array of length N.

​	 *You can prove this fact by examining the trace, which is an N-by-N table in which unshaded letters correspond to compares. About one-half of the entries in the table are unshaded—those on and above the diagonal. The entries on the diagonal each correspond to an exchange. More precisely, examination of the code reveals that, for each i from $$0$$ to $$N − 1$$, there is one exchange and $$N − 1 − i$$ compares, so the totals are N exchanges and $$(N − 1) + (N − 2) + . . . + 2 + 1+ 0 = N(N − 1) / 2 ~ N^2/2$$ compares.* 

![![img](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_01-selection.jpg)Selection trace](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_01-selection.jpg)



- Running time is insensitive of input. 
  - The process of finding the smallest item on one pass through the array does not give much information about where the smallest item might be on the next pass. 
  - This property can be disadvantageous in some situations. For example, the person using the sort client might be surprised to realize that it takes about as long to run selection sort for an array that is already in order or for an array with all keys equal as it does for a randomly-ordered array! As we shall see, other algorithms are better able to take advantage of initial order in the input.
- Data movement is minimal.
  - Selection sort uses N exchanges - the number of exchanges is a linear function of the array size.
  - None of the other sorting algorithms that we consider have this property (most involve linearithmic or quadratic growth).



#### Insertion sort

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0251-01.jpg)



##### Propositon B

>  Insertion sort uses ~$$N^2/4$$ compares and ~$$N^2/4$$ exchanges to sort a randomly ordered array of length N with distinct keys, on the average. The worst case is ~$$N^2/2$$ compares and ~$$N^2/2$$ exchanges and the best case is $$N − 1$$ compares and $$0$$ exchanges.

​	*Just as for [PROPOSITION A](#proposition-a),  the number of compares and exchanges is easy to visualize in the N-by-N 	diagram that we use to illustrate the sort. We count entries below the diagonal—all of them, in the worst case, and none of them, in the best case. For randomly ordered arrays, we expect each item to go about halfway back, on the average, so we count one-half of the entries below the diagonal.*

​	*The number of compares is the number of exchanges plus an additional term equal to N minus the number of times the item inserted is the smallest so far. In the worst case (array in reverse order), this term is negligible in relation to the total; in the best case (array in order) it is equal to $$N − 1$$.*



- Insertion sort works well for certain types of nonrandom arrays that often arise in practice, even if they are huge. 
  - For example, as just mentioned, consider what happens when you use insertion sort on an array that is already sorted. Each item is immediately determined to be in its proper place in the array, and the total running time is linear. (The running time of selection sort is quadratic for such an array.) The same is true for arrays whose keys are all equal (hence the condition in PROPOSITION B that the keys must be distinct).
- Insertion sort is an efficient method for partiaally sorted arrays; selection sort is not. 
  - Indeed, when the number of inversions is low, insertion sort is likely to be faster than any sorting method that we consider in this chapter.

> Partially Sorted Arrays.
>
> An inversion is a pair of entries that are out of order in the array. For instance, E X A M P L E has 11 inversions: E-A, X-A, X-M, X-P, X-L, X-E, M-L, M-E, P-L, P-E, and L-E. 
>
> If the number of inversions in an array is less than a constant multiple of the array size, we say that the array is partially sorted. Typical examples of partially sorted arrays are the following:
>
> ​	• An array where each entry is not far from its final position
>
> ​	• A small array appended to a large sorted array
>
> ​	• An array with only a few entries that are not in place



##### Proposition C.

> The number of exchanges used by insertion sort is equal to the number of inversions in the array, and the number of compares is at least equal to the number of inversions and at most equal to the number of inversions plus the array size minus 1.

​	*Every exchange involves two inverted adjacent entries and thus reduces the number of inversions by one, and the array is sorted when the number of inversions reaches zero.*

​	*Every exchange corresponds to a compare, and an additional compare might happen for each value of i from 1 to N-1 (when a[i] does not reach the left end of the array).*

- Insertion sort could be speeded up by shortening its inner loop, move the larger entries to the right one position rather than doing full exchanges.



#### Shell sort

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0259-01.jpg)

- Shell sort is a simple extension of insertion sort that gains speed by allowing exchanges of array entries that are far apart, to produce partially sorted arrays that can be efficiently sorted, eventually by insertion sort.

  - The idea is to rearrange the array to give it the property that taking every hth entry (starting anywhere) yields a sorted subsequence. Such an array is said to be h-sorted. 
  - ![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_04-shell3sorted.jpg)
  - Put another way, an h-sorted array is h independent sorted subsequences, interleaved together. By h-sorting for some large values of h, we can move items in the array long distances and thus make it easier to h-sort for smaller values of h. Using such a procedure for any sequence of values of h that ends in 1 will produce a sorted array: that is shellsort. The implementation in ALGORITHM uses the sequence of decreasing values ½(3k−1), starting at the smallest increment greater than or equal to ⌊N/3⌋ and decreasing to 1. We refer to such a sequence as an increment sequence ALGORITHM  computes its increment sequence; another alternative is to store an increment sequence in an array.

- How do we decide what increment sequence to use? In general, there is no clear answer.

  ![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_06-shell.jpg)

- Shellsort is much faster than insertion sort and selection sort, and its speed advantage increases with the array size

---

##### Q&A

1. *Which method runs faster for an array with all keys identical, selection sort or insertion sort?*

   Insertion sort because it will only make one comparison with the previous element (per element) and won't exchange any elements, running in linear time. Selection sort won't exchange any elements, but will run in quadratic time.

2. *Which method runs faster for an array in reverse order, selection sort or insertion sort?*

   Selection sort because even though both selection sort and insertion sort will run in quadratic time, selection sort will only make $$N-1$$ exchanges, while insertion sort will make $$N^2/ 2$$ exchanges.

3. *Suppose that we use insertion sort on a randomly ordered array where items have only one of three values. Is the running time linear, quadratic, or something in between?*

   Quadratic. Insertion sort's running time is linear when the array is already sorted or all elements are equal. With three possible values the running time quadratic.

4. *Why not use selection sort for h-sorting in shellsort?*

   Insertion sort is faster than selection sort for h-sorting because as "h" decreases, the array becomes partially sorted. Insertion sort makes less comparisons in partially sorted arrays than selection sort.

5. *Deck sort. Explain how you would put a deck of cards in order by suit (in the order spades, hearts, clubs, diamonds) and by rank within each suit, with the restriction that the cards must be laid out face down in a row, and the only allowed operations are to check the values of two cards and to exchange two cards (keeping them face down).*

   So let's modify bubblesort so that a couple things change to fit the deck of cards scenario. Let's not think of the deck of cards as a deck, but more as a list. Yes, the first card in the deck will constantly be changing with each iteration of our modified bubblesort but can we make it so that cards are shifting while still maintaining the position of the first card in the deck? This question is to key to solving the problem. What I am saying is that moving the cards to the bottom of the deck will not change in the initial order of the cards, only swapping will. For example, consider this deck of cards where leftmost is the top and rightmost is the bottom: 

   NOTE: (*) *signifies marked card*

   ```
   *5 3 1 2 6 

   ```

   In the algorithm that will later be explained, moving 5 to the bottom of the deck will make the deck look like this

   ```
   3 1 2 6 *5

   ```

   Notice how 5 is now at the bottom of the deck, but the order is still preserved. The * symbol signifies the first card in the list/deck, so if read from left to right, starting from 5 and looping back to 3, the order is preserved.

   Now to the algorithm, how do we use what I just said to make this modified version of bubblesort as similar to the original? Simple, use this marking mechanism to make it so that we're not really sorting a deck, but rather a list of numbers. Before starting the algorithm, mark the top card in the deck. Here are the rest of the steps for each iteration of bubblesort:

   1. Compare the top card with the card below it. If the top card is greater, then swap with the card below. If the marked card was swapped, then unmark the previously marked card and mark the new card at the top of the deck.
   2. Place the top card of the deck to the bottom.
   3. Steps 1 and 2 are repeated until the marked card resurfaces to being the second card in the deck (card below the top card)
   4. Put the top card to the bottom of the deck, making the marked card the top card in the deck.

   These steps are repeated for each iteration of the algorithm until an iteration occurs where no swaps were made. Here is an example to showcase the modified bubblesort: 

   NOTE: (*) *signifies marked card*

   Iteration One:

   ```
   5 3 1 2 6 //Mark the first card in the deck before starting
   *5 3 1 2 6 //Compare 5(top card) with 3(card below it)
   3 *5 1 2 6 //5 > 3 so swap
   *3 5 1 2 6 //Since the marked card (5) was swapped, 3 becomes the new marked 
              //card to preserve original order of the deck
   5 1 2 6 *3 //Top card is placed at the bottom
   1 5 2 6 *3 //5 > 1 so swap
   5 2 6 *3 1 //Put 1 at the bottom
   2 5 6 *3 1 //5 > 2 so swap
   5 6 *3 1 2 //Put 2 at the bottom
   5 6 *3 1 2 //5 < 6 so no swap
   6 *3 1 2 5 //Put 5 at the bottom
   *3 1 2 5 6 //Marked card is second to top card, so put 6 at the bottom
              //Marked card is now at the top, so new iteration begins

   ```

   Before going into iteration two, I would like to point out that if you ran the original bubblesort, the resulting sequence from one iteration would be the same as the resulting sequence from one iteration of our modified algorithm.

   Iteration Two:

   ```
   *3 1 2 5 6 //3 > 1, so swap
   *1 3 2 5 6 //Remark accordingly since the former marked card was swapped
   3 2 5 6 *1 //Place 1 at the bottom
   2 3 5 6 *1 //3 > 2, so swap
   3 5 6 *1 2 //Place 2 at the bottom
   3 5 6 *1 2 //3 < 5 so no swap
   5 6 *1 2 3 //Place 3 at the bottom
   5 6 *1 2 3 //5 < 6 so no swap
   6 *1 2 3 5 //Place 5 at the bottom.
   *1 2 3 5 6 //Since marked card is second to top card, place 6 at the bottom and end iteration

   ```

   Iteration Three:

   ```
   *1 2 3 5 6 //1 < 2 so no swap
   2 3 5 6 *1 //Place 1 at the bottom
   3 5 6 *1 2 //2 < 3 so no swap and place 2 at the bottom
   5 6 *1 2 3 //3 < 5 so no swap and place 3 at the bottom
   6 *1 2 3 5 //5 < 6 so no swap and place 5 at the bottom
   *1 2 3 5 6 //Since marked card is second to top card, place 6 at the bottom and end iteration.

   ```

   We now know to end the algorithm because an entire iteration has occurred without any swaps occurring, so the deck is now sorted. As for runtime, the worst case occurs, in terms of swaps, when the max number of iterations occurs which is n (size of the deck) times. And for each iteration, the worst case number of swaps occurs, which is also n times. So the big O is n*n or O(n^2).

6. *Expensive exchange. A clerk at a shipping company is charged with the task of rearranging a number of large crates in order of the time they are to be shipped out. Thus, the cost of compares is very low (just look at the labels) relative to the cost of exchanges (move the crates). The warehouse is nearly full—there is extra space sufficient to hold any one of the crates, but not two. What sorting method should the clerk use?*

   The clerk should use selection sort, since the cost of compares is relatively low, compared to cost of exchanges. It will take atmost N-1 exchanges.

7. *`Insertion sort with sentinel`. Develop an implementation of insertion sort that eliminates the j>0 test in the inner loop by first putting the smallest item into position. Use SortCompare to evaluate the effectiveness of doing so. Note: It is often possible to avoid an index-out-of-bounds test in this way—the element that enables the test to be eliminated is known as a sentinel.*

   ```java
   private static void insertionSortSentinel(Comparable[] array) {

           Comparable minimumElement = Double.MAX_VALUE;
           int minimumIndex = -1;

           for(int i = 0; i < array.length; i++) {
               if (array[i].compareTo(minimumElement) < 0) {
                   minimumElement = array[i];
                   minimumIndex = i;
               }
           }

           //Move smallest element to the first position
           Comparable temp = array[0];
           array[0] = array[minimumIndex];
           array[minimumIndex] = temp;

           for(int i = 1; i < array.length; i++) {
               for(int j = i; array[j].compareTo(array[j - 1]) < 0; j--) {
                   Comparable temp2 = array[j];
                   array[j] = array[j - 1];
                   array[j - 1] = temp2;
               }
           }
   }
   ```

   Insertion sort with senitel is slightly slower than the default insertion sort.

8. *Insertion sort without exchanges. Develop an implementation of insertion sort that moves larger elements to the right one position with one array access per entry, rather than using exch(). Use SortCompare to evaluate the effectiveness of doing so.*

   ```java
       private static void insertionSortWithoutExchanges(Comparable[] array) {

           for(int i = 0; i < array.length; i++) {
               Comparable aux = array[i];

               int j;

               for(j = i; j > 0 && aux.compareTo(array[j - 1]) < 0; j--) {
                   array[j] = array[j - 1];
               }

               array[j] = aux;
           }

       }

   ```



#### Merge Sort

##### Top-up merge sort

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0271-01.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0273-01.jpg)

- The primary drawback of mergesort is that it requires extra space proportional to N, for the auxiliary array for merging. If space is at a premium, we need to consider another method. On the other hand, we can cut the running time of mergesort substantially with some carefully considered modifications to the implementation.

  - Use insertion sort for small subarrays (say length 15).

    - We can improve most recursive algorithms by handling small cases differently, because the recursion guarantees that the method will be used often for small cases, so improvements in handling them lead to improvements in the whole algorithm. In the case of sorting, we know that insertion sort (or selection sort) is simple and therefore likely to be faster than mergesort for tiny subarrays

  - Test whether the array is already in order

    - We can reduce the running time to be linear for arrays that are already in order by adding a test to skip the call to merge() if a[mid] is less than or equal to a[mid+1]. With this change, we still do all the recursive calls, but the running time for any sorted subarray is linear

      ```java
          private static void topDownMergeSort(Comparable[] array, Comparable[] aux, int low, int high) {

              //Improvement #1 - Cutoff for small arrays
              if (high - low <= CUTOFF) {
                  insertionSort(aux, low, high);
                  return;
              }

              int middle = low + (high - low) / 2;

              topDownMergeSort(aux, array, low, middle);
              topDownMergeSort(aux, array, middle + 1, high);

              //Improvement #2 - Skip merge if the array is already in order
              if (array[middle].compareTo(array[middle + 1]) < 0) {
                  System.arraycopy(array, low, aux, low, high - low + 1);
                  return;
              }

              merge(array, aux, low, middle, high);
          }

      ```

      ​

  - Eliminate the copy to the auxiliary array

    - It is possible to eliminate the time (but not the space) taken to copy to the auxiliary array used for merging. To do so, we use two invocations of the sort method: one takes its input from the given array and puts the sorted output in the auxiliary array; the other takes its input from the auxiliary array and puts the sorted output in the given array. With this approach, in a bit of recursive trickery, we can arrange the recursive calls such that the computation switches the roles of the input array and the auxiliary array at each level.

      ```java
          private static void topDownMergeSort(Comparable[] array) {
              //Improvement #3 - Eliminate the copy to the auxiliary array on merge
              Comparable[] aux = array.clone();

              topDownMergeSort(aux, array, 0, array.length - 1);
          }
      ```

      ​

##### Bottom-up Merge sort

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0278-01.jpg)

- Even though we are thinking in terms of merging together two large subarrays, the fact is that most merges are merging together tiny subarrays. Another way to implement mergesort is to organize the merges so that we do all the merges of tiny subarrays on one pass, then do a second pass to merge those subarrays in pairs, and so forth, continuing until we do a merge that encompasses the whole array. This method requires even less code than the standard recursive implementation. We start by doing a pass of 1-by-1 merges (considering individual items as subarrays of size 1), then a pass of 2-by-2 merges (merge subarrays of size 2 to make subarrays of size 4), then 4-by-4 merges, and so forth. The second subarray may be smaller than the first in the last merge on each pass (which is no problem for merge()), but otherwise all merges involve subarrays of equal size, doubling the sorted subarray size for the next pass.

  ![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_15-mergeaiibu.jpg)

- Bottom-up merge sort is suited for sorting of linked lists.



##### Q&A

1. *Count number of inversions in an array.*

   - One approach is two for loops ~$$N^2$$ .
   - Use modified merge sort..(inversions in left half) + (inversions in right half) + (inversions while merging) ~$$NlogN$$ .

   ```java
      static int _mergeSort(int arr[], int temp[], int left, int right) 
       { 
         int mid, inv_count = 0; 
         if (right > left) 
         { 
           /* Divide the array into two parts and call _mergeSortAndCountInv() 
              for each of the parts */
           mid = (right + left)/2; 
          
           /* Inversion count will be sum of inversions in left-part, right-part 
             and number of inversions in merging */
           inv_count  = _mergeSort(arr, temp, left, mid); 
           inv_count += _mergeSort(arr, temp, mid+1, right); 
          
           /*Merge the two parts*/
           inv_count += merge(arr, temp, left, mid+1, right); 
         } 
         return inv_count; 
       } 
          
       /* This method merges two sorted arrays and returns inversion count in 
          the arrays.*/
       static int merge(int arr[], int temp[], int left, int mid, int right) 
       { 
         int i, j, k; 
         int inv_count = 0; 
          
         i = left; /* i is index for left subarray*/
         j = mid;  /* j is index for right subarray*/
         k = left; /* k is index for resultant merged subarray*/
         while ((i <= mid - 1) && (j <= right)) 
         { 
           if (arr[i] <= arr[j]) 
           { 
             temp[k++] = arr[i++]; 
           } 
           else
           { 
             temp[k++] = arr[j++]; 
          
            /*this is tricky -- see above explanation/diagram for merge()*/
             inv_count = inv_count + (mid - i); 
           } 
         } 
          
         /* Copy the remaining elements of left subarray 
          (if there are any) to temp*/
         while (i <= mid - 1) 
           temp[k++] = arr[i++]; 
          
         /* Copy the remaining elements of right subarray 
          (if there are any) to temp*/
         while (j <= right) 
           temp[k++] = arr[j++]; 
          
         /*Copy back the merged elements to original array*/
         for (i=left; i <= right; i++) 
           arr[i] = temp[i]; 
          
         return inv_count; 
       } 
   ```

   2. *Index sort. Develop and implement a version of mergesort that does not rearrange the array, but returns an int[] array perm such that perm[i] is the index of the ith smallest entry in the array.*

      ```java
          private static int[] indexSort(Comparable[] array) {
              int[] aux = new int[array.length];
              int[] indexSort = new int[array.length];

              for(int i = 0; i < array.length; i++) {
                  indexSort[i] = i;
              }

              indexSort(array, aux, indexSort, 0, array.length - 1);

              return indexSort;
          }

          private static void indexSort(Comparable[] array, int[] aux, int[] indexSort, int low, int high) {

              if (low >= high) {
                  return;
              }

              int middle = low + (high - low) / 2;

              indexSort(array, aux, indexSort, low, middle);
              indexSort(array, aux, indexSort, middle + 1, high);

              merge(array, aux, indexSort, low, middle, high);
          }

          @SuppressWarnings("unchecked")
          private static void merge(Comparable[] array, int[] aux, int[] indexSort, int low, int middle, int high) {

              for(int i = low; i <= high; i++) {
                  aux[i] = indexSort[i];
              }

              int leftIndex = low;
              int rightIndex = middle + 1;
              int arrayIndex = low;

              while (leftIndex <= middle && rightIndex <= high) {

                  if (array[aux[leftIndex]].compareTo(array[aux[rightIndex]]) <= 0) {
                      indexSort[arrayIndex] = aux[leftIndex];

                      leftIndex++;
                  } else {
                      indexSort[arrayIndex] = aux[rightIndex];
                      rightIndex++;
                  }

                  arrayIndex++;
              }

              while (leftIndex <= middle) {
                  indexSort[arrayIndex] = aux[leftIndex];
                  leftIndex++;
                  arrayIndex++;
              }
      }
      ```

   3. *3-way mergesort. Suppose instead of dividing in half at each step, you divide into thirds, sort each third, and combine using a 3-way merge. What is the order of growth of the overall running time of this algorithm?*

      An algorithm that divides the array into thirds on each step, and combine the subarrays using a 3-way merge has an order of growth of the running time of O(n log n), just like the regular mergesort, except for the fact that the log is base 3, instead of base 2.

   4. *Compare the implementation of merge sort with improvemnets* ?

      All of the improvements had better performance than the baseline mergesort, except for the improvement of testing if the array is already sorted, which had a similar performance. 
      The best results were achieved for the implementations in the following order:

      1- Improvement #3 - Avoid array copy by switching arguments
      2- Improvement #1 - Cutoff for small subarrays
      3- Copy second half of subarray in decreasing order during merge
      4- Improvement #2 - Test if array is already sorted to avoid merge
      4- Baseline top down mergesort

   5. *K-way- merge sort, suppose you are given k sorted arrays, with n elements each, discuss a best way to merge it?*

      - merge first and second then merge the third with the o/p of first and second, The first merge takes 2n, the second 3n, the third 4n...

        We are then given a summation 2n + 3n + 4n + 5n +6n + ... + kn

        Which is equal to n*(2+3+4+...k)

        Which is equal to n*(k*(k+1)/2 - 1)

        Which implies a runtime of n*k^2

        A faster solution would be to not merge successively. Merge all n until you get k/2 * 2n. Like merge sort, we merge evenly sized arrays at every step. This would yield a solution with the complexity of mergesort with K elements which is n*k log k. The amount of space is O(k*n)

        Given K arrays with N elements each. First take the smallest value of each array and put it into the heap. Keep track of which array each value came from. Then pop the smallest value off the array and put this into the final array. Then insert another value from the array the pop came from. The runtime for this is also k*n*log k since we have to go through all k*n elements. Insertion into a k size heap is log k and pop is O(1).

#### QuickSort

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0289-01.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0291-01.jpg)



- Quicksort is complementary to mergesort: 
  - for mergesort, we break the array into two subarrays to be sorted and then combine the ordered subarrays to make the whole ordered array; 
  - for quicksort, we rearrange the array such that, when the two subarrays are sorted, the whole array is ordered. 
  - In the first instance, we do the two recursive calls before working on the whole array; in the second instance, we do the two recursive calls after working on the whole array. 
  - For mergesort, the array is divided in half; for quicksort, the position of the partition depends on the contents of the array.

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_24-partitionex.jpg)

- Partitoning in place

  - If we use an extra array, partitioning is easy to implement, but not so much easier that it is worth the extra cost of copying the partitioned version back into the original.

- Staying in bounds

  - If the smallest item or the largest item in the array is the partitioning item, we have to take care that the pointers do not run off the left or right ends of the array, respectively. Our partition() implementation has explicit tests to guard against this circumstance. The test (j == lo) is redundant, since the partitioning item is at a[lo] and not less than itself. With a similar technique on the right it is not difficult to eliminate both tests

- Preserving randomness

  - The random shuffle puts the array in random order. Since it treats all items in the subarrays uniforml, has the property that its two subarrays are also in random order.
  - Another way of preserving randomness is  to choose a random item for partitioning within partition().

- Handling items with keys equal to the partitioning item’s key

  - It is best to stop the left scan for items with keys greater than or equal to the partitioning item’s key and the right scan for items with key less than or equal to the partitioning item’s key, as in above algo. Even though this policy might seem to create unnecessary exchanges involving items with keys equal to the partitioning item’s key, it is crucial to avoiding quadratic running time in certain typical applications

- The best case for quicksort is when each partitioning stage divides the array exactly in half. This circumstance would make the number of compares used by quicksort satisfy the divide-and-conquer recurrence C~N~ = 2C~N/2~ + N. 

- Despite its many assets, the basic quicksort program has one potential liability: it can be extremely inefficient if the partitions are unbalanced. For example, it could be the case that the first partition is on the smallest item, the second partition on the next smallest item, and so forth, so that the program will remove just one item for each call, leading to an excessive number of partitions of large subarrays. Avoiding this situation is the primary reason that we randomly shuffle the array before using quicksort. This action makes it so unlikely that bad partitions will happen consistently that we need not worry about the possibility.

- Improvements to quicksort :-

  - Cutoff to insertion sort :-

    ```java
    //replace the following line 
    if(hi <= lo); return;

    //with
    if(hi <= lo +M);
      Insertion.sort(a, lo, hi);
      return;

    // the optimum value of cutoff M is sytsem-dependent but any value between 5 and 15 is likely to work.
    ```

  - Median-of-three partitioning :-

    Quicksort with median-of-three partitioning functions nearly the same as normal quicksort with the only difference being how the pivot item is selected. In normal quicksort the first element is automatically the pivot item. This causes normal quicksort to function very inefficiently when presented with an already sorted list. The divison will always end up producing one sub-array with no elements and one with all the elements (minus of course the pivot item). In quicksort with median-of-three partitioning the pivot item is selected as the median between the first element, the last element, and the middle element (decided using integer division of n/2). In the cases of already sorted lists this should take the middle element as the pivot thereby reducing the inefficency found in normal quicksort.

  - With Senitels :-

    We can modify the code in above to remove both bounds checks in the inner while loops. The test against the left end of the subarray is redundant since the partitioning item acts as a sentinel (v is never less than a[lo]). To enable removal of the other test, put an item whose key is the largest in the whole array into a[length-1] just after the shuffle. This item will never move (except possibly to be swapped with an item having the same key) and will serve as a sentinel in all subarrays involving the end of the array. Note: For a subarray that does not involve the end of the array, the leftmost entry to its right serves as a sentinel for the right end of the subarray.

  - Entropy optimal sorting :-

     In a situation where there are large numbers of duplicate keys in the input array, the recursive nature of quicksort ensures that subarrays consisting solely of items with keys that are equal will occur often. There is potential for significant improvement.

    One straightforward idea is to partition the array into three parts, one each for items with keys smaller than, equal to, and larger than the partitioning item’s key. Accomplishing this partitioning is more complicated than the 2-way partitioning that we have been using, and various different methods have been suggested for the task. It was a classical programming exercise popularized by **E. W. Dijkstra as the Dutch National Flag problem**, because it is like sorting an array with three possible key values, which might correspond to the three colors on the flag.

    >Dijkstra’s solution to this problem leads to the remarkably simple partition code shown on the next page. It is based on a single left-to-right pass through the array that maintains a pointer lt such that a[lo..lt-1] is less than v, a pointer gt such that a[gt+1.. hi] is greater than v, and a pointer i such that a[lt..i-1] are equal to v and a[i..gt] are not yet examined.
    >
    >Starting with i equal to lo, we process a[i] using the 3-way comparison given by the Comparable interface (instead of using less()) to directly handle the three possible cases:
    >
    >• a[i] less than v: exchange a[lt] with a[i] and increment both lt and i
    >
    >• a[i] greater than v: exchange a[i] with a[gt] and decrement gt
    >
    >• a[i] equal to v: increment i

    ![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0299-01.jpg)

    This sort code partitions to put keys equal to the partitioning element in place and thus does not have to include those keys in the subarrays for the recursive calls. It is far more efficient than the standard quicksort implementation for arrays with large numbers of duplicate keys

    ![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_27-partition3way.jpg)

##### Q&A

1. *What is the maximum number of times during the execution of Quick.sort() that the largest item can be exchanged, for an array of length N?*

   N-1 times

2. *Suppose that the initial random shuffle is omitted. Give six arrays of ten elements for which Quick.sort() uses the worst-case number of compares.*

   [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
   [2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
   [3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
   [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
   [11, 10, 9, 8, 7, 6, 5, 4, 3, 2]
   [12, 11, 10, 9, 8, 7, 6, 5, 4, 3]

3. *Give a code fragment that sorts an array that is known to consist of items having just two distinct keys*

   Use dutch flag algorithm

4. *About how many compares will Quick.sort() make when sorting an array of N items that are all equal?*

   When sorting an array of N items that are all equal, Quick.sort() will make approximately N lg N compares. Each partition will divide the array in half, plus or minus one.

5. *Explain what happens when Quick.sort() is run on an array having items with just two distinct keys, and then explain what happens when it is run on an array having just three distinct key*

   On both cases (when Quick.sort() is run on arrays having items with just two or three distinct keys) there is a high occurrence of subarrays consisting solely of items with equal keys. To improve performance when sorting such arrays (from linearithmic to linear) quicksort with 3-way partitioning should be used.

6. *Suppose that we scan over items with keys equal to the partitioning item’s key instead of stopping the scans when we encounter them. Show that the running time of this version of quicksort is quadratic for all arrays with just a constant number of distinct keys.*

   You have to measure the sorts programatically.

7. *Given a set of n nuts of different sizes and n bolts of different sizes. There is a one-one mapping between nuts and bolts. Match nuts and  bolts efficiently.* 

   **Constraint:** *Comparison of a nut to another nut or a bolt to another bolt is not allowed. It means nut can only be compared with bolt and bolt can only be compared with nut to see which one is bigger/smaller.*

   *Other way of asking this problem is, given a box with locks and keys where one lock can be opened by one key in the box. We need to match the pair.*

   Since each nut matches to exactly one bolt, the array of nuts and the array of bolts have distinct elements.

   I would use the nuts as pivots, so to avoid a O(N ^ 2) complexity, initially I would shuffle the nuts array.

   I would use a similar method to Quicksorts part'ition to place the bolts in their correct location.
   Using one nut as the pivot at a time, I would partition the bolts array. This would result in the bolts array with a bolt (matching the nut pivot) in the correct place, all bolts smaller than it on its left and all bolts bigger than it on its right.

   Instead of removing it, though, I would leave the matched bolt in its correct place and store the information of its location and the corresponding nut in a hashmap, to match them at end of the algorithm.

   I would then choose the next nut as the pivot. However, in this case, I would first compare it with the bolt found in the previous iteration of the algorithm, to know in which subarray to check (either the left or the right), depending on whether the bolt is smaller or bigger than the current pivot nut. I would then repeat the previous process.

   I would repeat this steps for every nut, doing a binary search on the already found bolts to reduce the bolts subarray to search.
   At the end, I would iterate over the hashmap and connect all nuts and bolts.

   This method would yield a complexity of O(N lg lg N).
   N operations for the pivot method, lg N times, using a binary search to find the correct subarray (lg n).

8. *Median-of -three partitioning in quicksort*

   ```java
      private static void quickSort(Comparable[] array, int low, int high) {

           if (low >= high) {
               return;
           }

           int partition = partition(array, low, high);
           quickSort(array, low, partition - 1);
           quickSort(array, partition + 1, high);
       }

       private static int partition(Comparable[] array, int low, int high) {
           int middle = low + (high - low) / 2;

           //sort array[low], array[mid], array[high]

           if (ArrayUtil.less(array[middle], array[low])) {
               ArrayUtil.exchange(array, middle, low);
           }
           if (ArrayUtil.less(array[high], array[low])) {
               ArrayUtil.exchange(array, high, low);
           }
           if (ArrayUtil.less(array[high], array[middle])) {
               ArrayUtil.exchange(array, high, middle);
           }

           //Swap median with low
           ArrayUtil.exchange(array, middle, low);
           Comparable pivot = array[low];

           int i = low;
           int j = high + 1;

           while(true) {
               while (ArrayUtil.less(array[++i], pivot));

               while(ArrayUtil.less(pivot, array[--j]));

               if (i >= j) {
                   break;
               }

               ArrayUtil.exchange(array, i, j);
           }

           //Place pivot in the right place
           ArrayUtil.exchange(array, low, j);
           return j;
       }

   ```

   It runs faster than default quick sort algorithm.

9. *implement quicksort non-recursively*

   ```java
       private static void quickSort(Comparable[] array, int low, int high) {

           Stack<QuickSortRange> stack = new Stack<>();

           QuickSortRange quickSortRange = new QuickSortRange(low, high);
           stack.push(quickSortRange);

           while (stack.size() > 0) {
             //QuicksortRange{low, high}
               QuickSortRange currentQuickSortRange = stack.pop();

               int partition = partition(array, currentQuickSortRange.low, currentQuickSortRange.high);
               QuickSortRange leftQuickSortRange = new QuickSortRange(currentQuickSortRange.low, partition - 1);
               QuickSortRange rightQuickSortRange = new QuickSortRange(partition + 1, currentQuickSortRange.high);

               //Size = right - left + 1
               int leftSubArraySize = partition - currentQuickSortRange.low;
               int rightSubArraySize = currentQuickSortRange.high - partition + 2;

               //Push the larger sub array first to guarantee that the stack will have at most lg N entries
               if (leftSubArraySize > rightSubArraySize) {
                   if (leftSubArraySize > 1 && leftQuickSortRange.low < leftQuickSortRange.high) {
                       stack.push(leftQuickSortRange);
                   }

                   if (rightSubArraySize > 1 && rightQuickSortRange.low < rightQuickSortRange.high) {
                       stack.push(rightQuickSortRange);
                   }
               } else {
                   if (rightSubArraySize > 1 && rightQuickSortRange.low < rightQuickSortRange.high) {
                       stack.push(rightQuickSortRange);
                   }

                   if (leftSubArraySize > 1 && leftQuickSortRange.low < leftQuickSortRange.high) {
                       stack.push(leftQuickSortRange);
                   }
               }
           }
   }
   ```



#### Priority Queues

- We implement PQs using heap-ordered binary trees. There are two properties of heap ordered binary trees :-
  1. A heap ordered binary tree should be a complete binary tree. (Fill each levels from left to right).
  2. The key in each node is larger than or equal to the keys in that node’s two children (if any).

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0315-01.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0316-01.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0316-02.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0318-01.jpg)

- Multiway heaps:-
  - It is not difficult to modify our code to build heaps based on an array representation of complete heap-ordered ternary trees, with an entry at position k larger than or equal to entries at positions 3k−1, 3k, and 3k+1 and smaller than or equal to entries at position image(k+1) / 3image, for all indices between 1 and N in an array of N items, and not much more difficult to use d-ary heaps for any given d.
  - There is a tradeoff between the lower cost from the reduced tree height (logd N) and the higher cost of finding the largest of the d children at each node. This tradeoff is dependent on details of the implementation and the expected relative frequency of operations.
- Immutability of keys on which sorting is done should be maintaned:-
  - The priority queue contains objects that are created by clients but assumes that client code does not change the keys (which might invalidate the heap-order invariant). It is possible to develop mechanisms to enforce this assumption, but programmers typically do not do so because they complicate the code and are likely to degrade performance.



#### HeapSort

1. First build a heap-ordered data using the unordered array.
2. Once the largest item is at the root node and evry parent node value is higher than it's coressponding children's values, then we'll swap the largest value with the item at the end of the heap.
3. Then run sink func(or heapify func) on the root node, because after step 2, the last item might be in the right place but root might not be. We'll have to move the root item to it's right place.

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/p0324-01.jpg)

![image](https://www.safaribooksonline.com/library/view/algorithms-fourth-edition/9780132762564/graphics/02_35-heapselection.jpg)

- *sortdown* :

  Most of the work during heapsort is done during the second phase, where we remove the largest remaining item from the heap and put it into the array position vacated as the heap shrinks. This process is a bit like selection sort , but it uses many fewer compares because the heap provides a much more efficient way to find the largest item in the unsorted part of the array.

- When space is very tight (for example, in an embedded system or on a low-cost mobile device) it is popular because it can be implemented with just a few dozen lines (even in machine code) while still providing optimal performance. However, it is rarely used in typical applications on modern systems because it has poor cache performance: array entries are rarely compared with nearby array entries, so the number of cache misses is far higher than for quicksort, mergesort, and even shellsort, where most compares are with nearby entries.



##### Q&A

1. *Criticize the following idea: To implement find the maximum in constant time, why not use a stack or a queue, but keep track of the maximum value inserted so far, then return that value for find the maximum?*

   It only keeps the current max, say if current max is extracted out, we need to do a 0(n) operation to find the next maximum.

2. *The largest item in a heap must appear in position 1, and the second largest must be in position 2 or position 3. Give the list of positions in a heap of size 31 where the kth largest (i) can appear, and (ii) cannot appear, for k=2, 3, 4 (assuming the values to be distinct).*

   ```
   Heap-of-size-31 positions

                           1
              2                         3
         4           5             6            7
     8     9     10      11    12     13    14    15
   16 17 18 19  20 21  22 23  24 25  26 27 28 29 30 31

   Kth largest item  Can appear                     Cannot appear
   2                 2,3                          1,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31
   3                 2,3,4,5,6,7                    1,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31
   4                 4,5,6,7,8,9,10,11,12,13,14,15  1,2,3,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31
   ```

   3. *Suppose that we wish to avoid wasting one position in a heap-ordered array pq[], putting the largest value in pq[0], its children in pq[1] and pq[2], and so forth, proceeding in level order. Where are the parents and children of pq[k]?*

   Parent of pq[k]: pq[ (k - 1) / 2] (Rounded down)

   Children of pq[k]:
   Left: pq[k * 2 + 1]
   Right: pq[k * 2 + 2]

3. *Suppose that your application will have a huge number of find the maximum operations, but a relatively small number of insert and remove the maximum operations. Which priority-queue implementation do you think would be most effective: heap, unordered array, or ordered array?*

   Unordered one as insertion will take O(1) while for heap its O(log n) and for ordered its O(n).

4. *Suppose that your application will have a huge number of find the maximum operations, but a relatively small number of insert and remove the maximum operations. Which priority-queue implementation do you think would be most effective: heap, unordered array, or ordered array?*

   Orederd array for which it would be  O(1).

5. *Because the exch() primitive is used in the sink() and swim() operations, the items are loaded and stored twice as often as necessary. Give more efficient implementations that avoid this inefficiency, a la insertion sort*

   Use similar mechanism as of insertion sort, where instead excahnging each element, move elements to right. 

6. *Find min() in MAXPQ ?*

   Idea is simple, while inserting in max priority queue keep hold of min, and while deleting if the the element is min, then min() should return null.

   ```java
          public void insert(Key key) {
               if (size != priorityQueue.length - 1) {
                   size++;

                   if (min == null || ArrayUtil.less(key, min)) {
                       min = key;
                   }

                   priorityQueue[size] = key;
                   swim(size);
               }
           }

           Key deleteMax() {
               if (size == 0) {
                   throw new RuntimeException("Priority queue underflow");
               }

               size--;

               Key max = priorityQueue[1];
               ArrayUtil.exchange(priorityQueue, 1, size + 1);

               if (max == min) {
                   min = null;
               }

               priorityQueue[size + 1] = null;
               sink(1);

               return max;
           }

   ```

7. *Min/max priority queue. Design a data type that supports the following operations: insert, delete the maximum, and delete the minimum (all in logarithmic time); and find the maximum and find the minimum (both in constant time)*

   The idea is to use two heaps, minHeap and maxHeap, keep track of the indexes of an item in both the heaps, while inserting the item in any of the heaps, update both the indexes, while deleting the item, delete the top item depending on deleteMax() or deleteMin() , delete the top item from the respettive heap(min/max) and also delete the corresponding indexed item from the second heap.

   ```java
   public class Exercise29_MinMaxPriorityQueue  {

       private enum Orientation {
           MAX, MIN;
       }

       private class PQNode implements Comparable<PQNode>{
           Comparable key;
           int minHeapIndex;
           int maxHeapIndex;

           @Override
           public int compareTo(PQNode other) {
               return key.compareTo(other.key);
           }
       }

       private class MinMaxPriorityQueue<Key extends Comparable<Key>> {

           private PQNode[] minPriorityQueue;
           private PQNode[] maxPriorityQueue;
           private int size = 0;

           MinMaxPriorityQueue() {
               minPriorityQueue = new PQNode[2];
               maxPriorityQueue = new PQNode[2];
           }

           public boolean isEmpty() {
               return size == 0;
           }

           public int size() {
               return size;
           }

           public void insert(Key key) {

               if (size == minPriorityQueue.length - 1) {
                   resize(minPriorityQueue.length * 2);
               }

               PQNode pqNode = new PQNode();
               pqNode.key = key;

               size++;

               insertOnMinHeap(pqNode);
               insertOnMaxHeap(pqNode);
           }

           private void insertOnMinHeap(PQNode pqNode) {

               minPriorityQueue[size] = pqNode;
               swim(minPriorityQueue, size, Orientation.MIN);
           }

           private void insertOnMaxHeap(PQNode pqNode) {

               maxPriorityQueue[size] = pqNode;
               swim(maxPriorityQueue, size, Orientation.MAX);
           }

           //O(1)
           public Comparable findMin() {
               if (size == 0) {
                   return null;
               }

               return minPriorityQueue[1].key;
           }

           //O(1)
           public Comparable findMax() {
               if (size == 0) {
                   return null;
               }

               return maxPriorityQueue[1].key;
           }

           //O(lg N)
           public Comparable deleteMax() {

               if (size == 0) {
                   throw new RuntimeException("Priority queue underflow");
               }

               size--;

               PQNode max = maxPriorityQueue[1];

               deleteTopItem(maxPriorityQueue, Orientation.MAX);
               deleteItem(minPriorityQueue, Orientation.MIN, max.minHeapIndex);

               if (size == minPriorityQueue.length / 4) {
                   resize(minPriorityQueue.length / 2);
               }

               return max.key;
           }

           //O(lg N)
           public Comparable deleteMin() {

               if (size == 0) {
                   throw new RuntimeException("Priority queue underflow");
               }

               size--;

               PQNode min = minPriorityQueue[1];

               deleteTopItem(minPriorityQueue, Orientation.MIN);
               deleteItem(maxPriorityQueue, Orientation.MAX, min.maxHeapIndex);

               if (size == minPriorityQueue.length / 4) {
                   resize(minPriorityQueue.length / 2);
               }

               return min.key;
           }

           private void deleteTopItem(PQNode[] priorityQueue, Orientation orientation) {
               deleteItem(priorityQueue, orientation, 1);
           }

           private void deleteItem(PQNode[] priorityQueue, Orientation orientation, int index) {
               ArrayUtil.exchange(priorityQueue, index, size + 1);
               priorityQueue[size + 1] = null;

               if (index == size + 1) {
                   //We deleted the last value, so no need to sink
                   return;
               }

               sink(priorityQueue, index, orientation);
           }

           private void swim(PQNode[] priorityQueue, int index, Orientation orientation) {
               while(index / 2 >= 1) {
                   if ((orientation == Orientation.MAX && ArrayUtil.less(priorityQueue[index / 2], priorityQueue[index]))
                           || (orientation == Orientation.MIN && ArrayUtil.more(priorityQueue[index / 2], priorityQueue[index]))) {
                       ArrayUtil.exchange(priorityQueue, index / 2, index);

                       if (orientation == Orientation.MIN) {
                           priorityQueue[index].minHeapIndex = index;
                           priorityQueue[index / 2].minHeapIndex = index / 2;
                       } else {
                           priorityQueue[index].maxHeapIndex = index;
                           priorityQueue[index / 2].maxHeapIndex = index / 2;
                       }
                   } else {
                       break;
                   }

                   index = index / 2;
               }

               //Even if there were no exchanges, we still need to update the index
               if (orientation == Orientation.MIN) {
                   priorityQueue[index].minHeapIndex = index;
               } else {
                   priorityQueue[index].maxHeapIndex = index;
               }
           }

           private void sink(PQNode[] priorityQueue, int index, Orientation orientation) {
               while (index * 2 <= size) {
                   int selectedChildIndex = index * 2;

                   if (index * 2 + 1 <= size &&
                           (
                            (orientation == Orientation.MAX && ArrayUtil.less(priorityQueue[index * 2], priorityQueue[index * 2 + 1]))
                                 || (orientation == Orientation.MIN && ArrayUtil.more(priorityQueue[index * 2], priorityQueue[index * 2 + 1]))
                           )
                           ) {
                       selectedChildIndex = index * 2 + 1;
                   }

                   if ((orientation == Orientation.MAX && ArrayUtil.more(priorityQueue[selectedChildIndex], priorityQueue[index]))
                           || (orientation == Orientation.MIN && ArrayUtil.less(priorityQueue[selectedChildIndex], priorityQueue[index]))) {
                       ArrayUtil.exchange(priorityQueue, index, selectedChildIndex);

                       if (orientation == Orientation.MIN) {
                           priorityQueue[index].minHeapIndex = index;
                           priorityQueue[selectedChildIndex].minHeapIndex = selectedChildIndex;
                       } else {
                           priorityQueue[index].maxHeapIndex = index;
                           priorityQueue[selectedChildIndex].maxHeapIndex = selectedChildIndex;
                       }
                   } else {
                       break;
                   }

                   index = selectedChildIndex;
               }

               //Even if there were no exchanges, we still need to update the index
               if (orientation == Orientation.MIN) {
                   priorityQueue[index].minHeapIndex = index;
               } else {
                   priorityQueue[index].maxHeapIndex = index;
               }
           }

           private void resize(int newSize) {
               //Min heap
               PQNode[] newMinPriorityQueue = new PQNode[newSize];
               System.arraycopy(minPriorityQueue, 1, newMinPriorityQueue, 1, size);
               minPriorityQueue = newMinPriorityQueue;

               //Max heap
               PQNode[] newMaxPriorityQueue = new PQNode[newSize];
               System.arraycopy(maxPriorityQueue, 1, newMaxPriorityQueue, 1, size);
               maxPriorityQueue = newMaxPriorityQueue;
           }
   }
   ```

8. *Dynamic median finding*

   ```java
   public class Exercise30_DynamicMedianFinding {

       private class DynamicMedianFindingHeap<Key extends Comparable<Key>> {

           private PriorityQueueResize<Key> minPriorityQueue;
           private PriorityQueueResize<Key> maxPriorityQueue;

           private int size;

           DynamicMedianFindingHeap() {
               minPriorityQueue = new PriorityQueueResize<>(PriorityQueueResize.Orientation.MIN);
               maxPriorityQueue = new PriorityQueueResize<>(PriorityQueueResize.Orientation.MAX);
               size = 0;
           }

           //O(lg N)
           public void insert(Key key) {

               if (size == 0 || ArrayUtil.less(key, maxPriorityQueue.peek())) {
                   maxPriorityQueue.insert(key);
               } else {
                   minPriorityQueue.insert(key);
               }

               if (minPriorityQueue.size() > maxPriorityQueue.size() + 1) {
                   Key keyToBeMoved = minPriorityQueue.deleteTop();
                   maxPriorityQueue.insert(keyToBeMoved);
               } else if (maxPriorityQueue.size() > minPriorityQueue.size() + 1) {
                   Key keyToBeMoved = maxPriorityQueue.deleteTop();
                   minPriorityQueue.insert(keyToBeMoved);
               }

               size++;
           }

           //O(1)
           // if odd number of entries(1 mid index) return the middle value, if even         // no. of entries (2 mid indexes), return the left index. 
           public Key findTheMedian() {
               Key median;

               if (minPriorityQueue.size() > maxPriorityQueue.size()) {
                   median = minPriorityQueue.peek();
               } else {
                   median = maxPriorityQueue.peek();
               }

               return median;
           }

           //O(lg N)
           public Key deleteMedian() {
               Key median;

               if (minPriorityQueue.size() > maxPriorityQueue.size()) {
                   median = minPriorityQueue.deleteTop();
               } else {
                   median = maxPriorityQueue.deleteTop();
               }

               size--;

               return median;
           }
   }
   ```

9. *Improve insert operation, such that insert uses $$log log N$$ compares.*

   Observe the parent pointers in a heap are ordered (non-increasing/decreasing). We can use binary search on it to find the required ancestor.

   ```java

           private void swim(int index) {

               //No need to swim if we only have 1 element
               if (index == 1) {
                   return;
               }

               int targetAncestor = binarySearchToGetTargetAncestor(index);

               while (index / 2 >= targetAncestor) {
                   ArrayUtil.exchange(priorityQueue, index / 2, index);
                   index = index / 2;
               }
           }

           private int binarySearchToGetTargetAncestor(int index) {
               //Generate parents array
               int parentsArraySize = (int) (Math.log10(size) / Math.log10(2));
               int[] parentsIndex = new int[parentsArraySize];

               int parentsArrayIndex = parentsIndex.length - 1;
               for(int i = index / 2; i >= 1; i /= 2) {
                   parentsIndex[parentsArrayIndex] = i;
                   parentsArrayIndex--;
               }

               //Binary search
               int low = 0;
               int high = parentsIndex.length - 1;
               int middle;

               int targetAncestor = index;

               while (low < high) {
                   middle = low + (high - low) / 2;

                   numberOfCompares++;
                   if (ArrayUtil.more(priorityQueue[parentsIndex[middle]], priorityQueue[index])) {
                       high = middle;
                       targetAncestor = parentsIndex[middle];
                   } else {
                       low = middle + 1;
                   }
               }

               return targetAncestor;
   }
   ```
   ​
