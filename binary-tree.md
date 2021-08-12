# 二叉树
⌊⌋
## 转二叉树
儿子在⬅️，兄弟在➡。
![tree+20210806152052](https://i.loli.net/2021/08/06/8so1H7mILxiwd92.png)

## 性质
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

## 遍历
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

### 算法 - 递归
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