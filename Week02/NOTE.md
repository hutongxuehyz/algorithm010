学习笔记：
HashMap  put()方法总结:

①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。

JDK1.8HashMap的put方法源码如下:

	public V put(K key, V value) {
		//对key的hashCode()做hash,如果key的hashCode值一样，则哈希值一样，哈希值一样则存在哈希冲突
        return putVal(hash(key), key, value, false, true);
    }
	
	
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
		// 步骤①：tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
		// 步骤②：计算index，并对null做处理 
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
			// 步骤③：节点key存在，（存在哈希冲突并且key相等）,新值覆盖旧值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
			// 步骤④：判断该链表是否为红黑树（如果为红黑树的话，直接插入到树中）
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
				// 步骤⑤：遍历当前链表，如果链表中存在key的哈希值与key值和当前key相等，则覆盖旧值，如果遍历完都不相同，则将当前键值对放入链表头节点
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
							//如果单前链表节点数大于等于8，链表结构转换成红黑树，该方法里会有第二层判断（HashMap的容量大于等于64才进行转红黑树）
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
		// 步骤⑥：判断键值对数量是否大于最大容量，如果大于就扩容（扩容之后容量为原来的2倍）
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

 
//只要HashMap里添加的键值对大于HashMap的容量*负载因子，HashMap就会扩容，当HashMap的容量小于64时，即使单个哈希桶里链表的节点数大于等于8，链表也不会转成红黑树，会先扩容一次
//链表转红黑树需要满足2个条件，HashMap的容量大于等于64以及链表的节点数大于等于8

	final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
		//第二层判断，HashMap的容量大于等于64才进行转红黑树，小于64时进行扩容
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
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
                hd.treeify(tab);
        }
    }



测试代码：

 	@Test
    public void test01(){
        HashMap<School, String> map = new HashMap<>(4,0.75f);
        School school1 = new School("123");
        School school2 = new School("321");
        School school3 = new School("114");
        School school4 = new School("141");
        School school5 = new School("213");
        School school6 = new School("231");
        School school7 = new School("015");
        School school8 = new School("006");
        School school9 = new School("600");
        School school10 = new School("060");
        School school11 = new School("303");
        School school12 = new School("330");
        School school13 = new School("222");
        School school14 = new School("229");
        map.put(school1,"hyz");
        map.put(school2,"hyz");
        map.put(school3,"hyz");
        map.put(school4,"hyz");
        map.put(school5,"hyz");
        map.put(school6,"hyz");
        map.put(school7,"hyz");
        map.put(school8,"hyz");
        map.put(school9,"hyz");
        map.put(school10,"hyz");
        map.put(school11,"hyz");
        map.put(school12,"hyz");
        map.put(school13,"hyz");
        map.put(school14,"hyz");
    }
    public class School{
        public String name;

        public School(String name) {
            this.name = name;
        }
        @Override
        public int hashCode() {
            String name = this.name;
            int count = 0;
            char[] chars = name.toCharArray();
            for(int i = 0; i < chars.length; i++){
                count += Integer.valueOf(chars[i]);
            }
            return count;
        }

        @Override
        public boolean equals(Object obj) {
            return this.name.equals(obj);
        }
    }