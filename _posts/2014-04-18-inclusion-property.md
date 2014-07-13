---
layout: post
title: Implementing Inclusion Property with SimpleScalar  
categories: 
- Computer Architecture
- C/C++ 
tags:
- Architecture
- Cache
- C/C++
status: publish
type: post
published: true
author: Bo Yang
---

### 1. Inclusion Property

If the two-level cache system satisfies the inclusion property, it means everything in the primary cache is also in the secondary cache and all snooping can be done by the secondary cache. To implement the inclusion property, two requirements must be met:

- Whenever a new block of data are loaded into(or removed from) L1 cache, they also should be loaded into(or removed from) the L2 cache.
- Whenever a block of data are loaded into(or removed from) L2 cache, they also should be loaded into(or removed from) the L1 cache.

_SimpleScalar_ already implemented the first requirement. When there is a cache miss in L1 cache, L2 cache will be accessed for the block. If the block are also cannot be found in the L2 cache, they will be loaded from the memory. The data consistency is kept in this way.

However, the second requirement is not supported by the official SimpleScalar. To implement this feature, the data block must be checked every time L2 cache encounter a cache miss. Since all cache read/write operations are done in function cache_access(), the action of checking and removing data blocks from L1 cache should also be added into that function with the condition of accessing L2 cache.

### 2. Code Change 

Since there is no existing way in cache_t to find the upper level cache(DL1) of the DL2 cache, a new structure element 

`struct cache_t *ucp;		/* upper level cache */`

should be added into struct cache_t. This pointer should be set to NULL in function cache_create() if the cache name is not “dl2”. Otherwise the pointer to DL1 cache should be assigned.

To support the new added struct element, a new parameter 

`struct cache_t *ucp;		/* upper level cache */`

should also be added into function cache_create(), which is called in sim-cache.c and sim-outorder.c. The new interface of this function becomes:

	/* create and initialize a general cache structure */
	struct cache_t *			/* pointer to cache created */
	cache_create(char *name,		/* name of the cache */
		     int nsets,			/* total number of sets in cache */
		     int bsize,			/* block (line) size of cache */
		     int balloc,		/* allocate data space for blocks? */
		     int usize,			/* size of user data to alloc w/blks */
		     int assoc,			/* associativity of cache */
		     enum cache_policy policy,	/* replacement policy w/in sets */
		     /* block access function, see description w/in struct cache def */
		     unsigned int (*blk_access_fn)(enum mem_cmd cmd,
						   md_addr_t baddr, int bsize,
						   struct cache_blk_t *blk,
						   tick_t now),
		     unsigned int hit_latency,/* latency in cycles for a hit */
		     struct cache_t *ucp);	/* upper level cache */

For DL2 cache, the pointer ucp is set to cache_dl1 in sim-cache.c in function cache_create(). For other cache, NULL is set. For example:

	cache_dl2 = cache_create(name, nsets, bsize, /* balloc */FALSE,
					   /* usize */0, assoc, cache_char2policy(c),
					   dl2_access_fn, /* hit latency */1,cache_dl1);

As checking and removing blocks from DL1 only involves DL1 and DL2 cache, all the operations can be encapsulated in a function cache_inclusion(). This function is called after blocks moved out from DL2 in the case of cache miss. Then check every block of DL1 in DL2: if the block can be found in DL2, keep it; it the block cannot be found in DL2, it means this block should be moved out of cache, so flush this block in DL1 to guarantee the inclusion property.

	/*
	 * Inclusion Property:
	 * 	Make sure data leave DL2 also leave DL1 
	 * */
	int cache_inclusion(struct cache_t *cp_l1, /* L1 cache */
			struct cache_t *cp_l2, /* L2 cache */
			tick_t now)		/* time of access */
	{
	
		/* Loop through each set and each block in the L1 cache, 
		 * probe the L2 cache to guaranttee the inclusion */
		int num_l1_sets = cp_l1->set_mask + 1;
		int set;
	  	struct cache_blk_t *blk;
		md_addr_t addr;
		int hindex;
		int num_hash_buckets = cp_l1->hsize;
		int there; /* indicates if the L1 line is in L2 */
		int lat=0;
	
		/* for each set */
		for(set=0; set<num_l1_sets; set++){
			/* for each block in a higly-associativity cache*/
	  		if (num_hash_buckets){
				/* for each hash bucket */
				for(hindex=0; hindex<num_hash_buckets; hindex++){
				    /* for each block in a hash bucket */
				    for (blk=cp_l1->sets[set].hash[hindex];
					 blk;
					 blk=blk->hash_next)
					{	
						/* Probe the L2 cache for this block */
						addr = CACHE_MK_BADDR(cp_l1, blk->tag, set);
						there = cache_probe(cp_l2, addr);
						if(!there){ // line in DL1 but not in DL2
							lat += cache_flush_addr(cp_l1,addr,now);
							//fprintf(stderr, "\nEvict address %x from DL1!\n", addr);
						}
			   		}/* end for block in a bucket */
				}/* end for each bucket */
	  		}/* end if highly associative cache*/
	
			/* for each block in a low-associativity cache*/
			else {
				for (blk=cp_l1->sets[set].way_head;
				 blk;
				 blk=blk->way_next)
	    		{
					/* Probe the L2 cache for this block */
					addr = CACHE_MK_BADDR(cp_l1, blk->tag, set);
					there = cache_probe(cp_l2, addr);
					if(!there){ // line in DL1 but not in DL2
						lat += cache_flush_addr(cp_l1,addr,now);
						//fprintf(stderr, "\nEvict address %x from DL1!\n", addr);
					}
			    }/* end for */
	  		}/* end for each block */
		}/* end for each set */
		
		return lat;
	}

This function should be called in cache_access() after removing blocks from DL2. The basic idea of the above function is to  check every block of DL1 in DL2: if the block can be found in DL2, keep it; it the block cannot be found in DL2, it means this block should be moved out of cache, so flush this block in DL1 to guarantee the inclusion property.

The code segment for calling cache_inclusion() in cache_access() is:
	/* cache block not found */
	
	 /* **MISS** */
	 cp->misses++;
	…...
	
	/* link this entry back into the hash table */
	  if (cp->hsize)
	    link_htab_ent(cp, &cp->sets[set], repl);
	
	  /* Inclusion property:
	   * 	Make sure data leave DL2 also leave DL1, or vice versa */
	  if(!mystricmp(cp->name,"dl2") && cp->ucp != NULL) 
	  {
	  	lat += cache_inclusion(cp->ucp,cp,now);
	  }
	
	  /* return latency of the operation */
	  return lat;

### 3. Experiments

In my experiment, three benchmarks are used: gcc, mcf and vpr. For each benchmark, three different configurations are tested, which are the same as the three commands listed in the project requirement. Compared to the SimpleScalar without inclusion property, it took 2 times longer time after implementing the inclusion property. 

After implementing inclusion property, the miss rates of DL1 generally increase a little, while the miss rates of DL2 decrease or keep the same. This is consistent with the code change, because due to inclusion property, statistically fewer data can be stored in DL1, but more valid data can be stored in DL2 cache. 

![Miss rate of DL1]({{ site.url }}/assets/images/2014-04-18-inclusion-property/dl1.jpg)

Figure 1. Miss rate of DL1, with(red) and without(blue) inclusion.

![Miss rate of DL2]({{ site.url }}/assets/images/2014-04-18-inclusion-property/dl2.jpg)

Figure 2. Miss rate of DL2, with(red) and without(blue) inclusion.

### 4. Conclusion

In this project I implemented the inclusion property with SimpleScalar. I tested my change with three different benchmarks in three different configurations. Experiments show proves the correctness of my change. Inclusion property cannot guarantee the overall performance the cache, although it seems like a promising technique.
