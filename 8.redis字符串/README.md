![](https://img.shields.io/badge/-ceph-green)
# 疑问
## `signalModifiedKey(c,c->db,c->argv[1]);` 向数据库发送键被修改的信号
```
当某个客户端发来的命令修改了某个key后，都会调用到signalModifiedKey函数。
该函数仅仅是调用touchWatchedKey而已。touchWatchedKey函数的作用，就是当数据库db中的key被改变时，
增加REDIS_DIRTY_CAS标记到所有WATCH该key的客户端标志位中，这样这些客户端执行EXEC命令时，
就会直接回复客户端错误信息。
```
***
## `notifyKeyspaceEvent(REDIS_NOTIFY_STRING,"set",c->argv[j],c->db->id);` 发送事件通知
* 键空间通知：“某个键执行了什么命令”的通知称为键空间通知（key-space notification） 比如监视某个数据库某个键的各种更改通知
* 键事件通知：键事件通知（key-event notification）关注的是“某个命令被什么键执行了 比如客户端获取0号数据库中所有执行DEL命令的键
***
## `server.dirty++;` 将服务器设为脏
```
struct redisServer {
//...
    long long dirty;            /* changes to DB from the last save */
    long long dirty_before_bgsave; /* used to restore dirty on failed BGSAVE */
//...
};
如注释所言，redisServer中的dirty用来存储上次保存前所有数据变动的长度。

dirty将在如下情况被累加：
update
flush
delete
rename command
move command
expire command
persist command
exec command
script command
sort command
set系列命令： hset hsetnx hmset hincrby hdel
list系列命令：push pushx lset pop
command

dirty将在如下情况被削减：
rdbsave后减去rdb所存储的数据的长度

dirty将在如下情况被重新设置为0：
在initServer中被初始化为零
执行debug command
执行rdb存储

diry在aof中的一个应用：
在call函数中：
void call(redisClient *c) {
   //...
    dirty = server->dirty;
    c->cmd->proc(c);
    dirty = server->dirty-dirty;
//...
    if (server->appendonly && dirty > 0) { 
//...
         len = feedAppendOnlyFile(c->cmd,c->db->id,c->argv,c->argc);
//...
               
        }   
}
```
## `rewriteClientCommandArgument(c,0,shared.set);`








