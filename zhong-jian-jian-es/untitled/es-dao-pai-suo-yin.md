# ES 倒排索引

## 1  es 倒排索引的简单描述

 es 中不仅仅建立了 ”term dictionary -- doc “倒排索引，而且还对词进行了索引（term index -- term dictionary）,这样方便快速进行找到词，然后再由词找到对应的文档。

并且底层是基于FST\(Finite State Transducer\) 数据结构，对词典中的单词前缀和后缀进行重复利用，空间占用小。进而减少了访问磁盘的次数，提高了性能。





## 参考文献：

[https://zhuanlan.zhihu.com/p/33671444](https://zhuanlan.zhihu.com/p/33671444)

