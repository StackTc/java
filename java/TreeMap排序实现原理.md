# TreeMap 实现排序
## HashMap简介
在这里先简单的介绍一下HashMap，通常我们使用Map都是以键值对来使用HashMap的，因为HashMap底层是用hash表存储数据的，查询起来大大的增加了效率，基本上时间复杂度是O(1)。然而HashMap放进去的时候是无序的，我们希望有一个有序的Map应该怎么做呢，这里就使用到了TreeMap。

## TreeMap 属性
* **comparator**（比较器）
* **Entry<key,value> root**(树的根节点)
* **size**(个数)
## TreeMap 数据结构
简单的介绍一下```TreeMap```的数据结构，总所周知```HashMap```数据存储是放在一个hash表的数组中的，然而TreeMap的数据存储是放在一个二叉树里面的，节点是```Entry<key,value>```这样的对象，利用二叉树的结构来实现排序，这样就可以实现按照一定顺序将数据从左到右排序。
##  Entry<K,V>数据结构
* **key（Object）**
* **value（Object）**
* **left（Entry<K,V>）**
* **right（Entry<K,V>）**
* **parent（Entry<K,V>）**
## TreeMap方法实现
### push（K key, V value）

```
public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
	    // TBD:
	    // 5045147: (coll) Adding null to an empty TreeSet should
	    // throw NullPointerException
	    //
	    // compare(key, key); // type check
            root = new Entry<K,V>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<K,V>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```
### push方法逻辑说明
1. 如果输的根节点没数据,直接插入数据。
2. 查询是否有比较器，如果有比较器就根据比较结果，将插入节点跟存在节点对比，移动节点找到插入位置，将值插入。
3. 如果没有比较器就用默认的比较器比较，插入同上。
4. 最后fixAfterInsertion(e); 调整二叉树使平衡。
 
---

# get(Object key)

```
public V get(Object key) {
        Entry<K,V> p = getEntry(key);
        return (p==null ? null : p.value);
}


final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
	Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }    
```

### get方法逻辑说明
1. **如果key为空，则抛出异常。
2. **根据比较器，将key跟当前值比较。
3. 如果相等，返回value。
4. 如果不想等，如果比较值，移动当前结点到下一节点，继续进行比较，知道返回值。
