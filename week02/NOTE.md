# HashMap



### 方法分析

#### put

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
   int h;
   return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> 首先计算key的hash值，若key ==null，其hash值等于0。取key的hashCode值并和自身的高16位做异或计算

```java
/**参数解释
hash ： key的hash值
key ：键
value ：键对应的值
onlyIfAbsent: 若为true，则不改变已有值
evict: 若为false，hash表正在创建的模式下
*/
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
1        Node<K,V>[] tab; Node<K,V> p; int n, i;
2        if ((tab = table) == null || (n = tab.length) == 0)
3            n = (tab = resize()).length;
4        if ((p = tab[i = (n - 1) & hash]) == null)
5           tab[i] = newNode(hash, key, value, null);
6        else {
7            Node<K,V> e; K k;
8            if (p.hash == hash &&
9                ((k = p.key) == key || (key != null && key.equals(k))))
10                e = p;
11           else if (p instanceof TreeNode)
12               e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
13            else {
14                for (int binCount = 0; ; ++binCount) {
15                    if ((e = p.next) == null) {
16                        p.next = newNode(hash, key, value, null);
17                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
18                            treeifyBin(tab, hash);
19                        break;
20                    }
21                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
22                        break;
23                    p = e;
                }
            }
24            if (e != null) { // existing mapping for key
25                V oldValue = e.value;
26                if (!onlyIfAbsent || oldValue == null)
27                    e.value = value;
28                afterNodeAccess(e);
                return oldValue;
            }
        }
29        ++modCount;
30        if (++size > threshold)
31            resize();
32        afterNodeInsertion(evict);
33        return null;
    }
```

> 第2行判断hash表(Node数组)目前没有创建，调用resize()方法创建hash表，在resize()方法中长度为16的Node数组，阈值12。
>
> 第4行中 通过 (n-1) & hash 取模(类似于 hash % n )，判断节点数据是否存在
>
>  	1. 若节点不存在调用newNode()创建一个新的节点
>  	2. 若节点p存在，说明hash冲突了，接下来解决hash冲突
>       	1. p节点的hash值等于我们要put的key的hash值，并且键相同，说明key在hash表中存在(即p节点)，我们是做的一个替换的操作
>       	2. p节点的key不是我们当前put的key，则需进行遍历操作查找下key，此时p节点是一个TreeNode(说明底层是红黑树，不是链表),调用putTreeVal 向红黑树中插入key-value
>       	3. p节点是链表中的头节点，遍历变量查找key
>            	1. 查找到，根据条件是否覆盖节点的value
>            	2. 未查找到，新增节点，新增后判断链表长度，是否需要转化为红黑树。
>  	3. hash冲突解决后，添加了新的节点或者替换已有节点的值，判断key-vaue的数量是否超过了阀值，超过则需进行扩容。

#### get

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
}
```

> get方法主要是根据key的hash值查找对应的节点(链表节点或者红黑树节点)
>
> 1. 通过hash去摸，计算出在Node数组中的索引，比较该索引处的节点的key，若相同则查找结束；若未查找到，则需遍历节点的后续节点
>    1. 后续节点是链表
>    2. 后续节点是TreeNode(红黑树)，遍历红黑树。
> 2. 若查找到节点返回其value，若未查找到则返回null