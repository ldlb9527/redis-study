![](https://img.shields.io/badge/-ceph-green)
# 创建sds涉及到的特殊函数
## s_trymalloc_usable 和 s_malloc_usable
***
```
sh = trymalloc?
        s_trymalloc_usable(hdrlen+initlen+1, &usable) :
        s_malloc_usable(hdrlen+initlen+1, &usable);
```
* `s_trymalloc_usable`为`ztrymalloc_usable`的别名,尝试动态内存分配，如果分配失败就返回null，参数usable表示可用内存大小。采用 malloc 实现的。
```
void *ztrymalloc_usable(size_t size, size_t *usable) {
    /* 断言size大小是否超过最大值 */
    ASSERT_NO_SIZE_OVERFLOW(size);
    /* MALLOC_MIN_SIZE的定义 #define MALLOC_MIN_SIZE(x) ((x) > 0 ? (x) : sizeof(long)) */
    void *ptr = malloc(MALLOC_MIN_SIZE(size)+PREFIX_SIZE);

    if (!ptr) return NULL;
#ifdef HAVE_MALLOC_SIZE /* 系统存在 malloc_size 函数的情况 */
    size = zmalloc_size(ptr);
    update_zmalloc_stat_alloc(size);
    if (usable) *usable = size;
    return ptr;
#else
    *((size_t*)ptr) = size;
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    if (usable) *usable = size;
    return (char*)ptr+PREFIX_SIZE;
#endif
}
```
* `s_malloc_usable`为`zmalloc_usable`的别名，zmalloc 的变形，多了 usable 参数，记录可用内存大小。
***
## memcpy 和 memset
* `memcpy` 从源src所指的内存地址的起始位置拷贝n个支付到目标dst所指的内存地址的起始位置。
```
/* 返回dst的指针 */
void* __cdecl memcpy(
    _Out_writes_bytes_all_(_Size) void* _Dst, /* 用于存储复制内容的目标数组，类型强制转换为 void* 指针 */
    _In_reads_bytes_(_Size)       void const* _Src, /* 要复制的数据源，类型强制转换为 void* 指针 */
    _In_                          size_t      _Size /* 要复制的直接长度 */
    );
```
* `memset` 将_Dst中的前_Size个字符用_Val替代。
```
void* __cdecl memset(
    _Out_writes_bytes_all_(_Size) void*  _Dst,
    _In_                          int    _Val,
    _In_                          size_t _Size
    );
```


