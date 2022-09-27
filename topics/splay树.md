# splay树 (树ADT连载 5/13)

❗️ **【NEW】** ❗️

- 9-15:  [红黑树从入门到看开](https://leetcode.cn/circle/discuss/SwgIJV/)

这是小白 yuki 推出的「树ADT」系列文章的第 5 篇 (5/13) 。

***

> $keywords$ :
>
> splay树 / 展开 / 自顶向下展开 / 摊还时间上的平衡



本文介绍一种易于实现的平衡二叉查找树 –– splay树。实际上本文是为了后续讲解多种 **平衡二叉树** 的前置文章，最终目的是讲解 **「红黑树」** (已推出，[红黑树从入门到看开](https://leetcode.cn/circle/discuss/SwgIJV/))。

本文重点在于展现如何设计一个 splay树 类，分析主要方法的代码实现，并给出该类的完整实现代码。读者学习之后 **可以自己写出一个方法较为完备的splay树类** 。

***

yuki的其他文章如下，欢迎阅读指正！

| 文章                                                         | [发布时间] 字数/览/藏/赞 (~09-23)        |
| ------------------------------------------------------------ | ---------------------------------------- |
| [十大排序从入门到入赘](https://leetcode.cn/circle/discuss/eBo9UB/)  🔥🔥🔥 | [20220516]  2.5万字/58.2k览/3.5k藏/865赞 |
| [二分查找从入门到入睡](https://leetcode.cn/circle/discuss/ooxfo8/) 🔥🔥🔥 | [20220509]  2.3万字/35.9k览/2.1k藏/484赞 |
| [并查集从入门到出门](https://leetcode.cn/circle/discuss/qmjuMW/) 🔥🔥 | [20220514]  1.2万字/15.1k览/924藏/279赞  |
| [图论算法从入门到放下](https://leetcode.cn/circle/discuss/FyPTTM/) 🔥🔥 | [20220617]  5.6万字/17.3k览/1.2k藏/341赞 |
| 树ADT系列 (预计13篇)                                         | 系列文章，连载中                         |
| 3. [二叉查找树](https://leetcode.cn/circle/discuss/wPzlSb/)  | [20220801]  5千字                        |
| 4. [AVL树](https://leetcode.cn/circle/discuss/zbwD3p/)       | [20220817]  5千字                        |
| 5. [splay树](https://leetcode.cn/circle/discuss/BCK17f/)     | [20220817]  5千字                        |
| 6. [红黑树从入门到看开](https://leetcode.cn/circle/discuss/SwgIJV/) 🔥🤯🤯🤯 | [20220915]  3万字/3.1k览/186藏/52赞      |
| 10. [树状数组从入门到下车](https://leetcode.cn/circle/discuss/qGREiN/) 🔥🤯 | [20220722]  1.1万字/3.9k览/129藏/49赞    |
| 11. [线段树从入门到急停](https://leetcode.cn/circle/discuss/H4aMOn/) 🔥🤯 | [20220726]  2.5万字/4.8k览/324藏/93赞    |
| [图论相关证明系列](https://leetcode.cn/circle/discuss/GV0JrV/) | 系列文章                                 |
| 1. [Dijkstra正确性证明](https://leetcode.cn/circle/discuss/jJQn7V/) 🤯 | [20220531]                               |
| 2. [Prim正确性证明](https://leetcode.cn/circle/discuss/VVEc8f/) 🤯 | [20220919]                               |
| 3. [Bellman-Ford及SPFA正确性证明](https://leetcode.cn/circle/discuss/xeEwYl/) | [20220602]                               |
| 4. [Floyd正确性证明](https://leetcode.cn/circle/discuss/Nbzix4/) | [20220602]                               |
| 5. [最大流最小割定理证明](https://leetcode.cn/circle/discuss/tMIy36/) 🤯🤯 | [20220719]                               |
| 6. [Edmonds-Karp复杂度证明](https://leetcode.cn/circle/discuss/tN3sZc/) 🤯🤯 | [20220515]                               |
| 7. [Dinic复杂度证明](https://leetcode.cn/circle/discuss/T9Xa1R/) 🤯🤯 | [20220531]                               |



***

[TOC]

***

### splay树

伸展树是解决基本 BST 在非平衡状态下主要操作无法达到 $O(logn)$ 复杂度的另一种解决方案。相比 AVL 树，伸展树并不严格地使 BST 每次操作后保持平衡，因此伸展树的单次操作可能为 $O(n)$ 复杂度，但伸展树保证 $m$ 次操作的最坏时间复杂度为 $mO(logn)$，也就是说其主要操作的 **均摊时间复杂度** 为 $O(logn)$  。伸展树相对 AVL 树的优点是 **无需维护树的高度信息** ，从而能够节省一部分空间，且编程相对简单。伸展树的主要操作均基于 **「展开 (splay)」** ，展开操作的基础是我们已在 AVL 中讲解过的旋转， **「展开」动作将目标结点旋转至根结点处，并在这个过程中调整沿途上的结点** 。本节将介绍一种只需 $O(1)$ 辅助空间的「自顶向下」展开的伸展树。


> 伸展树由 [Daniel Sleator](https://en.wikipedia.org/wiki/Daniel_Sleator) 和 [Robert Tarjan](https://en.wikipedia.org/wiki/Robert_Tarjan) 共同发明，发表于1985年的 [论文](https://www.cs.cmu.edu/~sleator/papers/self-adjusting.pdf) 中。
>
> 本文内容为Mark Allen Weiss所著 [数据结构与算法分析：Java语言描述](https://awesome-programming-books.github.io/algorithms/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%EF%BC%9AJava%E8%AF%AD%E8%A8%80%E6%8F%8F%E8%BF%B0.pdf) 相关章节的整理和总结。代码亦来自该书，略作改动。

<br />

#### 展开

伸展树「展开」的想法基于这样一个事实：当一个结点被访问时 (查询最小/最大/是否存在/插入/删除等)，它很可能将在不久之后被再次访问。因此为了提高总体访问效率，在访问一个结点后，考虑将其移动到根结点处， **移动过程即「展开」** 。在 AVL 树一节中我们已经掌握如何通过旋转向一个方向移动目标结点，很容易想到通过有限次的「单旋转」将目标结点展开至根结点处 (使其成为根结点)。

以下图为例，通过简单旋转将左侧树的结点 1 展开至根处得到右侧树。同时我们希望在展开过程中能够降低树高，而下图单旋转方式显然没有改变树高。

![image.png](https://pic.leetcode-cn.com/1660706904-GveWEq-image.png)

为了使得展开同时具有降低树高的效果，伸展树采用 **双旋转与单旋转结合** 的方式展开。主要为以下三种情形，每种情形有对称的两种子情形 (不再列出)。

```text
单旋转: 与 AVL 树中介绍的单旋转相同，分为左单旋状和右单旋转。
之字形双旋转：与 AVL 树中介绍的之字形双旋转相同，实际上就是连续两次不同向的单旋转，分为左右双旋转和右左双旋转。
一字形双旋转：实际上就是连续两次同向的单旋转，分为左左双旋转和右右双旋转。
```

如下图左侧是我们在 AVL 树一节中已经熟悉的之字形左右双旋转 (先左旋 $P-X$ ，再右旋 $X-G$ )，右侧是一字形右右双旋转 (先右旋 $G-P$ ，再右旋 $P-X$ )。

![image.png](https://pic.leetcode-cn.com/1660706954-RvPxlZ-image.png)

伸展树中的「展开」操作要求 **在能够双旋转时采用双旋转，只能单旋转时才执行单旋转** 。现在我们利用此规则展开前面给出的 1 到 6 的链状树，如下，展开过程为两次一字形的双旋转和一次单旋转。尽管由于结点树较少而无法很好地看出树高减小的效果，但总体上，相比简单的单旋转，我们确实能够看到树高减小的效果。

![image.png](https://pic.leetcode-cn.com/1660707012-AlXOuk-image.png)

<br />

#### 自顶向下展开

展开操作的前提是找到目标结点，因此需要一次遍历来找到目标结点，然后再沿着此路径将目标结点旋转至根结点，这显然需要保存路径信息。在 AVL 树中我们以递归调用的方式，借助隐式保存了路径信息的递归栈实现上述做法。这里我们将介绍一种 **无需栈空间的「自顶向下」展开** ，即无需遍历找到目标后再展开，而是在自顶向下寻找目标的过程中实时地展开。

在前面的例子中我们没有展示之字形旋转的过程，实际上，本节给出的「自顶向下」的展开实现中， **对之字形的情况按照「单旋转」处理** 。此外，展开的过程中始终维护树 $L$ 、树 $M$ 以及树 $R$ ，使得旋转操作得到简化，其定义如下。

>  整棵树中小于中间树 $M$ 的根结点但不在 $M$ 中的结点存储在 $L$ 树中，大于中间树 $M$ 的根结点但不在 $M$ 中的结点存储在 $R$ 树中。初始时 $M$ 即为原树，$L, R$ 为空树。

下图从上到下表示 **单旋转、之字形旋转 (同单旋转) 以及一字形旋转** 。可以看到，与 AVL 的旋转相比，这里的「单旋转」实际上只是修改了一些链接关系 ( AVL 单旋转本质也是修改一些链接关系，但重点在于旋转主体目标结点相对关系的改变，例如下图的单旋转，若为 AVL 中的单旋转，则 $X$ 到 $Y$ 的链接「右旋」后将变为 $Y$ 到 $X$ 的链接)，而「一字形旋转」也只是执行了一次 $Y-X$ 右旋后修改了一些链接关系。因此相比 AVL 树的实现，「自顶向下」伸展树的代码十分易于编写。

![image.png](https://pic.leetcode-cn.com/1660707064-QHwUvv-image.png)

实际编程中应用了一些技巧，例如利用 $nullNode$ 来简化程序并提高了程序效率。通过设置 $header$ 来保存 $L$ 和 $R$ 首次不为空树时的树根，以便于最后的 **组装** 操作。下图表现了展开操作最后的组装动作，同样只是简单地修改一些链接关系。具体实现以及提到的一些实现技巧在初看时较难理解，因此我在本文「主要方法」以及「类的实现代码」中提供了详细注释，请读者务必仔细阅读。

![image.png](https://pic.leetcode-cn.com/1660707113-IQMlgo-image.png)

<br />


#### 伸展树类架构

以下是伸展树 SplayTree 架构。

| 类成员/方法                                           | 描述                                 |
| ----------------------------------------------------- | ------------------------------------ |
| `private BinaryNode<E> root`                          | 字段，splay树的根结点                |
| `private BinaryNode<E> newNode`                       | 字段，插入结点                       |
| `private BinaryNode<E> nullNode`                      | 字段，为方便编程而设置的 $null$ 结点 |
| `public SplayTree()`                                  | 构造器                               |
| `public void makeEmpty()`                             | 树置空                               |
| `public boolean isEmpty()`                            | 树判空                               |
| `public void insert(E e)`                             | 插入结点方法                         |
| `public void remove(E e)`                             | 删除结点方法                         |
| `public E findMin()`                                  | 查找最小结点驱动方法                 |
| `public E findMax()`                                  | 查找最大结点驱动方法                 |
| `public boolean contains(E e)`                        | 判断是否包含指定元素方法             |
| `public void printTree()`                             | 按中序遍历打印树的驱动方法           |
| `public int size()`                                   | 求树的结点个数驱动方法               |
| `private BinaryNode<E> splay(E e, BinaryNode<E> t)`   | 展开方法                             |
| `private BinaryNode<E> findMin(BinaryNode<E> t)`      | 返回树的最小结点                     |
| `private BinaryNode<E> findMax(BinaryNode<E> t)`      | 返回树的最大结点                     |
| `private void printTree(BinaryNode<E> t)`             | 中序遍历打印树                       |
| `private int size(BinaryNode<E> t)`                   | 递归地遍历所有结点，返回结点总数     |
| `private BinaryNode<E> rotateRight(BinaryNode<E> k2)` | 右单旋转                             |
| `private BinaryNode<E> rotateLeft(BinaryNode<E> k1)`  | 左单旋转                             |

以下是二叉树结点嵌套类 BinaryNode 的架构。

| 类成员/方法                                                  | 描述                   |
| ------------------------------------------------------------ | ---------------------- |
| `public E element`                                           | 字段，本结点数据       |
| `public BinaryNode<E> left`                                  | 字段，本结点的左子结点 |
| `public BinaryNode<E> right`                                 | 字段，本结点的右子结点 |
| `public BinaryNode()`                                        | 构造器                 |
| `public BinaryNode(E element)`                               | 构造器                 |
| `public BinaryNode(E element, BinaryNode<E> left, BinaryNode<E> right)` | 构造器                 |

<br />

#### 主要方法

##### splay

展开。伸展树中最主要的操作，传入目标结点值 $e$ 和当前根结点 $t$ (中间树 $M$ 的根结点) ， **在寻找目标结点的过程中完成展开** 。首先新建树 $L$ 与树 $R$，初始为空树，方法中并不直接声明这两棵树，而是通过 $header, leftTreeMax, rightTreeMin$ 来动态维护。动态维护主要体现在 $L$ 中的 $leftTreeMax$ 以及 $R$ 中的 $rightTreeMin$ ，一开始令此二者为 $header$ ，该变量仅用于保存 $L$ 和 $R$ 初次不为空树时的根结点，用于最后 $M, L, R$ 的组装。如其名，在方法的整个过程中，此二者始终代表 $L$ 中最大结点以及 $R$ 中最小结点。

> 首次执行 `rightTreeMin.left = t` 时，即 `header.left = t` ，且此后 `header.left` 引用不变， `leftTreeMax.right = t` 同理。

```java
private BinaryNode<E> splay(E e, BinaryNode<E> t) { // 自顶向下展开目标结点
    BinaryNode<E> leftTreeMax, rightTreeMin;
    header.left = header.right = nullNode;
    leftTreeMax = rightTreeMin = header;
    nullNode.element = e;   // 确保能够匹配，便于编程
    while(true) {
        int compareRes = e.compareTo(t.element);
        if(compareRes < 0) { // 目标在t的左子树中
            if(e.compareTo(t.left.element) < 0) t = rotateRight(t); // 一字型右旋(右单旋)
            if(t.left == nullNode) break; // t无左儿子，已完成一次单旋(包括之字形)，跳出循环
            rightTreeMin.left = t; // 挂接到R下，同时保存R首次不为空时的根，且此后head.left不变
            rightTreeMin = t; // 更新rightTreeMin
            t = t.left; // 令t.left为中间树M的根结点
        }
        else if(compareRes > 0) { // 目标在t的右子树中
            if(e.compareTo(t.right.element) > 0) t = rotateLeft(t); // 一字型左旋(左单旋)
            if(t.right == nullNode) break; // t无右儿子，已完成一次单旋(包括之字形)，跳出循环
            leftTreeMax.right = t; // 挂接到L下，同时保存了L首次不为空时的根，且此后head.right不变
            leftTreeMax = t; // 更新leftTreeMax
            t = t.right; // 令t.right为中间树M的根结点
        }
        else break; // 相等即可退出
    } // 后续四句完成组装
    leftTreeMax.right = t.left;
    rightTreeMin.left = t.right;
    t.left = header.right; // header.right为L的根
    t.right = header.left; // header.left为R的根
    return t;
}
```

<br />

##### insert

插入值为 $e$ 的结点。当前树空时，将待插入的结点作为根结点，否则 **将目标结点旋转至根处** ，再根据该结点值与 $e$ 的大小调整链接关系。

```java
public void insert(E e) { // 插入结点
    if(newNode == null) newNode = new BinaryNode<>(e);
    if(root == nullNode) { // 当前为空树，将newNode作为root插入
        newNode.left = newNode.right = nullNode;
        root = newNode;
    }
    else { // 否则将目标结点旋转至根处，比较大小后插入
        root = splay(e, root);
        int compareRes = e.compareTo(root.element);
        if(compareRes < 0) { // e小于root的值，将newNode作为根插入
            newNode.left = root.left;
            newNode.right = root;
            root.left = nullNode;
            root = newNode;
        }
        else if(compareRes > 0) { // e大于root的值，将newNode作为根插入
            newNode.right = root.right;
            newNode.left = root;
            root.right = nullNode;
            root = newNode;
        }
        else return; // 相等时不插入
    }
    newNode = null; // 下一次插入时newNode == null
}
```

<br />

##### findMin/findMax

查找具有最小/最大数据的结点的操作由驱动方法和具体方法实现。与 AVL 树中对应方法的不同仅在于此处的具体方法在找到目标结点后要对其执行一次 $splay$ 操作。

```java
public E findMin() { // 查找最小结点驱动方法
    if(isEmpty()) throw new NoSuchElementException();
    return findMin(root).element;
}
private BinaryNode<E> findMin(BinaryNode<E> t) { // 返回树的最小结点
    while(t.left != nullNode) t = t.left;
    root = splay(t.element, root);
    return t;
}
public E findMax() { // 查找最大结点驱动方法
    if(isEmpty()) throw new NoSuchElementException();
    return findMax(root).element;
}
private BinaryNode<E> findMax(BinaryNode<E> t) { // 返回树的最大结点
    while(t.right != nullNode) t = t.right;
    root = splay(t.element, root);
    return t;
}
```

<br />

##### remove

调用 $contains$ 方法查找是否存在值为 $e$ 的结点，若不存在直接返回，若存在则通过 $contains$ 方法，已将该结点展开为根结点，根据此时 $root$ 有无左儿子来执行具体的删除动作。若无左儿子，可直接删除 $root$ ，即让删除后的根结点为 $root.right$ ，否则通过 `splay(e, newRoot)` 将此时根的左子树中具有最大值的根结点 (必小于 $e$) 展开至 $newRoot$ ，以其为删除后的根结点，并执行 `newRoot.right = root.right` 。此删除为 **懒惰删除** ，即只改变链接关系来达到删除的效果。

```java
public void remove(E e) { // 删除结点
    if(!contains(e)) return; // 不存在直接返回，若存在，则该目标结点已被展开至根
    BinaryNode<E> newRoot;
    if(root.left == nullNode) newRoot = root.right; // 无左儿子，直接删除(懒惰删除)
    else { // 有左儿子
        newRoot = root.left;
        newRoot = splay(e, newRoot); // 将左子树中最大者旋转至根，令其为删除操作后的新根
        newRoot.right = root.right; // 将root.right接到newTree.right上
    }
    root = newRoot;
}
```

<br />

##### contains

若为空树直接返回 $false$ ，否则以值为 $e$ 的结点为目标执行 $splay$ ，最后根据 $splay$ 的目标结点值是否等于 $e$ 来返回 $true$ 或 $false$ 。

```java
public boolean contains(E e) { // 判断树中是否有指定元素的结点
    if(isEmpty()) return false;
    root = splay(e, root);
    return root.element.compareTo(e) == 0;
}
```

<br />

##### rotateRight

右单旋转，见前述说明。

<br />

##### rotateLeft

左单旋转，见前述说明。

<br />

#### 类的实现代码

```java
class SplayTree<E extends Comparable<? super E>> {
    private BinaryNode<E> root, newNode;
    private final BinaryNode<E> nullNode;
    public SplayTree() {
        this.nullNode = new BinaryNode<>();
        this.root = this.nullNode.left = this.nullNode.right = this.nullNode;
    }
    public void insert(E e) { // 插入结点
        if(newNode == null) newNode = new BinaryNode<>(e);
        if(root == nullNode) { // 当前为空树，将newNode作为root插入
            newNode.left = newNode.right = nullNode;
            root = newNode;
        }
        else { // 否则将目标结点旋转至根处，比较大小后插入
            root = splay(e, root);
            int compareRes = e.compareTo(root.element);
            if(compareRes < 0) { // e小于root的值，将newNode作为根插入
                newNode.left = root.left;
                newNode.right = root;
                root.left = nullNode;
                root = newNode;
            }
            else if(compareRes > 0) { // e大于root的值，将newNode作为根插入
                newNode.right = root.right;
                newNode.left = root;
                root.right = nullNode;
                root = newNode;
            }
            else return; // 相等时不插入
        }
        newNode = null; // 下一次插入时newNode == null
    }
    public void remove(E e) { // 删除结点
        if(!contains(e)) return; // 不存在直接返回，若存在，则该目标结点已被展开至根
        BinaryNode<E> newRoot;
        if(root.left == nullNode) newRoot = root.right; // 无左结点，直接删除(懒惰删除)
        else { // 有左结点
            newRoot = root.left;
            newRoot = splay(e, newRoot); // 将左子树中最大者旋转至根，令其为删除操作后的新根
            newRoot.right = root.right; // 将root.right接到newTree.right上
        }
        root = newRoot;
    }
    public E findMin() { // 查找最小结点驱动方法
        if(isEmpty()) throw new NoSuchElementException();
        return findMin(root).element;
    }
    public E findMax() { // 查找最大结点驱动方法
        if(isEmpty()) throw new NoSuchElementException();
        return findMax(root).element;
    }
    public boolean contains(E e) { // 判断树中是否有指定元素的结点
        if(isEmpty()) return false;
        root = splay(e, root);
        return root.element.compareTo(e) == 0;
    }
    public void printTree(){ // 按中序遍历打印树的驱动方法
        if(isEmpty()) System.out.println("Empty tree");
        else printTree(root);
    }
    public int size() { // 求树的结点个数驱动方法
        return size(root);
    }
    public void makeEmpty() { // 树置空
        root = nullNode;
    }
    public boolean isEmpty() { // 树判空
        return root == nullNode;
    }
    private final BinaryNode<E> header = new BinaryNode<>(null);
    private BinaryNode<E> splay(E e, BinaryNode<E> t) { // 自顶向下展开目标结点
        BinaryNode<E> leftTreeMax, rightTreeMin;
        header.left = header.right = nullNode;
        leftTreeMax = rightTreeMin = header;
        nullNode.element = e;   // 确保能够匹配，便于编程
        while(true) {
            int compareRes = e.compareTo(t.element);
            if(compareRes < 0) { // 目标在t的左子树中
                if(e.compareTo(t.left.element) < 0) t = rotateRight(t); // 一字型右旋
                if(t.left == nullNode) break; // t无左儿子，已完成一次单旋(包括之字形)，跳出循环
                rightTreeMin.left = t; // 挂接到R下，同时保存R首次不为空时的根，且此后head.left不变
                rightTreeMin = t; // 更新rightTreeMin
                t = t.left; // 令t.left为中间树M的根结点
            }
            else if(compareRes > 0) { // 目标在t的右子树中
                if(e.compareTo(t.right.element) > 0) t = rotateLeft(t); // 一字型左旋
                if(t.right == nullNode) break; // t无右儿子，已完成一次单旋(包括之字形)，跳出循环
                leftTreeMax.right = t; // 挂接到L下，同时保存了L首次不为空时的根，且此后head.right不变
                leftTreeMax = t; // 更新leftTreeMax
                t = t.right; // 令t.right为中间树M的根结点
            }
            else break; // 相等即可退出
        } // 后续四句完成组装
        leftTreeMax.right = t.left;
        rightTreeMin.left = t.right;
        t.left = header.right; // header.right为L的根
        t.right = header.left; // header.left为R的根
        return t;
    }
    private BinaryNode<E> rotateRight(BinaryNode<E> k2){ // 右单旋
        BinaryNode<E> k1 = k2.left;
        k2.left = k1.right;
        k1.right = k2;
        return k1; // 返回调整后原失衡处结点
    }
    private BinaryNode<E> rotateLeft(BinaryNode<E> k1){ // 左单旋
        BinaryNode<E> k2 = k1.right;
        k1.right = k2.left;
        k2.left = k1;
        return k2; // 返回调整后原失衡处结点
    }
    private BinaryNode<E> findMin(BinaryNode<E> t) { // 返回树的最小结点
        while(t.left != nullNode) t = t.left;
        root = splay(t.element, root);
        return t;
    }
    private BinaryNode<E> findMax(BinaryNode<E> t) { // 返回树的最大结点
        while(t.right != nullNode) t = t.right;
        root = splay(t.element, root);
        return t;
    }
    private void printTree(BinaryNode<E> t) { // 中序遍历打印树
        if(t != nullNode) {
            printTree(t.left);
            System.out.println(t.element);
            printTree(t.right);
        }
    }
    private int size(BinaryNode<E> t) { // 递归地遍历所有结点，返回结点总数
        if(t != nullNode) {
            if(t.left != nullNode && t.right != nullNode) {
                return 1 + size(t.left) + size(t.right);
            }
            else {
                return t.left != nullNode ? 1 + size(t.left) : 1 + size(t.right);
            }
        }
        return 0;
    }
    /**
     * 二叉树结点嵌套类
     */
    private static class BinaryNode<E>{
        public E element;
        public BinaryNode<E> left, right;
        @SuppressWarnings("unused")
        public BinaryNode(){
            this(null, null, null);
        }
        public BinaryNode(E element){
            this(element, null, null);
        }
        public BinaryNode(E element, BinaryNode<E> left, BinaryNode<E> right){
            this.element = element;
            this.left = left;
            this.right = right;
        }
    }
}
```

<br />

#### 测试代码

```java
package com.yukiyama;

import java.util.NoSuchElementException;

public class SplayTreeDemo {

    public static void main(String[] args) {
        SplayTree<Integer> t = new SplayTree<>( );
        int[] elements = {3,2,1,4,5,6,7,16,15,14,13,12,11,10,8,9};
        for(int e : elements) t.insert(e);
        t.printTree();
        System.out.printf("size: %d, min: %d, max: %d\n", t.size(), t.findMin(), t.findMax()); // 输出10
        System.out.printf("has 10? %s, has 100? %s\n", t.contains(10), t.contains(100));
        t.remove(1);
        t.remove(10);
        t.remove(16);
        t.printTree();
        System.out.printf("size: %d\n", t.size());// 输出0
        System.out.println("is empty: " + t.isEmpty()); // false
        t.makeEmpty();
        System.out.println("is empty: " + t.isEmpty()); // true
    }

}
```

<br />