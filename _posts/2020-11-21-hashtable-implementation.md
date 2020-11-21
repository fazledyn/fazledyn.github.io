---
layout: post
title:  "HashTable Implementation & Few Findings"
categories: [ programming ]
image: assets/images/8.jpg
tags: [ sticky, hashing, programming ]
lang: en
---

### What is a hash table ?

Hash table (also, hash map) is a data structure that basically maps keys to values. A hash table uses a hash function to compute an index into an array of buckets or slots, from which the corresponding value can be found. A hash table supports three basic operations- Search, Delete & Insert. All of them require having a complexity of O(1).

In our assignment, we were told to store a key-value pair in the hash table. The key will be string and the value, which is an integer will be associated with it. There should be three basic operation on the hash table- Search, Delete & Insert. Except insert, search and delete will be based on the key.

In this post, I will not dive into the basics and will assume that the reader has already grasped the beginner level knowledge about hash table/hash map. In the following, I will just explain the three different methods of hashing I used and hash functions that I had implemented.

I implemented four hash functions (Polynomial Roll Hash Function, Power Hash Function, Jenkins Hash Function, FNV Hash Function) that each take an input string s and returns a numerical value (basically a large integer). Then, the numerical value is modulated through another value (size of the hash table) to accomodate it within the range. I call it `hash_index`.


#### 1. Polynomial Roll Hash Function
We take a string **s** and then multiply it's i-th character (ASCII Value) with the i-th power of a constant (PRC). Then we sum up the whole value and return it. Since the value grow exponentially, the overall final value can be very very large. So large that it can't be accomodated in an integer data type. Thus, we have to store it in a `long long int` which is defined here as `u64`.

{% highlight c++ %}

u64 Hash1(string s) {
    u64 hash_value = 0;
    int PRC = 61;

    for (int i=0; i < s.size(); i++) {
        hash_value += int(s[i]) * pow(PRC, i);
    }
    return hash_value;
}

{% endhighlight %}

#### 2. Power Hash Function
This is similar to the previous one, the only difference here is that, we count each step as i-th power of the i-th character's ASCII value. Then the value gets summed up and returned.

{% highlight c++ %}

u64 Hash2(string s) {
    u64 hash_value = 0;
    for (int i=0; i < s.size(); i++) {
        hash_value += pow(int(s[i]), i);
    }
    return hash_value;
}

{% endhighlight %}

#### 3. Jenkins Hash Function
This one's taken from the internet, while browsing through some research papers of hashing. I wish I had found it earlier. I would have approached in a different way.

{% highlight c++ %}

u64 Hash3(string s) {
    u64 hash_value = 0;

    for (int i=0; i < s.size(); i++) {
        hash_value += s[i];
        hash_value += (hash_value << 10);
        hash_value ^= (hash_value >> 6);
    }
    hash_value += (hash_value << 3);
    hash_value ^= (hash_value >> 11);
    hash_value += (hash_value << 15);

    return hash_value;
}

{% endhighlight %}

#### 4. FNV Hash Function
FNV Hash Function (abbr. Fowler-Nol-Vo Hash Function) is another hash function named after these three computer scientists. This hash function involves using a prime number, referred as `FNV_PRIME`. Calculationg the value of this prime number is quite interesting. Read [more](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function).

{% highlight c++ %}

u64 Hash4(string s) {
    u64 hash_value = 0;
    const int FNV_PRIME = 16777619;

    for (int i=0; i < s.size(); i++) {
        hash_value = hash_value * FNV_PRIME;
        hash_value ^= s[i];
    }
    return hash_value;
}

{% endhighlight %}

> A few things to note here. Power function is quite costly in terms of computation. A more optimized approach would be to calculate it manually over each iteration. A custom function could be written instead of using the library function.

### The Hashing Methods (Double Hashing, Custom Probing)
Both of these methods require the use of another **Auxilary Hash Function**. This function is defined as a simple function but in reality, it's not. It has some guidelines attached to it which must be followed to make sure each elements gets its place. As mentioned in Geeks4Geeks, these two conditions must be fulfilled in order to make an auxilary function work. A good hash function is:
- It must never evaluate to zero.
- Must make sure that all cells can be probed/traversed.

{% highlight c++ %}

u64 auxHashFunc(string s) {
    u64 hash_value = 0;
    for (int i=0; i < s.size(); i++) {
        hash_value += int(s[i]);
    }
    hash_value = PREVIOUS_PRIME - (hash_value % PREVIOUS_PRIME);
    return hash_value;
}

{% endhighlight %}

> Note: The PREVIOUS_PRIME is a constant prime number which must have a value less than the hash table size. Usually it gives better performance if the previous prime of the table size is used.

***
#### Double Hashing Method
This method involves two hash function used as:
`doubleHash(s,i) = (hashFunc(s) + i*auxHash(s)) % TABLE_SIZE`

Any of the basic hash functions can be used as the first hash function. Note that the value of i increases on each iterations until the element finds an empty space in the hash table. In my opinion, it's just a superset of [linear probing method](https://en.wikipedia.org/wiki/Linear_probing). The only difference is that the offset value involves the use of another hash function.

#### Custom Probing Method
This method is of the following form:
`customProbe(s,i) = (hashFunc(s) + C1*i*auxHash(s) + C2*i*i) % TABLE_SIZE`

Only difference here is the constants C1, C2 and the additional part of i^2. The offset value found through this method is usually more than the double hashing method. The location for the hash_index are more sparse, more uniformly distributed throughout the hash table. From observation, setting C1, C2's value as single digit prime makes it more efficient as well as fast. This however is also a superset of [quadratic probing method](https://www.geeksforgeeks.org/quadratic-probing-in-hashing/).

### HashTable & Some Universal Rules

In a HashTable, the sizes aren't predefined. The tables are constructed in align to **modular arithmetic rules** such as, having a prime number as the hash table size. In our assignment, the hash table size was taken as user input. After a few trial and error, it became evident that there is no way to actually accomodate all key-value pairs in the hash table, within a reasonable time & without data loss.

As a solution, the hash table had to be constructed following some certain guidelines. One of them was to create the hash table array with a size of the nearest prime number. A one-time sacrifice had to be made for the overall performance of the hash table operations. The **PREVIOUS_PRIME** can also be evaluated during the same operation.

##### Prime number as the hash table size?

It is quite evident that the additional hash functions (auxHash, customProbeHash) can return value of all kind: even-odd, prime-non prime, multiple of 10, 5 etc. These additional values are considered to be the offset for next cell in the hash table. Having common factors between **additional value** and **table size** can and will have effect on the operations of a hash table. While jumping from one index to another, there can be some indexes that can never be probed/traversed.

On the other hand, if we have a hash table size of a prime number, all cells can be traversed eventually. Which is the most important point while constructing a hash table with probing operation.

**You can find my HashTable implementation in C++ [here](https://github.com/fazledyn/L2T2_Offlines/tree/master/CSE%20208/Offline8%20-%20Hashtable)**

**References:** [Aozturk's Medium Blog](https://aozturk.medium.com/simple-hash-map-hash-table-implementation-in-c-931965904250), [GeeksForGeeks](https://www.geeksforgeeks.org/double-hashing/), [Damn Fast Hash Table](http://www.idryman.org/blog/2017/05/03/writing-a-damn-fast-hash-table-with-tiny-memory-footprints/)