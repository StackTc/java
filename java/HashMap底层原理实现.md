## HashMap底层原理实现
### HashMap数据结构
* ```K key```
* ```V value```
* ```Entry next```
* ```int hash```
*

HashMap内部储存数据的是一个由```Entry```构成的数组，数组中的元素如上图所示，key存放键，value存放值，hash存放标志位，next是存放当不同key所生成的hash值相同时（不同的值可以存在相同的hash值），用来存放第二个键值对。

![HashMap](https://github.com/StackTc/java/blob/master/photos/1517245358258.jpg "HashMap")





### HashMap方法实现
结下来介绍一下HashMap主要的2个方法的底层实现原理。

```get（key）```，```put(key,vaule)```。

### put(key,value)

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}



final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```
从上面可以看到完成一个put过程需要```key```跟```value```，但是key会根据类中的```hash```方法生成对应的```hash值```,根据```hash值```去数组中查找是否存在该hash值所对应的```Entry```,如果存在，那么替换```Entry```,然后替换所对应的值，

### get(key)

```
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) 
        == null ? null : e.value;
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

