##help @string
<pre>

set  mset
get  mget
APPEND key1 test1  追加
GETRANGE key1 0 4   test1
SETRANGE key1 5 hello  test1hellp 从第5个字符后面用hello覆盖
STRLEN  key1  返回5  字符的个数

MSET  
MSETNX
同时为多个键设置值。
如果某个给定键已经存在， 那么 MSET 将使用新值去覆盖旧值， 如果这不是你所希望的效果， 请考虑使用 MSETNX 命令， 这个命令只会在所有给定键都不存在的情况下进行设置。
MSET 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况。

</pre>

[对象的类型与编码](http://redisbook.com/preview/object/object.html)  
整数		    "int"  
embstr 编码的简单动态字符串（SDS）		"embstr"  
简单动态字符串		"raw"  
字典		    "hashtable"  
双端链表		"linkedlist"  
压缩列表		"ziplist"  
整数集合		"intset"  
跳跃表和字典	"skiplist"  

<pre>
INCR k1  自增
INCRBY  k1  100
INCRBYFLOAT  k1 0.5

DECR k1  自减
DECRBY   k1  100

getset k1  test  用新值覆盖老值,同时把老值显示出来,这样可以节约成本(网络流量的请求)
</pre>
