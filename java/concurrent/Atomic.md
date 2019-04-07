### Atomic

AtomicInteger中的一段代码：

```java
public final int incrementAndGet() {
    //getAndAddInt返回当前值再加1，底层使用unsafe的cas方法
    //通过this, valueOffset两个参数就可以读取及修改value值
	return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

private static final long valueOffset;//value的内存地址
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

#### AomicLong与LongAdder

参考：[AtomicLong和LongAdder的区别](https://blog.csdn.net/yao123long/article/details/63683991)

AtomicLong底层使用死循环进行cas操作，并发量大时可能会长时间无法cas成功，会浪费资源，使性能降低。

LongAdder在AtomicLong的基础上将单点的更新压力分散到各个节点，在低并发的时候通过对base的直接更新可以很好的保障和AtomicLong的性能基本保持一致，而在高并发的时候通过分散提高了性能。缺点是LongAdder在统计的时候如果有并发更新，可能导致统计的数据有误差。

LongAdder部分代码：

```java
public void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    //cells为空，尝试直接更新base，更新成功直接返回（低并发场景）
    //cells不为空或者cas更新base失败
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        boolean uncontended = true;//是否存在竞争
        //cells为空表示以前不存在竞争，所以上面标志为true
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}

public long sum() {
    Cell[] as = cells; Cell a;
    long sum = base;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

ABA解决：AtomicStampedReference

用的较少：

```
AtomicReferenceArray
AtomicReference
```

更新对象的属性：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater，属性必须为volatile修饰的整型数据