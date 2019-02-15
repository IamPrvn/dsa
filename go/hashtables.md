[TOC]



# Hashtables

31 Oct 2013

I wanted to write a post answering the question *Why Does Reading From A Hashtable Require Synchronization* when I realized that a distinct introduction to hashtables would prove useful. So, here's the first of a few post dedicated to my favorite data structure.

### Introduction

I like to think of Hashtables as flexible arrays. Where arrays rigidly require continuous integer based indexes, hashtables allow non-integer, and therefore non-continuous, based indexes. What's amazing about hashtables is that, despite this extra flexibility, typical performance characteristics is largely the same as arrays. Namely, getting and setting values is O(1). In other words, getting a value from a hashtable with 5 items should perform the same as a hashtable with 5 billion items - just like an array.

This is possible because a hashtable *is* an array, with extra logic which converts the key into an integer. Consider this basic implementation:

```
//think of interface{} as "object" in many other languages

type Hash struct {
  size uint32
  buckets []interface{}
}

func New(size int) *Hash{
  return &Hash{
    size: uint32(size),
    buckets: make([]interface{}, size),
  }
}

func (h *Hash) Set(key []byte, value interface{}) {
  h.buckets[h.getIndex(key)] = value
}

func (h *Hash) getIndex(key []byte) uint32 {
  hasher := fnv.New32a()
  hasher.Write(key)
  return hasher.Sum32() % h.size
}

```

The above code is just a thinly wrapped array. We don't set a value directly into our array at a specific position. Instead, we use a hashing algorithm on our key, and mod that to fit within the array.

In order to get a value based on a key, we reuse the `getIndex` method:

```
func (h *Hash) Get(key []byte) interface{} {
  return h.buckets[h.getIndex(key)]
}

```

### Hash Algorithm

For now, all the magic of our hashtable is in the hashing algorithm. Above, we use the popular [Fowler Noll Vo (FNV) algorithm](http://en.wikipedia.org/wiki/Fowler_Noll_Vo_hash). When you think of hash algorithms, you might think of MD5 or SHA1. However, hash algorithms exhibit a number of distinct properties; the properties you want for a hashtable aren't usually the same as the properties you want for cryptographic purposes. As a clear example, cryptographic hash algorithms benefit from being computationally expensive (to limit brute forcing), whereas hash algorithms for hashtable want to be as fast as possible.

An even more relevant example is that cryptographic hash algorithms should be collision free, whereas hashtable algorithms aren't. The reason for this should be obvious. In `getIndex` we took the result of our hash, which could be a very large number, and did a modulus operation. Consider the following:

```
  hasher := fnv.New32a()
  hasher.Write([]byte("over"))
  first := hasher.Sum32() //838226447

  hasher.Reset()
  hasher.Write([]byte("9000"))
  second := hasher.Sum32() //58738199

  // second % 12 == first % 12

```

In other words, since the result of the hashing algorithm gets reduced, you'll almost certainly have to deal with collisions anyways (reducing collision is still important, it just isn't as important as with cryptographic hashes).

A lot of work has gone into various hashing algorithm. Some work specifically well with certain types of CPUs, or key types, or various other factors. While it's far from being the most exhaustive list, [here's a nice comparison](http://programmers.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed/145633#145633) of some of the more popular options.

### Collision

Except for cases when you'll know all possible keys ahead of time, collisions will happen. You might be tempted to try using a huge array. Not only is that inefficient, it also [won't work](https://en.wikipedia.org/wiki/Birthday_problem).

One strategy for dealing with collisions is to chain colliding values using another data structure - more often than not, a linked list. What this means is that we'll hash the key to get a specific bucket, but then store the key and value within a linked list: 

```
type HashValue struct {
  key []byte
  value interface{}
}

type Hash struct {
  size uint32
  buckets []*list.List
}

func New(size int) *Hash{
  h := &Hash{
    size: uint32(size),
    buckets: make([]*list.List, size),
  }
  for i := 0; i < size; i++ { h.buckets[i] = list.New() }
  return h
}

func (h *Hash) Set(key []byte, value interface{}) {
  h.buckets[h.getIndex(key)].PushBack(&HashValue{key, value})
}

```

It's important that we maintain the key since we'll need it to pull the correct value from the bucket. This is accomplished by `Get` which scans the list for the actual match:

```
func (h *Hash) Get(key []byte) interface{} {
  list := h.buckets[h.getIndex(key)]
  for element := list.Front(); element != nil; element = element.Next() {
    wrapped := element.Value.(*HashValue)
    if bytes.Equal(wrapped.key, key) {
      return wrapped.value
    }
  }
  return nil

```

Your initial reaction to this might be dismay. Haven't we just gone from O(1), back to O(N)? Yes, but mostly no. It's true that, in the worst case, all values hash to the same bucket, and you end up with O(N) performance. However, with a hashing algorithm that provides good distribution and an properly sized hashtable, hashtable performance remains O(1).

*Why?*, you ask. It comes down to the fill factor. The fill factor is the number of items in the hashtable divided by the number of buckets. If we store 1 000 000 items in a hashtable with 1 000 buckets, our fill factor is 1 000. For a hashtable, this is high. However, if we can keep our fill factor closer to 5 (as an example), performance will probably end up being somewhere between O(1) and O(log n). To reduce the load factor, we must increase the number of buckets - which is something we'll talk about in the next post. For now, know that it isn't uncommon for load factors to be less than 1!

Finally, the above implementation is the simplest possible solution. It really does work well when the chain isn't more than 5 or 10 items. However, there's all types of strategies we can use to improve this. For example, we could chain using a tree, or we could promote frequently used items at the start of the list. There are also completely different ways to deal with collisions that we won't cover.

### Conclusion

I initially said that I like to think of hashtables as flexible arrays. This is a good way to look at it from a usage point of view. From an implementation point of view, it's rather naive since it doesn't consider collisions. At an implementation level, I think of them as a sharding layer that sits on top of another data structure. A hashtable turns a single array, linked list or tree of X items, into Y arrays, linked lists or trees of X / Y items. You can either have 1 list of 25 items, or 5 lists of 5 items:

```
[a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y]

```

We end up with:

```
0 => [a, b, c, d, e]
1 => [f, g, h, i, j]
2 => [k, l, m, n, o]
3 => [p, q, r, s, t]
4 => [u, v, w, x, y]

```

The benefit, beyond being able to have non-integer keys, is speed. In the above, you don't have to search through 25 items. Instead, you have to search through 5 items (plus the initially bucket lookup).



### Improving hash algorithm (redis way)

What we saw was that collisions are a big part of what a hashtable must deal with. Improperly handled, collisions can have a significant negative impact on performance. There are two keys to managing this. First, our hashing algorithm must offer good distribution. Second, our fill factor, which assumes even distribution, must be kept low.

Recapping the previous post, the fill factor is the number of items in our hashtable divided by the number of buckets. The fill factor is really the number of elements we should expect to be chained within a given bucket. If we hold 1 000 000 items within 100 000 buckets, each bucket should have roughly 10 items. Chaining with a simple linked list would require 11 operations (1 to find the bucket, 10 to linearly scan the list).

While critical, fill factors aren't usually exposed to consumers. That's because most implementation automatically adjust the number of buckets they have as items are added (I have no idea if most implementation also shrink when items are removed). This is commonly known as rehashing.

The simplest implementation of rehashing is to add more buckets and remap all keys:

```
func (h *Hash) Set(key []byte, value interface{}) {
  ...
  h.count++
  if h.count / len(h.buckets) > 5 {
    h.rehash()
  }
}

func (h *Hash) rehash() {
  h.size *= 2 //getIndex will use this new value
  newBuckets := make([]*list.List, h.size)
  for i := 0; i < int(h.size); i++ {
    newBuckets[i] = list.New()
  }

  for i := 0; i < len(h.buckets); i++ {
    for element := h.buckets[i].Front(); element != nil; element = element.Next() {
      wrapped := element.Value.(*HashValue)
      newIndex := h.getIndex(wrapped.key)
      newBuckets[newIndex].PushBack(wrapped)
    }
  }
  h.buckets = newBuckets
}

```

This is similar to how buffer-backed arrays work. We double the size and copy values from the old to the new.

In reality, that's all you need to know about reashing and how hashtables keep a reasonable fill factor. However, I'd like to dive a little bit deeper into Redis rehashing implementation. Hashtables play a critical role in Redis. Not only are Sets, Hashes and Strings implemented using hashtables, but it's also the base object which holds all keys. Most of the relevant code sits in [src/dict.c](https://github.com/antirez/redis/blob/unstable/src/dict.c).

Whenever a value is added, `_dictExpandIfNeeded` is called to begin a rehashing process if necessary. Redis rehashes when the fill factor reaches 1 (or if rehashing is disabled, Redis will still force a rehash when the fill facto reach 5). However, unlike our simplistic approach above, Redis incrementally does a rehash. How? First it starts the same way by doubling the number of buckets. Instead of moving all elements over, it simply marks the hashtable as being in a "rehashing mode". As long as the the hashtable is in "rehasing mode" two sets of buckets exist, the old and the twice-larger new one.

What does the rehashing mode do? If you search the code, you'll see a handful of lines that look like:

```
if (dictIsRehashing(d)) _dictRehashStep(d);

```

`_dictRehashStep` moves 1 bucket from the old to the new. This means that hashtables which see a lot of activity get quickly rehashed, while hashtables which see little activity don't take up as much precious cycles. Eventually, when all the buckets are moved over, the old set of buckets can be deleted and the hashtable is no longer in "rehashing mode".

Of course, having two sets of buckets introduces a new level of complexity. Want to delete or get a value? You need to check both sets of buckets (which Redis calls tables). Take a look at the `get` implementation:

```
dictEntry *dictFind(dict *d, const void *key)
{
    dictEntry *he;
    unsigned int h, idx, table;

    if (d->ht[0].size == 0) return NULL; /* We don't have a table at all */
    if (dictIsRehashing(d)) _dictRehashStep(d);
    h = dictHashKey(d, key);
    for (table = 0; table <= 1; table++) {
        idx = h & d->ht[table].sizemask;
        he = d->ht[table].table[idx];
        while(he) {
            if (dictCompareKeys(d, key, he->key))
                return he;
            he = he->next;
        }
        if (!dictIsRehashing(d)) return NULL;
    }
    return NULL;
}

```

This might seem like a lot, but let's break it down. The key parts are:

1. Do a rehashing step if we're in "rehashing mode"  
2. Get the hashcode from the provided key (Redis uses MurmurHash2)  
3. Loop through our bucket sets (most of the time there'll only be 1, but when we're rehashing, there'll be 2)  
4. Get the bucket which holds the key and iterate through the chain finding a match  
5. If we've found our match, we return it  
6. Don't check the 2nd bucket set if we aren't rehashing (it won't exist)

The only other relevant piece of information is that Redis will use idle time to reshash the main key's dictionary. You can see this in the `dictRehashMilliseconds` function which is called from `incrementallyRehash` in `redis.c`.

The magic of keeping hashtable efficient, despite varying size, is not that different than the magic behind dynamic arrays: double in size and copy. Redis' incremental approach ensures that rehashing large hasthables doesn't cause any performance hiccups. The downside is internal complexity (almost every hashtable operation needs to be rehashing-aware) and a longer rehashing process (which takes up extra memory while running).