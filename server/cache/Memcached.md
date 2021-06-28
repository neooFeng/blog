# memcached 

## Hashing/Distribution Types
1. client hash by key
2. constant hash
> servers sequence in client config is matter.


## Data Expiry
1. lazy evict when request arrive
2. three level LRU, evict when no enough mem
   If there is not enough space within a **suitable slab** to store the value, then an existing 
   least recently used (LRU) object is removed (evicted) from the cache to make room for the new item.
3. explicitly set expired time, not return expired items


## mem allocate
1. lazy allocate when save request arrive
2. allocate by page(1M)
   improves the usage of the memory and protects it from the fragmentation in case the data expires from the cache
3. slab has many pages, page has many chunks, chunks in a slab always with same size.
4. chunks size is depend on the size of data to save, incr by a factor(default 1.25)
   1. when save data, first look for suite slabs
      a. if found, found free chunks and save it;
      b. if not found, allocate one page with special thunk size; 
      c. if not enough physic mem, use LRU to evict some items.
   2. when update data, and the new data size is changed, relocate the suit slab use the above logic.
   

## threads
1. worker threads
2. use libevent, each thread is able to handle many clients.
3. more like nginx than tomcat
4. single thread vs. concurrent thread with lock


## ops
### stats
1. max_bytes, bytes 
2. max_conn, current_conn, 
3. cmd_get, get_misses
4. evictions, Number of valid items removed from cache to free memory for new items. 
5. evict_unfetched, evict_expired

## stats slabs
1. total_chunks, free_chunks
2. chunk_size

## stats items
1. evicted, The number of items evicted to make way for new entries
2. evicted_time, The time of the last evicted entry

## stats sizes
first column is the size of the item , and the second column is the count of the number of items of that size within the cache

