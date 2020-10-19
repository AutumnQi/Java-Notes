# 二叉搜索树

>  binary search tree, BST

定义：对于任意一个节点，满足右子树中的所有元素都大于本节点，左子树都小于本节点的二叉树

用途：用来存储有序数据，可以实现较为高效的插入和查询

### 插入新节点

按照最基本的比较规则

### 删除节点

1. 没有子节点则直接删除
2. 有一个子节点则直接将子节点连接到父节点
3. 有两个子节点则找到该节点的前继/后继节点来进行替换，再使用1，2条规则删除前继/后继节点

# 平衡二叉搜索树

> Self-balancing binary search tree

定义：任何一个节点左右子树的高度相差不超过1的二叉搜索树

用途：适用于修改少，查询多的场景，拥有最高效的比较查询效率（树和hashmap不同，其中的每个节点必须是可比的）

### 插入新节点

### 删除节点

# 红黑树

> 目的是高效实现自平衡的二叉树以提高搜索的效率

红黑树，Red-Black Tree 「RBT」是一个自平衡(不是绝对的平衡)的二叉查找树(BST)，树上的每个节点都遵循下面的规则:

1. 每个节点都有红色或黑色
2. 树的根始终是黑色的 (黑土地孕育黑树根， )
3. 没有两个相邻的红色节点（红色节点不能有红色父节点或红色子节点，**并没有说不能出现连续的黑色节点**）
4. 从节点（包括根）到其任何后代NULL节点(叶子结点下方挂的两个空节点，并且认为他们是黑色的)的每条路径都具有相同数量的黑色节点

**红黑树虽然不是绝对平衡，但可以保证两边子树的到每个叶节点所经过的黑色节点数量相同，来拟合平衡二叉树的效果**

### 红黑树有两大操作:

1. recolor (重新标记黑色或红色)
2. rotation (旋转，这是树达到平衡的关键)

### 红黑树插入新节点的规则

1. 将新插入的节点标记为红色
2. 如果 X 是根结点(root)，则标记为黑色
3. 如果 X 的 parent 不是黑色，同时 X 也不是 root:

- 3.1 如果 X 的 uncle (叔叔) 是红色

- - 3.1.1 将 parent 和 uncle 标记为黑色
  - 3.1.2 将 grand parent (祖父) 标记为红色
  - 3.1.3 让 X 节点的颜色与 X 祖父的颜色相同，然后将祖父设置为新的X节点重复步骤 2、3

- 3.2 如果 X 的 uncle (叔叔) 是黑色，我们要分四种情况处理

- - 3.2.1 左左 (P 是 G 的左孩子，并且 X 是 P 的左孩子)
  - ![img](https://picb.zhimg.com/80/v2-9e139a0f8b4a5e00ca8e643e2130403c_1440w.jpg)
  - 3.2.2 左右 (P 是 G 的左孩子，并且 X 是 P 的右孩子)
  - ![img](https://pic1.zhimg.com/80/v2-3fb33fbb3a42e34ed8a058a047a44cc3_1440w.jpg)
  - 3.2.3 右右 (和 3.2.1 镜像过来，恰好相反)
  - ![img](https://pic2.zhimg.com/80/v2-62a42ada09cb4547191aa4b9051c7c23_1440w.jpg)
  - 3.2.4 右左 (和 3.2.2 镜像过来，恰好相反)
  - ![img](https://pic1.zhimg.com/80/v2-210d1400ea1b098dfe1582589a6064c3_1440w.jpg)


 ### 插入原理总结

- 1. 每次新节点的插入完毕后，其子节点为两个空的黑色节点，将其自身标记为红色
  2. 在新节点的父节点和uncle节点都为黑色时，不需要额外的操作
  3. 父节点为红色时产生规则冲突，逐层往上recolor和rotation
     1. 若uncle节点为红色，则只需要将三个节点recolor（father、uncle、grandpa）
     2. 若uncle节点为黑色，则需要根据情况做左旋或右旋来对树的高度进行调整

 ### 删除动作规则

- 红黑树和二叉搜索树的删除类似，只不过加上颜色属性（**这里的子节点均指非NULL节点**）：

- 1. 无子节点时，删除节点可能为红色或者黑色；
      1.1 如果为红色，直接删除即可，不会影响黑色节点的数量；
      1.2 如果为黑色，则需要进行删除平衡的操作了；
  2. 只有一个子节点时，删除节点只能是黑色，其子节点为红色，否则无法满足红黑树的性质了。 此时用删除节点的子节点接到父节点，且将子节点颜色涂黑，保证黑色数量。
  3. 有两个子节点时，与二叉搜索树一样，使用后继节点作为替换的删除节点，情形转至为1或2处理。

- ![img](https://upload-images.jianshu.io/upload_images/7779607-185008085d0b4fba.png?imageMogr2/auto-orient/strip|imageView2/2/w/751/format/)

- 故最复杂的一步操作在1.2中实现，以上图为例来进行讲解

 ##### 前提条件：

- 1. 出现图中的情况意味着P和N中间有一个黑色的节点D已经被删除，N被recolor为黑色
  2. 用h(A->B->叶子)表示从A走到B再走到某一个叶子路径的黑色节点数量（A与B，B与叶子之间可能间隔了多个节点）

 D被删除后经过N的路径的黑色数量减1，即h(P->N->叶子) 比 h(P->S->叶子) 少1，采用以下方式进行平衡：

  1. h(P->N->叶子)不变，h(P->S->叶子)减1，此时已经子平衡；然而h(GP->P->叶子)还是会比h(GP -> U ->叶子)少1。此时需要将P当作新的N，向上递归处理；
  2. h(P->N->叶子)加1，h(P->S->叶子)不变，也就是恢复了原来的状态，此时已经平衡，因为h(GP->P->叶子)=h(GP -> U ->叶子)。

 以上两种方式放在具体的操作中可以表现为下表：

 ![img](https://upload-images.jianshu.io/upload_images/7779607-e1716801d002f2b1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/)

 ### 总结

- 红黑树与平衡搜索二叉树AVL相比，高效地实现了增删查三种功能，该数据结构被广泛应用如：

- 1. Linux内核中的完全公平调度器、高精度计时器、ext3文件系统、虚拟内存的管理
  2. Java的TreeMap和TreeSet，hashMap等
  3. C++ STL的map、multimap、multiset等

- ```java
  //节点定义
  class  Node<T>{
     public  T value;
     public   Node<T> parent;
     public   boolean isRed;
     public   Node<T> left;
     public   Node<T> right;
  }
  ```

# B-树

#### 引入B树的原因

- 文件系统和数据库一般都是存在电脑硬盘上的，如果数据量太大的话不一定能一次性加载到内存中。（一棵树不能一次性加载完怎么查找对吧？）但是B树可以多路存储。也正因为B树的这一个优点，可以在文件查找的时候每次只加载一个节点的内容存入内存来查找。而红黑树在内存中查找非常块，但是如果在数据库和文件系统中，显然B树更优。

B-树不同于前面的二叉搜索树，B-树是一种多路搜索树（并不一定是二叉的）

一棵m阶B树(balanced tree of order m)是一棵平衡的m路搜索树。它或者是空树，或者是满足下列性质的树：

  1. 根结点至少有两个子女；
  2. 每个非根节点所包含的关键字个数 j 满足：┌m/2┐ - 1 <= j <= m - 1；
  3. 除根结点以外的所有结点（不包括叶子结点）的度数正好是关键字总数加1，故内部子树个数 k 满足：┌m/2┐ <= k <= m ；
  4. 所有的叶子结点都位于同一层。

![B-树](https://img-blog.csdn.net/20160805191715603)

##### B-树的特性：

1. 关键字集合分布在整颗树中；

2. 任何一个关键字出现且只出现在一个结点中；

3. 搜索有可能在非叶子结点结束；

4. 其搜索性能等价于在关键字全集内做一次二分查找；

5. 自动层次控制；



# B+树

B+ 树是一种树数据结构，是一个n叉树，每个节点通常有多个孩子，一棵B+树包含根节点、内部节点和叶子节点。根节点可能是一个叶子节点，也可能是一个包含两个或两个以上孩子节点的节点。

##### 用途：

B+树多用于数据库中的索引。

那么为什么B+树用于数据库中的索引呢？
因为在数据库中select常常不只是查询一条记录，常常要查询多条记录。比如：按照id的排序的后10条。如果是多条的话，B树需要做中序遍历，可能要跨层访问。而B+树由于所有数据都在叶子结点，不用跨层，同时由于有链表结构，只需要找到首尾，通过链表就能够把所有数据取出来了。

B+ 树通常用于[数据库](http://lib.csdn.net/base/14)和操作系统的文件系统中。NTFS, ReiserFS, NSS, XFS, JFS, ReFS 和BFS等文件系统都在使用B+树作为元数据索引。B+ 树的特点是能够保持数据稳定有序，其插入与修改拥有较稳定的对数时间复杂度。B+ 树元素**自底向上插入**。

1. 有n棵子树的结点中含有n个关键字，每个关键字不保存数据，只用来索引，所有数据都保存在叶子节点。

2. 所有的叶子结点中包含了全部关键字的信息，及指向含这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。

3. 所有的非终端结点可以看成是索引部分，结点中仅含其子树（根结点）中的最大（或最小）关键字。 
     通常在B+树上有两个头指针，一个指向根结点，一个指向关键字最小的叶子结点。

![B+树](https://img-blog.csdn.net/20160805192039968)

 ##### B+的特性：

 1. 所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的；

 2. 不可能在非叶子结点命中；

 3. 非叶子结点相当于是叶子结点的索引（稀疏索引），叶子结点相当于是存储（关键字）数据的数据层；

 4. 更适合文件索引系统；

# B*树

 > 是B+树的变体，在B+树的非根和非叶子结点再增加指向兄弟的指针；

​                         ![B*树](https://img-blog.csdn.net/20160805192156312)



**B\*树定义了非叶子结点关键字个数至少为(2/3)*M，即块的最低使用率为2/3**

- B+树的分裂：当一个结点满时，分配一个新的结点，并将原结点中1/2的数据复制到新结点，最后在父结点中增加新结点的指针；B+树的分裂只影响原结点和父结点，而不会影响兄弟结点，所以它不需要指向兄弟的指针；
- B\*树的分裂：当一个结点满时，如果它的下一个兄弟结点未满，那么将一部分数据移到兄弟结点中，再在原结点插入关键字，最后修改父结点中兄弟结点的关键字（因为兄弟结点的关键字范围改变了）；如果兄弟也满了，则在原结点与兄弟结点之间增加新结点，并各复制1/3的数据到新结点，最后在父结点增加新结点的指针；所以，B*树分配新结点的概率比B+树要低，空间使用率更高；



# 二叉搜索树对比哈希表不具备的优点：

1. 二叉搜索树用中序遍历可以很容易对数据进行排序，这个是哈希表做不到的。
2. 哈希表遇到哈希冲突的时候需要扩容，这个扩容操作相当耗时，性能有时候不稳定，虽然说二叉搜索树的性能也不稳定， 但是 我们有更特殊的 平衡二叉搜索树可以把时间复杂度稳定在O(logn)
3. 二叉搜索树比较简单，数据结构一目了然，递归递归递归就行了，但是哈希表你们懂的，实现起来超级复杂。



# HashMap中链表和红黑树的转化

```java
/**
			the expected occurrences of
     * list size k are (exp(-0.5) * pow(0.5, k) / factorial(k)). The
     * first values are:
     *
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million		


     * The load factor for this table. Overrides of this value in
     * constructors affect only the initial table capacity.  The
     * actual floating point value isn't normally used -- it is
     * simpler to use expressions such as {@code n - (n >>> 2)} for
     * the associated resizing threshold.
     */
    private static final float LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2, and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * The value should be at least 4 * TREEIFY_THRESHOLD to avoid
     * conflicts between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

1. 数组什么时候扩容？
   1. 在新建hashMap的时候可以指定参数float loadFactor，默认为0.75
   2. 每个数组的初始长度为length（2的幂），当hashmap内的Node数量大于length*loadFactor时进行扩容
   3. 将数据长度修改为2*length
2. 链表什么时候转化为红黑树？
   1. 条件一：数组的长度已经超过64
   2. 条件二：链表的长度大于等于8
   3. 为什么选择8，假设数据满足泊松分布，某个位置上冲突超过8次的概率为0.00000006，保证了只有在小概率下才会更改数据格式
3. 红黑树什么时候转化为链表？
   1. 条件：链表的长度小于6
   2. 中间的7是为了避免频繁的转化，为什么选择6？
      1. 链表查找的时间复杂度为O(n/2)
      2. 红黑树查找的时间复杂度为O(logn)
      3. 时间开销在n=6左右平衡
4. 如何转化？

# Reference

[CSDN博客](https://blog.csdn.net/ff_simon/article/details/101055134?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

[美团技术博客](https://tech.meituan.com/2016/12/02/redblack-tree.html)

[简书博客](https://www.jianshu.com/p/84416644c080)

