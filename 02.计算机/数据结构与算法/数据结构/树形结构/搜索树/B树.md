# B树

B 树其实就是多路的平衡二叉搜索树，下面我们来看看究竟什么是 B 树。

## B 树的概念

### 1 节点属性：

1. 当前节点存储的关键字个数
2. 非降序存储的关键字
3. 是否为叶节点的标识
4. n+1 个孩子

```javascript 
class Node 
{
    #n;         // 关键字个数
    #keys;      // 关键字
    #isLeaf;    // 是否为叶节点
    #children;  // n+1 个孩子
}
```

### 2 节点规则

- 节点的关键字允许的范围是根据父节点的关键字所划定的。
- 每个叶节点的深度相同。 
- 每个节点包含的关键字有上界和下界，上界和下界根据 B 树的最小度数 (t >= 2) 确定。
- 非空 B 树至少有一个关键字。
- 除根节点，其他节点至少含有 t-1 个关键字，至少有 t 个孩子。
- 每个节点至多包含 (2t - 1) 个关键字，至多有 2t 个孩子。

### 3 B 树的性质 

1. 高度为 h， 含有 n >= 1 个关键字, 高度为 h 的 b 树有  h <= logt[(n+1) / 2]。 


### B 树的操作

#### 1 搜索

搜索的操作大致可以概括为：

1. 定位到节点的某个关键字区间内。
2. 若查找到关键字，则返回；未查找到且是叶节点，返回未查找到；未查找到不是叶节点，递归查找子树。

```javascript
function search(node,  key) 
{
    let i = 1;
    while(i < node.n && key > node.key) 
    {
        i++;
    }

    if(i <= node.n && key === node.key)
    {
        return {node, i};
    }
    else if(node.isLeaf)
    {
        return null;
    }
    else 
    {
        return search(node.children[i], key);
    }
} 
```


#### 2 创建

B 树上创建的事件复杂度为 O(1)。

```javascript 
function create(tree)
{
    const node = new Node();
    node.isLeaf = true;
    node.n = 0;
    diskWrite(node);
    tree.root = node;
}
```

#### 3 插入

为了在插入过程中满足 B 树的性质，我们必须引入 B 树的分裂操作。

##### 3.1 分裂 

分裂，即为将一个满的节点（含有 2t-1 个关键字）分裂成为两个各含 t-1 个关键字的节点，中间关键字被提升为父节点的关键字，若父节点也满，也必须递归进行分裂。

我们可以在查找新的关键字时，就分裂沿途遇到的每个满节点，为节点的插入做准备。

分裂操作的过程可以概括如下 

1. 新建节点。
2. 将被分裂节点的后 t-1 个关键字和后 t-1 个孩子赋值给新节点。
3. 将新节点插入到父节点，将被分裂节点的中间关键字提升到父节点。

```javascript 
function split(node, i)
{
    const nodeA = new Node();
    const nodeB = node.children[i];
    nodeA.isLeaf = nodeB.isLeaf;
    nodeB.n = t - 1;
    nodeA.keys = nodeB.keys.slice(t, 2t);
    !nodeA.isLeaf && (nodeA.children = nodeB.children.slice(t, 2t);
    nodeB.n = t - 1;
    node.children = node.children.slice(0, i).concat([nodeA]).concat(node.children.slice(i, node.n));
    node.keys = node.keys.slice(0, i).concat([nodeB.key[t]]).concat(node.keys.slice(i, node.n));
    node.n++;
}
```

##### 3.2 在未满的节点插入

```javascript
function insertNotFull(node, key)
{   
    let i = node.n;
    while(i >= 1 && key < node.key[i])
    {
        i--; 
    }
    if(node.isLeaf)
    {   
        const {keys} = node;
        node.keys = keys.slice(0, i-1).concat([key]).concat(keys.slice(i-1, node.n)); 
        node.n++;
    }
    else 
    {
        i++;
        if(node.children[i].n === (2*t - 1))
        {
            split(node, i);
            if(key > node.keys[i])
            {
                i++
            }
        }
        insertNotFull(node.children[i], key);   
    }
    
} 
```

##### 3.2 插入 

```javascript
function insert(tree, key)
{
    const {root} = tree;
    if( root.n === (2*t-1)) 
    {
        const node = new Node();
        tree.root = node;
        node.isLeaf = false;
        node.n = 0;
        node.children = [root];
        split(node, key);
        insertNotFull(node, key);
    }
    else 
    {
        insertNotFull(node, key);
    }
}
```
