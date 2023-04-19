# Chapter 43: Hash Functions
## Section 43.1: Hash functions in Python

Python has a built-in hash() function that returns a hash value for an object. The hash value is an integer that represents the object and can be used for various purposes such as indexing, comparison, and data deduplication. The hash() function is used internally by Python for implementing dictionaries and sets.

The hash value returned by the hash() function is not guaranteed to be unique for different objects. In fact, it's quite common for different objects to have the same hash value, which is known as a hash collision. To handle hash collisions, Python uses an algorithm called open addressing, which tries to find a different slot in the hash table to store the colliding object.

Python also provides a way to define custom hash functions for user-defined classes using the hash() method. The hash() method should return an integer that represents the object's hash value. If two objects compare as equal using the eq() method, they should have the same hash value.

Here's an example implementation of the hash() method for a simple Point class:

    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __hash__(self):
        return hash((self.x, self.y))
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
       
In this implementation, the hash value for a Point object is computed by hashing a tuple of its x and y coordinates using the built-in hash() function. The eq() method is also implemented to compare two Point objects for equality based on their x and y coordinates.

In addition to the built-in hash() function, Python provides several hash functions in the hashlib module for cryptographic purposes, such as SHA-256 and MD5. These hash functions are designed to be secure and resistant to attacks such as collision attacks and preimage attacks. They can be used for verifying the integrity of data and generating digital signatures. However, they are generally slower than the built-in hash() function and should only be used when security is a concern.


## Section 43.2: SHA-3

SHA-3 (Secure Hash Algorithm 3) is a cryptographic hash function that is designed to be secure and resistant to attacks such as collision attacks and preimage attacks. It was developed by the National Institute of Standards and Technology (NIST) and was selected as the winner of the SHA-3 competition in 2012.

SHA-3 is based on the Keccak algorithm, which is a family of hash functions that use a sponge construction. The sponge construction is a way of transforming an input message into an output hash value by absorbing the input message into a buffer and then squeezing the buffer to produce the hash value.

In Python, SHA-3 hash functions are available in the hashlib module. To use SHA-3, you can create a new SHA-3 object using the sha3_224(), sha3_256(), sha3_384(), or sha3_512() constructor, and then update the object with the input message using the update() method. Finally, you can obtain the hash value by calling the digest() or hexdigest() method.

Here's an example of using the sha3_256() hash function to compute the hash value of a message:

    message = b"Hello, world!"
    hash_object = hashlib.sha3_256(message)
    hex_dig = hash_object.hexdigest()
    print(hex_dig)


In this example, the message "Hello, world!" is first encoded as a bytes object and then passed to the sha3_256() constructor to create a new SHA-3 object. The object is then updated with the message using the update() method, although in this case, the message is small enough to be passed directly as an argument. Finally, the hexdigest() method is called to obtain the hash value as a hexadecimal string.


## Section 43.3: Consistent Hashing

Consistent hashing is a technique used in distributed systems for assigning keys to nodes in a way that minimizes reassignment of keys when nodes are added or removed from the system. It achieves this by mapping each key to a point on a ring, and then mapping each node to a point on the same ring. Keys are then assigned to the node whose point is closest in a clockwise direction on the ring.

Some key advantages of consistent hashing include:

* balancing: Since each node is responsible for a range of keys, the load is evenly distributed across nodes.
* Scalability: Nodes can be added or removed from the system with minimal key reassignment.
* Fault-tolerance: When a node fails, only the keys assigned to that node need to be reassigned, rather than the entire key space.

    ```
    class ConsistentHashRing:
    def __init__(self, nodes, replicas=3):
        self.replicas = replicas
        self.ring = dict()
        for node in nodes:
            self.add_node(node)

    def add_node(self, node):
        for i in range(self.replicas):
            key = self.hash('{}-{}'.format(node, i))
            self.ring[key] = node

    def remove_node(self, node):
        for i in range(self.replicas):
            key = self.hash('{}-{}'.format(node, i))
            del self.ring[key]

    def get_node(self, key):
        if not self.ring:
            return None
        hash_key = self.hash(key)
        for node in sorted(self.ring.keys()):
            if hash_key <= node:
                return self.ring[node]
        return self.ring[min(self.ring.keys())]

    def hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)


In this implementation, we define a ConsistentHashRing class that takes a list of node names and a number of replicas as input. The number of replicas determines how many points on the ring each node is assigned to, with the default value being 3.

**add_node** method adds a node to the ring by generating replicas keys for the node and mapping each key to the node's name in the ring dictionary.

**remove_node** method removes a node from the ring by generating the same number of keys for the node and deleting them from the ring dictionary.

**get_node** method returns the node that a given key is assigned to by computing the hash of the key, finding the first node on the ring whose key is greater than or equal to the hash, and returning that node.
 
 **Links**  
 
[Consistent Hashing - Wikipedia](https://en.wikipedia.org/wiki/Consistent_hashing) 

[Consistent Hashing and Random Trees: Distributed Caching Protocols for Relieving Hot Spots on the World Wide Web - ACM](https://dl.acm.org/doi/10.1145/258533.258660)



## Section 43.4: Merkle Trees

Merkle Trees are a data structure used in cryptography and computer science to efficiently verify the integrity and authenticity of data. They are named after their inventor, Ralph Merkle.

A Merkle tree is a binary tree where each leaf node represents a single data block, and each non-leaf node represents the hash of its child nodes. The root node of the tree represents the hash of the entire data set. The main advantage of using a Merkle tree is that it allows a large data set to be efficiently verified without having to compute the hash of the entire set.

Key takeaways about Merkle Trees are:

* Merkle trees are a binary tree structure used to efficiently verify the integrity and authenticity of data.
* Each leaf node represents a single data block, and each non-leaf node represents the hash of its child nodes.
* The root node of the tree represents the hash of the entire data set.
* Merkle Trees are used in various applications such as blockchain technology and digital signatures.


```
import hashlib

class MerkleTree:
    def __init__(self, data):
        self.data = data
        self.tree = self.build_tree()
    
    def build_tree(self):
        data_len = len(self.data)
        if data_len == 1:
            return [hashlib.sha256(self.data[0].encode()).hexdigest()]

        if data_len % 2 != 0:
            self.data.append(self.data[-1])

        new_data = [hashlib.sha256((self.data[i] + self.data[i + 1]).encode()).hexdigest()
                    for i in range(0, len(self.data), 2)]
        return self.build_tree(new_data)

    def root_hash(self):
        return self.tree[0]
  ```
        
This implementation creates a Merkle tree from a list of data blocks. It recursively builds the tree by hashing pairs of data blocks until it reaches the root node.

The root_hash() method returns the root hash of the tree, which can be used to verify the authenticity and integrity of the entire data set.

**Links** 
 
 [R. C. Merkle, "A Digital Signature Based on a Conventional Encryption Function," Advances in Cryptology—CRYPTO '87, Springer Berlin Heidelberg, 1988, pp. 369-378, doi: 10.1007/3-540-48184-2_32](https://people.eecs.berkeley.edu/~raluca/cs261-f15/readings/merkle.pdf) 
 
 
## Section 43.5: Hash Table

A hash table is a data structure that allows for efficient insertion, deletion, and lookup of key-value pairs. It is implemented using a hash function that maps keys to an index in an array, where the corresponding value is stored. The hash function should ideally be fast and generate unique indices for each key to minimize Chaining: In this algorithm, each slot in the hash table contains a linked list of elements that hash to that slot. When a new element hashes to a slot that already contains one or more elements, it is simply added to the end of the linked list. This algorithm is simple to implement and handles collisions well, but it can be slow when the linked lists get long.


**Separate Chaining:**
>In this algorithm, each bucket of the hash table is a linked list. When a collision occurs, the colliding item is added to the linked list corresponding to its hash value. This algorithm has the advantage of being simple and easy to implement, but can result in slower performance due to the need to traverse the linked lists.
```
class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.next = None
        
class HashTable:
    def __init__(self, capacity=16):
        self.capacity = capacity
        self.size = 0
        self.buckets = [None] * self.capacity
        
    def hash(self, key):
        hash_value = 0
        for char in str(key):
            hash_value += ord(char)
        return hash_value % self.capacity
    
    def put(self, key, value):
        index = self.hash(key)
        node = self.buckets[index]
        
        # check if the key already exists and update the value
        while node:
            if node.key == key:
                node.value = value
                return
            node = node.next
        
        # create a new node and add it to the list
        new_node = Node(key, value)
        new_node.next = self.buckets[index]
        self.buckets[index] = new_node
        self.size += 1
        
    def get(self, key):
        index = self.hash(key)
        node = self.buckets[index]
        
        # traverse the list to find the key
        while node:
            if node.key == key:
                return node.value
            node = node.next
        
        # key not found
        return None
    
    def remove(self, key):
        index = self.hash(key)
        node = self.buckets[index]
        prev = None
        
        # traverse the list to find the key and remove the node
        while node:
            if node.key == key:
                if prev:
                    prev.next = node.next
                else:
                    self.buckets[index] = node.next
                self.size -= 1
                return
            prev = node
            node = node.next
        
        # key not found
        return None

```

**Open addressing:** 
>In this algorithm, when a collision occurs, a sequence of other slots in the hash table is probed until an empty slot is found. There are several variants of this algorithm, including linear probing (where the sequence of slots is simply the next slot in the table), quadratic probing (where the sequence is a quadratic function of the slot number), and double hashing (where the sequence is determined by a second hash function).


**Cuckoo hashing:**
>In this algorithm, each element in the hash table is stored in one of two possible slots, determined by two different hash functions. When a collision occurs, the element currently occupying one of the slots is moved to its alternate slot, and the new element is placed in the vacated slot. This process continues until either a cycle is detected (indicating that the table is too full to insert the new element) or the new element is successfully inserted.

**Robin Hood hashing:** 
>In this algorithm, elements are stored in the hash table in order of increasing distance from their preferred slot (i.e., the slot they would hash to in the absence of collisions). When a new element collides with an existing element, their distances from their preferred slots are compared, and the one with the smaller distance is moved to the next available slot with a larger distance. This process continues until the new element is successfully inserted.collisions.

**Links**
 
 
 ["Cuckoo Filter: Practically Better Than Bloom" (2014) by Bin Fan, Dave Andersen, Michael Kaminsky, and Michael D. Mitzenmacher](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf) 
  
  ["Robin Hood Hashing" (2016) by Eric Bainville.](https://dspace.mit.edu/bitstream/handle/1721.1/130693/1251799942-MIT.pdf)

