### ThreadLocal



#### 小结

1. Thread中有个成员变量ThreadLocal.ThreadLocalMap threadLocals，该map的key为threadLocal对象，且该key为弱引用；
2. 如一个ThreadLocal对象被多个线程使用，则每个线程中都有一个map，key为该ThreadLocal对象，value初始值都一样，这样每个线程都有了value的副本，互不干扰（引用对象除外）；
3. 通过ThreadLocal设置获取值的方法都会先获取当前线程，然后获取map，就可以设置获取值了；
4. 每个Thread可以有多个ThreadLocal，存入map中，不同于hashMap处理碰撞的方式（拉链法），这里使用的是开放地址法。

