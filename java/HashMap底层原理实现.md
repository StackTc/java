## HashMap底层原理实现
### HashMap数据结构

* ```DEFAULT_INITIAL_CAPACITY = 1 << 4 (默认容量)```
* ```MAXIMUM_CAPACITY = 1 << 30 (最大容量)```
* ```DEFAULT_LOAD_FACTOR = 0.75f (默认加载因子)```
* ```Entry<?,?>[] EMPTY_TABLE = {} (哈希表)```
* ```size```
* ```threshold(当数组为空时，拿该参数来初始化哈希数组)```
* ```loadFactor（map实际容量）```

### 散列表结构
散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。
* ```key```
* ```value```
* ```next```
* ```hash```


HashMap内部储存数据的是一个由```Entry（哈希表）```构成的数组，数组中的元素如上图所示，key存放键，value存放值，hash存放标志位，next是存放当不同key所生成的hash值相同时（不同的值可以存在相同的hash值），用来存放第二个键值对。

```
对不同的关键字可能得到同一散列地址，即k1≠k2，而f(k1)=f(k2)，这种现象称为碰撞
```

![HashMap](https://github.com/StackTc/java/blob/master/photos/1517245358258.jpg "HashMap")





### HashMap方法实现
结下来介绍一下HashMap主要的2个方法的底层实现原理。

```get（key）```，```put(key,vaule)```。

### put(key,value)

```
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }

```

从上面可以看到完成一个put过程需要```key```跟```value```，首先如果散列表数组为空，则会根据```threshold```的大小默认生成一个散列数组，如果```key```为空则抛出异常，key会根据类中的```hash(Object k)```方法生成对应的```hash值```,以```hash值```为下表去数组中查找是否存在该hash值所对应的```Entry```，如果该位置上为null，那么插入，如果存在并且他们的key相同，那么替换```Entry```,然后替换所对应的值，如果该位置他们的```hash值```相同但是key不相同（哈希碰撞），那么将值赋值给next（哈希表的下一个节点），

### get(key)

```
public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
}

final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
}

```
从上可以看出``get方法```首先是通过参数key来生成hash值作为数组的下表去寻找，如果找不到，则返回空，如果找到了那么再判断他们的key是否相等，如果相等返回对应值，如果不相等，说明产生了哈希碰撞，在该哈希表的下一个链表寻找。
