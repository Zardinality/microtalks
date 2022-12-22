# first part
* what I want to convey:
    * why do we have cache
        * because ram is slow and cheap, cache is quick and expensive
    * what do we want from cache
        * just like it is transparent to user
        * we want it to quickly store when it is not full
        * we want it to quickly load when it has the answer
        * **from function part, it is just like a hash map**
    * the cache structure
        * what is the hash map key and value
        * key is address[address looking, virtual or real?]
            * because that is all we have in machine code
        * value is cache line
            * what is cache line
            * why cache are made of cache lines? rather than cache items?
                * ram bus has 64 bytes
                * data locality
        * cache is a hash map
            * what is its container?
            * what is its equal function?
            * what is its hash function?
    * when it is SMP
        * distributed hash map
        * how do we achieve sync?
            * status
            * message
        * MESI
            * M E S I
            * messages
            * example
    * cache contention