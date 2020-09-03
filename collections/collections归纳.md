###java集合类型归纳总结


#####java集合框架的类图    
![avar](../imags/collections/collections-01.png)    

- ArrayList 源码分析
```text
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final int DEFAULT_CAPACITY = 10;//默认初始容量
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};//默认空数组
    transient Object[] elementData; //存储数据的数组
    private int size;//当前数组的大小
    
    //自定义了初始容量
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
        //初始化指定容量大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;//默认空数组
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    
    //默认空数组
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    //指定默认集合，尽心元素拷贝
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
    
    public boolean add(E e) {
        //数组的扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;//存入新增元素
        return true;
    }
    
    //比较容量大小，如果给定容量大于初始容量，则返回给定容量，否则初始容量
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)//如果给定容量已经超过数组容量时，则需要扩容
            grow(minCapacity);
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;//旧的数组长度
        int newCapacity = oldCapacity + (oldCapacity >> 1);//新数组长度=旧长度+旧长度/2
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);//数组元素的拷贝
    }
}
```

- LinkedList源码分析
```text
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;
    
    //说明linkedList是一个双向链表的形式
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    //默认在队尾添加
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;//将尾节点指向当前节点
        if (l == null)//如果尾节点为null，则表明是一个空的链表
            first = newNode;//则将头节点指向当前节点
        else
            l.next = newNode;//否则就需要将旧的尾节点的下一个节点指向当前节点，即当前节点为队尾节点
        size++;
        modCount++;
    }
    
    public void addFirst(E e) {
        linkFirst(e);
    }
    
    private void linkFirst(E e) {
        final Node<E> f = first;//队首节点
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;//将队首节点指向当前节点
        if (f == null)//空链表
            last = newNode;//将队尾节点指向当前节点
        else
            f.prev = newNode;//旧的队首节点的前一个节点指向当前节点
        size++;
        modCount++;
    }
}
```

需要说明一下的是，不论是LinkedList还是ArrayList也好，都是有序的且可以重复的，当是需要注意的是，他们不是线程安全的


- HashSet源码分析
```text
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;//hashSet底层默认是用HashMap实现的
    private static final Object PRESENT = new Object();//默认的value
    
    public HashSet() {
        map = new HashMap<>();
    }
    
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    
    //这里是用LinkedHashMap构造有序的hashSet，这里主要是给LinkedHashSet使用的，外部人员是无法构造的，但是可以通过反射构造
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
}
```

- TreeSet源码分析
```text
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    private transient NavigableMap<E,Object> m;
    private static final Object PRESENT = new Object();//默认的value
    
    TreeSet(NavigableMap<E,Object> m) {//NavigableMap要求是必须有需的
        this.m = m;
    }
    
    public TreeSet() {
        this(new TreeMap<E,Object>());//默认情况下TreeSet是采用TreeMap实现的
    }
    
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }

}
```

####java Map接口实现
![avar](../imags/collections/collections-02.png)
- HashMap的源码分析
```text
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;//默认的初始容量，思考为什么是16   
    static final int MAXIMUM_CAPACITY = 1 << 30; //最大的数组容量 1 << 30为为2的30次方，思考为什么是1 << 30
    static final float DEFAULT_LOAD_FACTOR = 0.75f;//扩容因子
    static final int TREEIFY_THRESHOLD = 8;//链表允许的最大长度，超过此长度即会将链表转化成红黑树 思考为什么是8
    static final int UNTREEIFY_THRESHOLD = 6;//缩容时，链表小于此长度，会将红黑树转成链表 思考为什么是6
    static final int MIN_TREEIFY_CAPACITY = 64;//链表转红黑树时，要求链表的长度大于等于8，且数组的元素个数超过64才可以 思考为什么是64
    
    
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;//从这里可以看出针对hash碰撞，HashMap采用的是链地址法

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);  //Node的hash实际上就是key的hash亦或value的hash，其用意就是为了hash散列的更均匀
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) && //Objects.equals()比较的就是地址或者是对象的equals()方法
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
    
    transient Node<K,V>[] table;//从这里可以看出我们HashMap的底层就是数组+链表或者红黑树
    
    transient int size;//当前存入数组的元素大小
    
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    static final int hash(Object key) {
        int h;//实际上是32位int值
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//高16位亦或低16位，得到的这个散列值更加的均匀
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                       boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;//第一次put时，初始化数组，默认大小16，默认扩容值12
        if ((p = tab[i = (n - 1) & hash]) == null) //获取元素在数组中的下标
            tab[i] = newNode(hash, key, value, null);//如果当前下标没有元素，则存入当前下标
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))//如果当前下标就是该元素
                e = p;
            else if (p instanceof TreeNode)//如果当前下标是红黑树
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {//否则就是链表，需要遍历
                    if ((e = p.next) == null) {//从队尾加入元素
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st 队尾元素超过7
                            treeifyBin(tab, hash);//进行红黑树转化
                        break;
                    }
                    if (e.hash == hash && //遍历找到当前元素，则在后面的操作中修改
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;//在链表中找到了当前元素
                }
            }
            if (e != null) { // existing mapping for key 如果找到当前元素
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)//!onlyIfAbsent put操作返回的是false
                    e.value = value;//替换旧的值
                afterNodeAccess(e);//给子类一个在put后的操作的机会
                return oldValue;//返回旧的值
            }
        }
        ++modCount;
        if (++size > threshold) //如果当前数组的元素个数超过了容量值则扩容
            resize();
        afterNodeInsertion(evict);////给子类一个在put后的操作的机会
        return null;
    }
    
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;//旧的容量
        int oldThr = threshold;//旧的容量值
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {//超过最大容器
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold  否则就扩容一倍
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            //第一次初始化
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
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//构建数组
        table = newTab;
        if (oldTab != null) {//如果是扩容，则需要对数据进行迁移
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {//当前下标没有数据，不进行迁移
                    oldTab[j] = null;
                    if (e.next == null) //如果当前下标只有一个元素，没有链式结构
                        newTab[e.hash & (newCap - 1)] = e;//重新计算元素的下标，进行数据迁移
                    else if (e instanceof TreeNode)//如果当前是红黑树结构，则需要对红黑树进行拆分
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order 否则就是链表结构
                        Node<K,V> loHead = null, loTail = null;//低位的头，尾节点
                        Node<K,V> hiHead = null, hiTail = null;//高位的头，尾节点
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //oldCap旧的容量值，以16为例...01 0000 e.hash & oldCap) == 0，如果该值为0，则表明，则表明第五位一定是0，
                            //其他位可能是1也可能是0，也即在新的数组中，如果再去进行hash计算下标，该元素一定在低位，
                            //(e.hash & oldCap) == 0 这里只有两种情况要为为1要么为0，因为oldCap是2的幂次方 以16为例 01000 ，所以任何
                            //数与oldCap进行与运算一定是0或者是1，也即用这个来区别高位和低位，即扩容期的下标，和扩容后的下标
                            if ((e.hash & oldCap) == 0) {//低位迁移 构建低位的链表
                                if (loTail == null)//如果低位尚未迁移，即低位的尾节点为空，
                                    loHead = e;//则将低位的头节点指向当前元素
                                else
                                    loTail.next = e;//从低位队尾加入元素
                                loTail = e;
                            }
                            else {//高位迁移
                                if (hiTail == null)//如果高位尚未迁移，即高位的尾节点为空
                                    hiHead = e;//则将高位的头节点指向当前元素
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
    
}
```
HashMap的数据结构
![avar](../imags/collections/collections-03.png)

resize扩容高低位迁移示意图        
![avar](../imags/collections/collections-04.png)
补充知识hash冲突的四种解决方案
- 开放地址法
    1.线性
    2.平方
    3.随机

- 拉链法(链地址法)

- 再hash

- 暂存溢区