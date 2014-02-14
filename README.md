AllocatorBuilder
================

A highly composable, policy based C++ allocator based on ideas from Andrei Alexandrescu, presented at the "C++ and Beyond 2013" seminar.

The background behind the idea is to compensate the main problem of malloc and the other standard allocators, separation of memory pointer and the allocated size. This makes it very difficult for all kind of allocators to handle in a fast way memory allocations and deallocations. 
All users of manually allocated memory have to store the size anyway to ensure that no access beyond the length of the allocated buffer takes place.

So the appropached is every allocator returns such a Block

    struct Block {
      void* ptr;
      size_t length;
    };


And request goes this way:
    auto mem = allocator.allocate(42);


Motivation
----------
Raw memory is temporary needed inside a method. Most of the time the amount memory would fit on the stack and so :alloca() is ones friend. But in seldom cases more is needed and so :malloc() must be used. (Allocation on the stack is much faster because it a wait-free operation and in many cases the allocated memory is much more cache friendly.)

So the code could look like this
  
    const int STACK_THRESHOLD = 1024;
    int neededBytes = 42
    bool wouldFitOnTheStack = neededBytes < STACK_THRESHOLD;
    char* p = wouldFitOnStack? : (char*):alloca(neededBytes) : nullptr;
    
    std::unique_ptr<char[]> heapAllocated(!wouldFitOnStack? new char[neededBytes] : nullptr);
  
    if (!p) {
      p = heapAllocated.get();
    }
  
    // ... work with p[0..neededBytes-1]

Everybody would agree that this is not nice! So what if one could encapsulate this into something like:

    const int STACK_THRESHOLD = 1024;
    int neededBytes = 42

    typedef FallbackAllocator<StackAllocator<STACK_THRESHOLD>, Mallocator<>> LocalAllocator; 
    LocalAllocator localAllocator;
  
    auto block = localAllocator.allocate(neededBytes);
    SCOPED_EXIT { localAllocator.deallocate(block); };
    
    auto p = static_cast<char*>(block.ptr);
    
    // ... work with p[0..neededBytes-1]
 
So, isn't this nicer? 
  
  
Content
-------
  * AffixAllocator - Allows to automatically pre- and sufix allocated regions.
  * Bucketizer - Manages a bunch of Allocators with increasing bucket size
  * FallbackAllocator - Either the default Allocator can handle a request, otherwise it is passed to a fallback Allocator
  * Mallocator - Provides and interface to systems ::malloc()
  * Segregator - Separates allocation requests depending on a threshold to Allocator A or B
  * SharedFreeList - Manages a list of freed memory blocks in a list for faster reusage in a thread safe manner
  * SharedCascadingAllocator - Manages in a thread safe way Allocators and automatically creates a new one when the previous are out of memory
  * SharedHeap - A thread safe heap with minimal overhead and as far as possible in a lock-free way.
  * StackAllocator - Provides a memory access, taken from the stack
  

Author 
------
  Felix Petriconi
  

Contributions
-------------

Contributions are welcome! Please make a forke, request for integration.
  
License
-------
  Boost 1.0 License


Version
-------
  0.1.0

Requirements
------------
  * C++ 11 (partly, as far as Visual Studio 2012 supports them)
  * boost 1.55
  * GoogleTest (is now part of the repository, because its CMakeFiles.txt needed some patches.)


Platform
--------
  No platform specific features used, but tested so far only with MS Visual Studio 2012

ToDo
----
  * Add UnitTests
  ** FallbackAllocator
  ** Segregator
  ** Bucketizer
  * Add FreeList
  * Add CascadingAllocators
  * Add expand at the missing places
  ** SharedHeap


