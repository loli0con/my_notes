# 二叉树
⌊⌋
## 树转二叉树
儿子在⬅️，兄弟在➡。
![tree+20210806152052](https://i.loli.net/2021/08/06/8so1H7mILxiwd92.png)

## 二叉树的性质
1. 在二叉树的第i层上至多有2<sup>i-1</sup>个结点(i≥1)，至少有1个结点。
2. 深度为k的二叉树至多有2<sup>k-1</sup>个结点(k≥1)，至少有k个结点。
![tree+20210806163031](https://i.loli.net/2021/08/06/7ImhyGVUnpZ5taP.png)
3. 对任何一棵二叉树T，如果其叶子数为n<sub>0</sub>，度为2的结点数为n<sub>2</sub>，则有n<sub>0</sub>=n<sub>2</sub>+1。
![tree+20210806163844](https://i.loli.net/2021/08/06/N9lZhFVHAfXMnjw.png)
4. 具有n个结点的**完全二叉树**的深度为⌊log<sub>2</sub>n⌋+1。
   * ⌊x⌋：称作x的底，表示不大于x的最大整数。（向下取整）
![tree+截屏2021-08-06 17.48.48](https://i.loli.net/2021/08/06/bEpcvTfsOeRVIG1.png)
![tree+截屏2021-08-06 18.08.46](https://i.loli.net/2021/08/06/QJDa3g9Me5luoIf.png)
5. 如果对一棵有n个结点的**完全二叉树**的结点按层序编号，则对**任一结点i**(1≤i≤n)，有：
   1. 如果i=1，则结点i是二叉树的根，无双亲；如果i>1，则其**双亲是⌊i/2⌋**。
   2. 如果2i>n，则结点i为叶子结点，无左孩子；否则，其**左孩子是结点2i**。
   3. 如果2i+1>n，则结点i无右孩子；否则，其**右孩子是结点2i+1**。

## 二叉树的遍历
![binary-tree+20210812113655](https://i.loli.net/2021/08/12/uQfTCHhz5ZeA8Rp.png)

### 顺序

若规定**先左后右**，则有三种情况：
* 先序遍历：**根**-左-右
* 中序遍历：左-**根**-右
* 后序遍历：左-右-**根**

```
先序序列：A B D G C E H F
中序序列：D G B A E H C F
后序序列：G D B H E F C A
```

### 根据遍历序列确定二叉树
中序序列 + ( 先序序列 | 后序序列 ) => 二叉树

#### 分析
先序序列 | 后序序列 -> 可以确定*根的位置*  
中序序列 -> 利用*根的位置*，可以确定*左子树*和*右子树*

### 遍历算法 - 递归
```c++
// 二叉树先序遍历算法
Status PreOrderTraverse(BiTree T){
    if(T==NULL) return OK;//空二叉树
    else{
        visit(T);//访问根结点
        PreOrderTraverse(T->lchild);//递归遍历左子树
        PreOrderTraverse(T->rchild);//递归遍历右子树
    }
}

// 二叉树中序遍历算法
Status InOrderTraverse(BiTree T){
    if(T==NULL) return OK;//空二叉树
    else{
        InOrderTraverse(T->lchild);//递归遍历左子树
        visit(T);//访问根结点
        InOrderTraverse(T->rchild);//递归遍历右子树
    }
}

// 二叉树后序遍历算法
Status PostOrderTraverse(BiTree T){
    if(T==NULL) return OK;//空二叉树
    else{
        PostOrderTraverse(T->lchild);//递归遍历左子树
        PostOrderTraverse(T->rchild);//递归遍历右子树
        visit(T);//访问根结点
    }
}
```

#### 分析
![binary-tree+20210812121755](https://i.loli.net/2021/08/12/j9dJieAn7sWRXCa.png)

* 时间效率：O(n) - 每个结点只访问一次
* 空间效率：O(n) - 栈占用的最大辅助空间（最坏情况：单枝树）


### 遍历算法 - 非递归
```c++
// 二叉树中序遍历算法
Status PreOrderTraverse(BiTree T){
    BiTree p; InitStack(S); p = T;
    while(p || !StackEmpty(S)){
        if(p) {Push(S,p); visit(p); p = p -> lchild;}
        else {Pop(S,p); p = p -> rchild;}
    }
    return OK;
}

// 二叉树中序遍历算法
Status InOrderTraverse(BiTree T){
    BiTree p; InitStack(S); p = T;
    while(p || !StackEmpty(S)){
        if(p) {Push(S,p); p = p -> lchild;}
        else {Pop(S,p); visit(p); p = p -> rchild;}
    }
    return OK;
}

// 二叉树层序遍历算法
void LevelOrder(BTNode *b){
    BTNode *p; SqQueue *qu;
    InitQueue(qu);//初始化队列
    enQueue(qu,b);//根结点入队
    while(!QueueEmpty(qu)){//队列不为空，则循环
        deQueue(qu,p);//出队结点p
        visit(p);//访问p
        if(p->lchild!=NULL) enQueue(qu,p->lchild);//p有左孩子，则左孩子入队
        if(p->rchild!=NULL) enQueue(qu,p->rchild);//p有右孩子，则右孩子入队
    }
}
```

```Java
// 二叉树后序遍历算法
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    List<Integer> ans = new ArrayList<>();
    public List<Integer> postorderTraversal(TreeNode root) {
        Stack<TreeNode> s = new Stack<>();
        TreeNode cur = root;  
        TreeNode pre = null;  // 用于记录上一次访问的节点
        while(cur!=null || !s.isEmpty()) {
            while(cur!=null) {
                s.push(cur);
                cur = cur.left;
            }
            if(!s.isEmpty()) {
                cur = s.pop();
                if(cur.right==null || pre==cur.right) { // 访问节点的条件
                    ans.add(cur.val); // 访问
                    pre = cur; // 这一步是记录上一次访问的节点
                    cur = null; // 此处为了跳过下一次循环的访问左子节点的过程，直接进入栈的弹出阶段，因为但凡在栈中的节点，它们的左子节点都肯定被经过且已放入栈中。
                }
                else { // 不访问节点的条件
                    s.push(cur); // 将已弹出的根节点放回栈中
                    cur = cur.right; // 经过右子节点
                }
            }
        }
        return ans;
    }
}
```

### 遍历算法的应用

### 建立二叉树 - 先序 
![binary-tree+20210812150428](https://i.loli.net/2021/08/12/lqvTLzjKxYesWco.png)

### 复制二叉树 - 先序
![binary-tree+20210812150849](https://i.loli.net/2021/08/12/tkLxVANPX8BSqcY.png)

### 计算二叉树深度 - 后序
![binary-tree+20210812151235](https://i.loli.net/2021/08/12/eu94AJOylY1RiDV.png)

### 计算二叉树结点总数 - 后序
![binary-tree+20210812151819](https://i.loli.net/2021/08/12/UR6yAK1Hg5NCjub.png)

### 计算二叉树叶子总数 - 后序
![binary-tree+20210812151922](https://i.loli.net/2021/08/12/sUVgb8wv9j3O6H7.png)

## 二叉线索树
利用二叉链表中的空的指针域：  
如果某个结点的左孩子为空，则将左孩子指针域改为**指向其前驱**；如果某结点的右孩子为空，则将空的右孩子指针域改为**指向其后继**。  
————这种**改变指向的指针**成为“**线索**”，加上了线索的二叉树称为**线索二叉树**，对二叉树按某种遍历次序，使其变为线索二叉树的过程叫做**线索化**。

## 二叉搜索树
二叉搜索树：一棵二叉树，可以为空；如果不为空，满足以下性质：
1. 非空左子树的所有键值小于其根结点的键值
2. 非空右子树的所有键值大于其根结点的键值
3. 左、右子树都是二叉搜索树

### 重要函数
Find、FindMin、FindMax、Insert、Delete

#### Find
```c++
Position Find(ElementType X, BinTree BST)
{
    if(!BST) return NULL;
    if(X > BST->Data)
        return Find(X, BST->Right);
    else if(X < BST->Data)
        return Find(X, BST->Left);
    else
        return BST;
}

// 尾递归很容易就能用迭代来实现

Position IterFind(ElementType X, BinTree BST)
{
    while(BST){
        if(X > BST->Data)
            BST = BST->Right;
        else if(X < BST->Data)
            BST = BST->Left;
        else
            return BST;
    }
    return NULL;
}

查找的效率决定于树的高度
```

#### FindMin & FindMax
* 最大元素一定是在树的最右分支的端结点上
* 最小元素一定是在树的最左分支的端结点上

```c++
//查找最小元素的递归函数
Position FindMin(BinTree BST)
{
    if(!BST) return NULL;
    else if(!BST->Left)
        return BST;
    else
        return FindMin(BST->Left);
}

//查找最大元素的迭代函数
Position FindMax(BinTree BST)
{
    if(BST)
        while(BST->Right) BST=BST->Right;
    return BST;
}
```

#### Insert
关键是找到元素应该插入的位置，可以采用与Find类似的方法。

```c++
BinTree Insert(ElementType X, BinTree BST)
{
    if( !BST ){
        BST = malloc(sizeof(struct TreeNode));
        BST->Data = X;
        BST->Left = BST->Right = NULL;
    }else
        if( X < BST->Data )
            BST->Left = Insert(X, BST->Left);
        else if( X > BST->Data )
            BST->Right = Insert(X, BST->Right);

    return BST;
}
```

#### 删除
考虑三种情况：
* 要删除的是**叶结点**：直接删除，并再修改其父结点指针——置为NULL
* 要删除的结点**只有一个孩子**结点：将其父结点的指针指向要删除结点的孩子结点
* 要删除的结点**有左、右两棵子树**：用另一结点代替被删除结点：**右子树的最小元素**或者**左子树的最大元素**。这两个元素共有的特性“一定不是有两个儿子的结点”，这样做的好处是“使复杂的问题简单化——把有两个儿子的情况变成要么只有一个儿子/要么没有儿子的情况，即上述的两种情况”。

```c++
BinTree Delete(ElementType X, BinTree BST)
{
    Positition Tmp;
    if(!BST) printf("要删除的元素未找到");
    else if (X < BST->Data)
        BST->Left = Delete(X, BST->Left); // 左子树递归删除
    else if (X > BST->Data)
        BST->Right = Delete(X, BST->Right); // 右子树递归删除
    else
        if(BST->Left && BST->Right){ // 被删除的结点有左右两个子结点
            Tmp = FindMin(BST->Right); // 在右子树中找最小的元素填充删除结点
            BST->Data = Tmp->Data;
            BST->Right = Delete(BST->Data, BST->Right); // 在删除结点的右子树中删除最小元素
        } else { // 被删除结点只有一个或无子结点
            Tmp = BST;
            if(!BST->Left) // 有右孩子或无子结点
                BST = BST->Right;
            else if(!BST->Right) // 有左孩子
                BST = BST->Left;
            free(Tmp);
        }
}
```

## 平衡二叉树(AVL🌲)
平衡因子(Balance Factor，简称BF)：BF(T) = h<sub>L</sub>-h<sub>R</sub>，  
其中h<sub>L</sub>和h<sub>R</sub>分别为左、右子树的高度。

平衡二叉树(Balanced Binary Tree)：空树，或者任一结点左、右子树高度差的绝对值不超过1，即|BF(T)|≤1。

### 维基百科
平衡二叉搜索树（英语：Balanced Binary Search Tree）是一种结构平衡的二叉搜索树，它是一种每个节点的左右两子树高度差都不超过一的二叉树。它能在O(log n)内完成插入、查找和删除操作，最早被发明的平衡二叉搜索树为AVL树。

常见的平衡二叉搜索树有：
* AVL树
* 红黑树
* Treap
* 节点大小平衡树

### 平衡二叉树的高度（log<sub>2</sub>n）
树的高度 即 查找的效率，此处探究：  
**结点数 - 树高度 - 查找效率** 的关系

结点数取最大值，即满二叉树，结点数和树高度的关系为h=O(log<sub>2</sub>n)。

下面讨论的是结点数取最小值的情况。

设n<sub>h</sub>是高度为h的平衡二叉树的最少结点数，有如下推论：
![binary-tree+20210908121054](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908121054.png%2B2021-09-08-12-10-55)

![binary-tree+20210908121113](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908121113.png%2B2021-09-08-12-11-14)

![binary-tree+20210908121139](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908121139.png%2B2021-09-08-12-11-40)

![binary-tree+20210908130154](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908130154.png%2B2021-09-08-13-01-56)

![binary-tree+20210908130938](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908130938.png%2B2021-09-08-13-09-40)

因此可得：
由n个结点构成的平衡二叉树的查找效率为O(log<sub>2</sub>n)。

### 平衡二叉树的调整
平衡二叉树插入/删除元素之后，有可能需要计算平衡因子；当平衡因子的绝对值>1时（即不满足平衡二叉树的定义），需要对树进行调整（使其满足平衡二叉树的定义）。
#### RR
不平衡的“**发现者**”是A，“**麻烦结点**”B<sub>R</sub>在发现者**右**子树的**右**边，因而叫**RR插入**，需要**RR旋转（右单旋）**。

![binary-tree+20210908153559](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908153559.png%2B2021-09-08-15-36-00)

#### LL
不平衡的“**发现者**”是A，“**麻烦结点**”B<sub>L</sub>在发现者**左**子树的**左**边，因而叫**LL插入**，需要**LL旋转（左单旋）**。

![binary-tree+20210908153847](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908153847.png%2B2021-09-08-15-38-48)

#### LR
“**发现者**”是A，“**麻烦结点**”C<sub>L</sub>/C<sub>R</sub>在发现者**左**子树的**右**边，因而叫**LR插入**，需要**LR旋转**。

![binary-tree+20210908154022](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908154022.png%2B2021-09-08-15-40-24)

#### RL
“**发现者**”是A，“**麻烦结点**”C<sub>L</sub>/C<sub>R</sub>在发现者**右**子树的**左**边，因而叫**RL插入**，需要**RL旋转**。

![binary-tree+20210908154536](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210908154536.png%2B2021-09-08-15-45-38)


## 堆(heap)
### 优先队列
优先队列(Priority Queue)：特殊的“队列”，取出元素的顺序是依照元素的优先权(关键字)大小，而不是元素进入队列的先后顺序。

在队列中，调度程序反复提取队列中第一个作业并运行，因为实际情况中某些时间较短的任务将等待很长时间才能结束，或者某些不短小，但具有重要性的作业，同样应当具有优先权。堆即为解决此类问题设计的一种数据结构。

组织优先队列：
1. 一般的数组、链表
2. 有序的数组或者链表
3. 二叉搜索树/AVL树

#### 数组/链表
* 数组：
  * 插入：元素总是插入尾部 ～ O(1)
  * 删除：
    1. 查找最大(或最小)关键字 ～ O(n)
    2. 从数组中删去需要移动元素 ～ O(n)
* 链表：
  * 插入：元素总是插入链表的头部 ～ O(1)
  * 删除：
    1. 查找最大(或最小)关键字 ～ O(n)
    2. 删去结点 ～ O(1)
* 有序数组：
  * 插入：
    1. 找到合适的位置 ～ O(log<sub>2</sub>n)
    2. 移动元素并插入 ～ O(n)
  * 删除：删去最后一个元素 ～ O(1)
* 有序链表：
  * 插入
    1. 找到合适的位置 ～ O(n)
    2. 插入元素 ～ O(1)
  * 删除：删除首元素或最后元素 ～ O(1)

### 维基百科
堆（英语：Heap）是计算机科学中的一种特别的完全二叉树。若是满足以下特性，即可称为堆：“给定堆中任意节点P和C，若P是C的母节点，那么P的值会小于等于（或大于等于）C的值”。若母节点的值恒小于等于子节点的值，此堆称为最小堆（min heap）；反之，若母节点的值恒大于等于子节点的值，此堆称为最大堆（max heap）。在堆中最顶端的那一个节点，称作根节点（root node），根节点本身没有母节点（parent node）。

### 堆的特性
* 结构性:用数组表示的完全二叉树
* 有序性:任一结点的关键字是其子树所有结点的最大值(或最小值)
  * 最大堆(MaxHeap)”,也称“大顶堆”:最大值
  * “最小堆(MinHeap)”,也称“小顶堆” :最小值

注意:从根结点到任意结点路径上结点序列的有序性!

### 堆的抽象数据类型描述
* 类型名称:最大堆(MaxHeap)
* 数据对象集:完全二叉树，每个结点的元素值不小于其子结点的元素值
* 操作集:最大堆H ∈ MaxHeap，元素item ∈ ElementType，主要操作有:
  * MaxHeap Create( int MaxSize ):创建一个空的最大堆。
  * Boolean IsFull( MaxHeap H ):判断最大堆H是否已满。
  * Insert( MaxHeap H, ElementType item ):将元素item插入最大堆H。
  * Boolean IsEmpty( MaxHeap H ):判断最大堆H是否为空。
  * ElementType DeleteMax( MaxHeap H ):返回H中最大元素(高优先级)。

```c++
typedef struct HeapStruct *MaxHeap;
struct HeapStruct {
    ElementType *Elements; /* 存储堆元素的数组 */
    int Size; /* 堆的当前元素个数 */
    int Capacity; /* 堆的最大容量 */
};

MaxHeap Create( int MaxSize )
{
    /* 创建容量为MaxSize的空的最大堆 */
    MaxHeap H = malloc( sizeof( struct HeapStruct ) );
    H->Elements = malloc( (MaxSize+1) * sizeof(ElementType));
    H->Size = 0;
    H->Capacity = MaxSize;
    H->Elements[0] = MaxData;
    /* 定义“哨兵”为大于堆中所有可能元素的值，便于以后更快操作 */
    return H;
}

// T (N) = O ( log N )
void Insert( MaxHeap H, ElementType item )
{ 
    /* 将元素item 插入最大堆H，其中H->Elements[0]已经定义为哨兵 */
    int i;
    if ( IsFull(H) )
    {
        printf("最大堆已满");
        return;
    }
    i = ++H->Size; /* i指向插入后堆中的最后一个元素的位置 */
    for ( ; H->Elements[i/2] < item; i/=2 )
        H->Elements[i] = H->Elements[i/2]; /* 向下过滤结点 */ 
    H->Elements[i] = item; /* 将item 插入 */
}

// T (N) = O ( log N )
ElementType DeleteMax( MaxHeap H )
{ 
    /* 从最大堆H中取出键值为最大的元素，并删除一个结点 */ 
    int Parent, Child;
    ElementType MaxItem, temp;
    if ( IsEmpty(H) ) 
    {
        printf("最大堆已为空");
        return; 
    }
    MaxItem = H->Elements[1]; /* 取出根结点最大值 */
    /* 用最大堆中最后一个元素从根结点开始向上过滤下层结点 */ 
    temp = H->Elements[H->Size--];
    for( Parent=1; Parent*2<=H->Size; Parent=Child ) 
    {
        Child = Parent * 2;
        if( (Child!= H->Size) && (H->Elements[Child] < H->Elements[Child+1]) )
            Child++; /* Child指向左右子结点的较大者 */

        if( temp >= H->Elements[Child] )
            break;
        else /* 移动temp元素到下一层 */
            H->Elements[Parent] = H->Elements[Child];
    }
    H->Elements[Parent] = temp;
    return MaxItem;
}
```

### 建堆时间复杂度O(n)
假设目标堆是一个满堆，即第 k 层节点数为 2ᵏ。输入数组规模为 n, 堆的高度为 h, 那么 n 与 h 之间满足 n=2ʰ⁺¹ - 1，可化为 h=log₂(n+1) - 1。 (层数 k 和高度 h 均从 0 开始，即只有根节点的堆高度为0，空堆高度为 -1)。

备注：n与h的关系，可以根据[二叉树的性质](#二叉树的性质)中的第二点得出。

建堆过程中每个节点需要一次下滤操作，交换的次数等于该节点到叶节点的深度。那么每一层中所有节点的交换次数为节点个数乘以叶节点到该节点的深度（如第一层的交换次数为 2⁰ · h，第二层的交换次数为 2¹ · (h-1)，如此类推）。从堆顶到最后一层的交换次数 Sn 进行求和：  
Sn = 2⁰ · h + 2¹ · (h - 1) + 2² · (h - 2) + ...... + 2ʰ⁻² · 2 + 2ʰ⁻¹ · 1 + 2ʰ · 0

把首尾两个元素简化，记为①式：  
①: Sn = h + 2¹ · (h - 1) + 2² · (h - 2) + ...... + 2ʰ⁻² · 2 + 2ʰ⁻¹

对①等于号左右两边乘以2，记为②式：  
②: 2Sn = 2¹ · h + 2² · (h - 1) + 2³ · (h - 2) + ...... + 2ʰ⁻¹ · 2 + 2ʰ

那么用②式减去①式，其中②式的操作数右移一位使指数相同的部分对齐（即错位相减法）：  
![binary-tree+20210909150920](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210909150920.png%2B2021-09-09-15-09-21)

化简可得③式：  
③ = Sn = -h + 2¹ + 2² + 2³ + ...... + 2ʰ⁻¹ + 2ʰ

对指数部分使用等比数列求和公式：
![binary-tree+20210909151043](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210909151043.png%2B2021-09-09-15-10-44)

化简为④式：  
④ = Sn = 2ʰ⁺¹ - (h + 2)

在前置条件中已得到堆的节点数 n 与高度 h 满足条件 n=2ʰ⁺¹ - 1（即 2ʰ⁺¹=n+1） 和 h=log₂(n+1) - 1，分别代入④式中的 2ʰ⁺¹ 和 h，因此：
![binary-tree+20210909151113](https://raw.githubusercontent.com/loli0con/picgo/master/images/binary-tree%2B20210909151113.png%2B2021-09-09-15-11-13)

化简后为：  
Sn = n - log₂(n + 1)

因此最终可得渐进复杂度为 O(n).