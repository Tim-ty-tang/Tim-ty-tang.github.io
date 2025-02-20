---
title: "Make Things Small"
date: "2023-12-01"
summary: "A tidbit on Python and Memory Footprints."
description: "A tidbit on Python and Memory Footprints."
toc: false
readTime: true
autonumber: true
math: true
tags: ["performance", "python"]
showTags: true
hideBackToTop: false
---

## Making Python Objects [Small](https://habr.com/en/post/458518/)

Very often python consumes a stupid amount of memory, and it make suck processing scripts or mass processing annoying. TO summarize the tips from the blogpost:

1. Data classes are smaller than dicts. Quote:  
    `Starting in Python 3.3, the shared space is used to store keys in the dictionary for all instances of the class. This reduces the size of the instance trace in RAM.`  
    Which seems a bit magical, but I will take it.
2. __slots__ helps to remove __dict__ and __weakref__ from a class and it is noted that it's the main memory saving technique for pure python classes. Quote:  
    `This reduction is achieved by the fact that in the memory after the title of the object, object references are stored — the attribute values, and access to them is carried out using special descriptors that are in the class dictionar`y.
3. There are some additional classes we can use in the [RecordClass](https://bitbucket.org/intellimath/recordclass/src/master/) library, which involves tuples not part of the cyclic GC process. This can minimally reduce size of the pure python class object size.
4. Cython is not just for speed. It's for memory footprint too.
5. Numpy for arrays (as usual
