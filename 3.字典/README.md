![](https://img.shields.io/badge/-ceph-green)
# 字典
* 这一策略是必要的，因为在一次完整的迭代过程中，
* 哈希表的大小有可能在两次迭代之间发生改变。
***
* 哈希表的大小总是 2 的某个次方，并且哈希表使用链表来解决冲突，
* 因此一个给定元素在一个给定表的位置总可以通过 Hash(key) & SIZE-1
* 公式来计算得出，
* 其中 SIZE-1 是哈希表的最大索引值，
* 这个最大索引值就是哈希表的 mask （掩码）。
***
* 举个例子，如果当前哈希表的大小为 16 ，
* 那么它的掩码就是二进制值 1111 ，
* 这个哈希表的所有位置都可以使用哈希值的最后四个二进制位来记录。
* 
* WHAT HAPPENS IF THE TABLE CHANGES IN SIZE?
* 如果哈希表的大小改变了怎么办？
* 当对哈希表进行扩展时，元素可能会从一个槽移动到另一个槽，
* 举个例子，假设我们刚好迭代至 4 位游标 1100 ，
* 而哈希表的 mask 为 1111 （哈希表的大小为 16 ）。
* 如果这时哈希表将大小改为 64 ，那么哈希表的 mask 将变为 111111 
***
这个迭代器是完全无状态的，这是一个巨大的优势，
* 因为迭代可以在不使用任何额外内存的情况下进行。
***
* 这个设计的缺陷在于：
*
* 1) It is possible that we return duplicated elements. However this is usually
*    easy to deal with in the application level.
*    函数可能会返回重复的元素，不过这个问题可以很容易在应用层解决。
* 2) The iterator must return multiple elements per call, as it needs to always
*    return all the keys chained in a given bucket, and all the expansions, so
*    we are sure we don't miss keys moving.
*    为了不错过任何元素，
*    迭代器需要返回给定桶上的所有键，
*    以及因为扩展哈希表而产生出来的新表，
*    所以迭代器必须在一次迭代中返回多个元素。
* 3) The reverse cursor is somewhat hard to understand at first, but this
*    comment is supposed to help.
*    对游标进行翻转（reverse）的原因初看上去比较难以理解，
*    不过阅读这份注释应该会有所帮助。



