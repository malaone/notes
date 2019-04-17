---
title: Java-容器
categories: 算法
date: 2017-03-27 18:32:21
tags: [Java, 容器]
---

#### HashTable

HashTable的方法如put和get都加了synchronize，线程安全，但是效率低。HashTable基于Dictionary类，而HashMap是基于AbstractMap，实现方式与HashMap类似，都是散列表。

------

#### HashMap

##### Java7中的HashMap

参考文章：

- [HashMap实现原理及源码分析](http://www.cnblogs.com/chengxiao/p/6059914.html)

Java7中HashMap的数据结构如下：

![collector-hashmap-java7-structure](E:\typoraNote\picture\collector\collector-hashmap-java7-structure.png)

构造器：HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值，initialCapacity默认为16，loadFactory默认为0.75

```java
public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal initial capacity: " +initialCapacity);
  //此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

	this.loadFactor = loadFactor;
	threshold = initialCapacity;
	init();//init方法在HashMap中没有实际实现，不过在其子类如 linkedHashMap中就会有对应实现
}
```

put方法：构造函数中没有对table分配空间，在put中分配空间

```java
public V put(K key, V value) {
  //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4
	if (table == EMPTY_TABLE) {
		inflateTable(threshold);
	}
   //如果key为null，存储位置为table[0]或table[0]的冲突链上
	if (key == null)
		return putForNullKey(value);
	int hash = hash(key);
	int i = indexFor(hash, table.length);
	for (Entry<K,V> e = table[i]; e != null; e = e.next) {//遍历链表是否是相同的key，有则覆盖value
		Object k;
		if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {//hash值相同且key值相同
			V oldValue = e.value;
			e.value = value;//覆盖value
			e.recordAccess(this);
			return oldValue;//返回老的value
		}
	}

	modCount++;//保证并发访问时，若HashMap内部结构发生变化，快速响应失败
	addEntry(hash, key, value, i);//没有相同的key，则直接添加
	return null;
}

private void inflateTable(int toSize) {
    int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
  //此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}

void addEntry(int hash, K key, V value, int bucketIndex) {
	if ((size >= threshold) && (null != table[bucketIndex])) {//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
		resize(2 * table.length);
		hash = (null != key) ? hash(key) : 0;
		bucketIndex = indexFor(hash, table.length);
	}

	createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
	Entry<K,V> e = table[bucketIndex];
	table[bucketIndex] = new Entry<>(hash, key, value, e);
	size++;
}
```

hash方法：

```java
final int hash(Object k) {
	int h = hashSeed;//默认为0
	if (0 != h && k instanceof String) {
		return sun.misc.Hashing.stringHash32((String) k);//待研究。。。
	}
//先计算key的hashcode，可以参考String中的hashcode
	h ^= k.hashCode();

	h ^= (h >>> 20) ^ (h >>> 12);
	return h ^ (h >>> 7) ^ (h >>> 4);
}

//String中的hashcode
public int hashCode() {
	int h = hash;
	if (h == 0 && value.length > 0) {
		char val[] = value;

		for (int i = 0; i < value.length; i++) {
			h = 31 * h + val[i];
		}
		hash = h;
	}
	return h;
}
```

获取下标方法indexFor：

```java
static int indexFor(int h, int length) {
  return h & (length-1);//保证index值不溢出
}
```

扩容相关方法：

```java
void resize(int newCapacity) {
	Entry[] oldTable = table;
	int oldCapacity = oldTable.length;
	if (oldCapacity == MAXIMUM_CAPACITY) {
		threshold = Integer.MAX_VALUE;
		return;
	}

	Entry[] newTable = new Entry[newCapacity];
	transfer(newTable, initHashSeedAsNeeded(newCapacity));
	table = newTable;
	threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

void transfer(Entry[] newTable, boolean rehash) {
	int newCapacity = newTable.length;
　　//for循环中的代码，逐个遍历链表，重新计算索引位置，将老数组数据复制到新数组中去（数组不存储实际数据，所以仅仅是拷贝引用而已）
	for (Entry<K,V> e : table) {
		while(null != e) {
			Entry<K,V> next = e.next;
			if (rehash) {//重新计算hash
				e.hash = null == e.key ? 0 : hash(e.key);
			}
			int i = indexFor(e.hash, newCapacity);
			e.next = newTable[i];//如果index相同，复制后的链表与原来的倒置
			newTable[i] = e;
			e = next;
		}
	}
}
```

hashSeed也是一个非常重要的角色，可以把它看成一个开关，如果开关打开，并且key的类型是String时可以采取sun.misc.Hashing.stringHash32方法获取其hash值。
需要注意的是，hashSeed的默认值是0，hashSeed会在capacity发生变化的时候调用initHashSeedAsNeeded方法重新计算，具体代码如下：

```java
final boolean initHashSeedAsNeeded(int capacity) {
	boolean currentAltHashing = hashSeed != 0;
	boolean useAltHashing = sun.misc.VM.isBooted() &&
			(capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
	boolean switching = currentAltHashing ^ useAltHashing;
	if (switching) {
		hashSeed = useAltHashing
			? sun.misc.Hashing.randomHashSeed(this)
			: 0;
	}
	return switching;
}
```

从上代码可以看到，hashSeed的计算流程涉及到一个设定值Holder.ALTERNATIVE_HASHING_THRESHOLD，该设定值是通过JVM的参数jdk.map.althashing.threshold来设置的。

注：
在JDK 8 中，hashSeed已经被移除掉了，移除掉的原因是调用sun.misc.Hashing.randomHashSeed计算hashSeed时会调用方法java.util.Random.nextInt()，该方法使用AtomicLong，在多线程情况下会有性能问题。

Java7中的无限循环：

Thread1和Thread2都进入扩容程序的时候可能会造成链表环

oldTable[i]----->1----->2----->3，正确扩容后(假设扩容后index没有变化)应该是newTable[i]----->3----->2----->1

Thread1对应newTable1，Thread2对应newTable2

1. 执行Thread1，newTable1[i]----->2----->1，挂起
2. 执行Thread2，newTable2[i]----->1，newTable2[i]----->2----->1，newTable2[i]----->3----->2----->1，挂起
3. 执行Thread1，此时会把2的next插入到2的前面，oldTable中2的next是3，Thread2后，2的next为1，故newTable1[i]----->1----->2----->1，这样1和2就成了死环

------

##### Java8中的HashMap

参考文章：

- [Map(一)之HashMap（java8）](http://blog.csdn.net/lchpersonal521/article/details/51899210)(有介绍java8中的bug)
- [JAVA_8 HashMap原理](http://blog.csdn.net/u014352080/article/details/53501867)

Java8中HashMap的数据结构：

![collector-hashmap-java8-structure](E:\typoraNote\picture\collector\collector-hashmap-java8-structure.png)

Java8中加入了红黑树，何时链表转树，见上图，当多次remove后，元素个数小于UNTREEIFY_THRESHOLD时，非树化，将树转为链表

```java
static final int TREEIFY_THRESHOLD = 8;
static final int MIN_TREEIFY_CAPACITY = 64;
static final int UNTREEIFY_THRESHOLD = 6;

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
					if (binCount >= TREEIFY_THRESHOLD - 1) //树化条件一
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

final void treeifyBin(Node<K,V>[] tab, int hash) {
	int n, index; Node<K,V> e;
	//这里是保证桶中元素和HashMap容量同时满足条件才对相应桶进行树化。否则只进行resize操作
	if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
		resize();
     //树化条件二，容量大于MIN_TREEIFY_CAPACITY
	else if ((e = tab[index = (n - 1) & hash]) != null) {
		TreeNode<K,V> hd = null, tl = null;
	//这里可以理解为把链表中每个结点都替换成树结点，实际上是创建一个新链表，结点类型由Node变为TreeNode
		do {
			//创建树结点
			TreeNode<K,V> p = replacementTreeNode(e, null);
			if (tl == null)
				hd = p;
			else {
				p.prev = tl;
				tl.next = p;
			}
			tl = p;
		} while ((e = e.next) != null);
		if ((tab[index] = hd) != null)
			//具体树化算法
			hd.treeify(tab);
	}
}

final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
	TreeNode<K,V> b = this;
	// Relink into lo and hi lists, preserving order
	TreeNode<K,V> loHead = null, loTail = null;
	TreeNode<K,V> hiHead = null, hiTail = null;
	int lc = 0, hc = 0;
	for (TreeNode<K,V> e = b, next; e != null; e = next) {
		next = (TreeNode<K,V>)e.next;
		e.next = null;
		if ((e.hash & bit) == 0) {
			if ((e.prev = loTail) == null)
				loHead = e;
			else
				loTail.next = e;
			loTail = e;
			++lc;
		}
		else {
			if ((e.prev = hiTail) == null)
				hiHead = e;
			else
				hiTail.next = e;
			hiTail = e;
			++hc;
		}
	}

	if (loHead != null) {
		if (lc <= UNTREEIFY_THRESHOLD)//非树化
			tab[index] = loHead.untreeify(map);
		else {
			tab[index] = loHead;
			if (hiHead != null) // (else is already treeified)
				loHead.treeify(tab);
		}
	}
	if (hiHead != null) {
		if (hc <= UNTREEIFY_THRESHOLD)//非树化
			tab[index + bit] = hiHead.untreeify(map);
		else {
			tab[index + bit] = hiHead;
			if (loHead != null)
				hiHead.treeify(tab);
		}
	}
}
```

resize方法：

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;//还没有拷贝就替换，此时get就为null
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // 可以保证链表按照原来的顺序复制，不会发生死环
                        Node<K,V> loHead = null, loTail = null;//index没有变化时的头尾链表指针
                        Node<K,V> hiHead = null, hiTail = null;//index变化时的头尾链表指针
                        Node<K,V> next;
                        do {
                            next = e.next;
                        //oldCap的值为2的幂，注意与oldCap-1的区别
                            if ((e.hash & oldCap) == 0) {//判断index是否变化，变化则为j+oldCap
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

从上面代码中可以发现两个bug：

- HashMap中有值，get返回的是null，见代码中注解


- 拷贝链表时，虽然不会发生死环，但是容易丢失数据

oldTable[i]---->1---->2---->3---->4---->5---->null

正确拷贝后：

newTable[i]---->1---->3---->4---->null，newTable[i+oldCap]---->2---->5---->null

两个线程Thread1和Thread2：

1. 执行Thread1，Thread1-loHead---->1---->3---->4，Thread1-hiHead---->2，挂起，此时Thread1-e---->5---->null
2. 执行Thread2，经过Thread1后，链表结构为：1---->3---->4---->5---->null，2---->3，故：Thread2-loHead---->1---->3---->4，Thread2-hiHead---->5，挂起
3. 执行Thread1，Thread1-loHead---->1---->3---->4---->null，Thread1-hiHead---->2---->5---->null，Thread1的拷贝结果正确
4. 执行Thread2，Thread2-loHead---->1---->3---->4---->null，Thread2-hiHead---->5---->null
5. 执行newTab[j + oldCap] = hiHead; Thread2比Thread1后执行则丢失数据3

------

#### ConcurrentHashMap

参考文章：

- [ConcurrentHashMap演进从Java7到Java8](https://segmentfault.com/p/1210000010020931/read)
- [ConcurrentHashMap原理分析](https://my.oschina.net/hosee/blog/639352)
- [ConcurrentHashMap总结](http://www.importnew.com/22007.html)

##### Java7中的ConcurrentHashMap

![collector-concurrenthashmap-java7-structure](E:\typoraNote\picture\collector\collector-concurrenthashmap-java7-structure.png)









------

##### Java8中的ConcurrentHashMap

![collector-concurrenthashmap-java8-structure](E:\typoraNote\picture\collector\collector-concurrenthashmap-java8-structure.png)









------

#### TreeMap



#### ConcurrentSkipListMap





#### ArrayList

##### 容量

```
1.ArrayList<Integer> list = new ArrayList<>();
  此时使用数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，默认初始容量为10
2.ArrayList<Integer> list = new ArrayList<>(0);
  此时使用数组EMPTY_ELEMENTDATA，当添加元素时会进入扩容，扩容为oldCapacity的1.5倍，与size+1取最大值，故加第一个元素后，容量为1
```

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;//2
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;//1
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {//minCapacity=size + 1
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);//最小容量为10
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)//minCapacity=size + 1
        grow(minCapacity);
}

//扩容
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//JDK1.8 1.5倍
    if (newCapacity - minCapacity < 0)//1.5oldCapacity与minCapacity=size + 1取最大值
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)//newCapacity大于最大值时，根据minCapacity来决定新容量
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?//新容量MAX_ARRAY_SIZE和Integer.MAX_VALUE
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```



#### CopyOnWriteArrayList

每次写都重新创建新数组，并复制老数组到新数组，实现读写分离，但读数据不是实时的

```java
private transient volatile Object[] array;
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;//array指向新数组
}

public E get(int index) {
    return get(getArray(), index);
}
final Object[] getArray() {
    return array;
}
```

#### HashSet



#### CopyOnWriteArraySet



#### TreeSet



#### ConcurrentSkipListSet