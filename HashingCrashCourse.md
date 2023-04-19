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


## Section 43.2: Consistent Hashing

Consistent hashing is a technique used in distributed systems for assigning keys to nodes in a way that minimizes reassignment of keys when nodes are added or removed from the system. It achieves this by mapping each key to a point on a ring, and then mapping each node to a point on the same ring. Keys are then assigned to the node whose point is closest in a clockwise direction on the ring.

Some key advantages of consistent hashing include:

* balancing: Since each node is responsible for a range of keys, the load is evenly distributed across nodes.
* Scalability: Nodes can be added or removed from the system with minimal key reassignment.
* Fault-tolerance: When a node fails, only the keys assigned to that node need to be reassigned, rather than the entire key space.

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
