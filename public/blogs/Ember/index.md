# 双向链表及其应用

## 简介

​	双向链表记录了两个方向的指针。双向链表的节点定义同时包含指向**后继节点**（下一节点）和**前驱节点**（上一节点）的指针。相较于单向链表，双向链表更具**灵活性**，可以朝两个方向遍历链表，但相应地也需要占用更多的内存空间。

> **灵活性**

双向链表的灵活性体现在三个方面：

- **插入**：可以直接插入节点，单向链表在插入时，先遍历找到指定节点，还需要记录前驱节点。然而在双向链表情况下，遍历找到指定节点，通过双向链表的性质就可以直接进行插入，不需要像单链表一样去记录前驱节点的值。
- **删除**：允许直接删除指定节点。不需要在遍历时记录前驱节点的值。
- **移动节点**：删除链表中的指定节点(不释放节点)+插入删除节点到指定位置。这是实现**LRU**、**LFU**算法的关键。

> **双向链表类型**

双向链表的实现方式与单链表一样有两种：

- **普通双向链表**：这样的实现需要两个虚拟(dummy)节点，`tail`节点用于快速访问尾节点进行尾插与尾删。
- **环形双向链表**：环形双向链表只使用了一个虚拟节点，通过`head`节点的`prev`与`next`指针同样可以快速的操作链表。

**NOTE：**在观看下图(包括后面的图)是需要注意的是，如绿色的`next`所示箭头都是从**内部**出发指向了下一个节点的边框，代表这个`next`指针保存的是下个节点的地址。

![dobly_linklist](https://ember-img.oss-cn-chengdu.aliyuncs.com/dobly_linklist.png)

相比于双向链表的实现方式，环形链表方式更加适用于LRU算法(在xv6中的LRU算法是使用环形双向链表)，一方面是可以少存储虚拟节点，不会造成浪费，另一方面是也可以像双向链表一样在$O(1)$时间复杂度实现头插和头删。注意：本文的双向链表的实现方式就是环形的。

## 环形双向链表定义

> **节点定义：**

在Node结构体中定义了

- **数据域**：m_data，当然可以使用template将Node数据成员泛型化。
- **指针域**：指针域中`prev`记录前驱节点的地址值，next记录后继节点的地址值。
- **构造函数**

```cpp
struct Node{
    int m_data;
    Node* next{nullptr};
    Node* prev{nullptr};
    Node() = default;
    Node(int data) : m_data(data){}
};
```



> **环形链表类:**

在环形链表类中定义了函数方法与成员

- 数据成员：`size`记录链表节点个数、`head`记录双向链表的头节点是一个虚拟节点
- 函数成员
  - **插入**：`insert`指定位置插入、`insertHead`头插、`insertTail`尾插。
  - **删除**：`remove`指定位置删除、`removeHead`头删、`removeTail`尾删。
  - **遍历**：`traversal`遍历整个链表节点。
  - **销毁链表**：`destroyList`在析构函数中调用，目的是清空链表内存。

```cpp
//环形链表
class ListNode{
public:
	ListNode();
    ~ListNode();

	//插入
	void insert(int, int);
    void insertHead(int);
    void insertTail(int);
	//删除
	void remove(int);
    void removeHead();
    void removeTail();
    
    void traversal();
private:
    Node<int>  *getNode(int);
    void destroyList();//释放节点内存
	int size;
    Node *head;//dummy node
};
```



下面是比较基本的两个函数

- **构造函数**：在构造函数中对虚拟节点进行初始化，将`head`的`prev`与`next`指针指向本身，目的是将所有的插入操作包装成一个函数，方便操作(详情可以参见插入操作部分)。

- **获取索引节点**：通过输入的索引值，遍历链表获取这个索引下的节点地址值。

```cpp
//初始化虚拟节点
ListNode::ListNode(){
    head = new Node();
    head->prev = head->next = head; //对于环形双向链表，这步是必要的
    this->size = 0;
}

//获取指定索引的节点地址值
Node *ListNode::getNode(int idx){
    Node* temp = head->next;
    while(idx--){
        temp = temp->next;
    }
    return temp;
}
```



## 插入

- **头插：**跟单链表一样，双链表的头插复杂度为$O(1)$，但是相比单链表是要复杂一点的，因为毕竟需要设置两个指针。具体的步骤如下：注意下述head是一个虚拟节点，在此并不将认作是链表中的节点。

1.将`head`的后驱节点`head->next`(双链表的第一节点，这里是head本身)节点的`prev`指针指向**新插入节点1**(新的第一节点)。

> 在此可以解释，为什么在初始化虚拟节点head时需要将`head`的`prev`与`next`指针指向`head`本身? 如果将**插入操作**包装成函数，在最开始插入时`head->next`就是`null`值肯定就有问题了。

2.将节点1的`next`指针指向`head->next`(被节点1替代的旧的第一节点，这里仍然是head本身)。

![dlist_insertHead1](https://ember-img.oss-cn-chengdu.aliyuncs.com/dlist_insertHead1.png)

3.将节点1的`prev`指针指向`head`，说明节点1的前驱节点是head这个虚拟节点了。

4.最后将`head`的`next`指针指向节点1，说明head的后驱节点就是新插入的节点1了。到这也就成功将节点插入链表头部了。

![dlist_insertHead2](https://ember-img.oss-cn-chengdu.aliyuncs.com/dlist_insertHead2.png)

由于上述演示没有节点存在，情况比较特殊，但是后续的头插都与上述相同，如有疑惑建议自行画图增强一下查过过程的理解。后续介绍的尾插操作会在上述的链表的基础上继续进行操作。

> 代码：头插

```cpp
void ListNode::insertHead(int data){
    Node *node = new Node(data);
    //修改指针指向
    head->next->prev = node;
    node->next = head->next;
    node->prev = head;
    head->next = node;
    this->size++;
}
```



- **尾插：**

在链表已经存在一个节点的情况下进行尾插操作：

1.将链表尾节点的`next`指针指向待插入节点node2(新的尾节点)

2.将node2的`prev`指针指向旧的尾节点node1

![dlist_insertTail1](https://ember-img.oss-cn-chengdu.aliyuncs.com/dlist_insertTail1.png)

3.将node2的`next`指针指向`head`。

4.将head的`prev`指针指向node2，那么node2就作为了`head`的前驱节点。

![dlist_insertTail2](https://ember-img.oss-cn-chengdu.aliyuncs.com/dlist_insertTail2.png)

```cpp
void ListNod::insertTail(int data){
    Node *node = new Node(data);
    //修改指针指向
    head->prev->next = node;
    node->prev = head->prev;
    node->next = head;
    head->prev = node;
    //增加size
    this->size++;
}
```



- **指定位置插入：**在头插的基础上没有太多变化，这里需要注意的是我们使用`getNode(idx-1)`获取插入索引位置的前驱节点(可以将这个temp节点假设为head)，那么操作就与头插无异了。

```cpp
void ListNode::insert(int data,int idx){
    if(idx < size){//O(n)
        Node *node = new Node(data);
        Node* temp = getNode(idx-1);//找到插入位置的前驱节点
        //插入操作
        temp->next->prev = node;
        node->next = temp->next;
        node->prev = temp;
        temp->next = node; 
        this->size++;
    }else{//tail insert O(1)
		insertTail(data);
    }
}
```





## 删除

删除节点的操作相对于插入节点来说是简单了很多的，因此我们直接从指定位置查看这个过程。

- **指定位置删除**：如下图所示，首先使用getNode(idx)获取指定索引的节点node2，那么待删除节点为node2，我们所需要执行的操作就是将待删除节点的前驱节点以及后继节点连接起来就可以了。最后再释放这个待删除节点即可。

![dlist_remove](https://ember-img.oss-cn-chengdu.aliyuncs.com/dlist_remove.png)

```cpp
void ListNode::remove(int idx){
    if(size == 0){
        cout<<"empty list"<<endl;
        return;
    }
    if(idx < size){
        Node* p = getNode(idx);//找到要删除的节点
        p->next->prev = p->prev;
        p->prev->next = p->next;
       	this->size--;
        delete p;
    }else{
        removeTail();
    }   
}
```



- **头删、尾删**：这两个操作是针对与指定位置插入的特殊化，难度不大直接看代码实现就可以了。时间复杂度都为$O(1)$。

```cpp
void ListNode::removeHead(){
    Node* p = head->next;
    p->next->prev = head;
    head->next = p->next;
    delete p; 
    this->size--;  
}
void ListNode::removeTail(){
    Node* p = head->prev;
    p->prev->next = head;
    head->prev = p->prev;
    delete p;
    this->size--;  
}
```



## 遍历

- 方法一：正向遍历，使用size字段控制循环条件，从第一个实体节点(`head->next`)开始遍历到尾节点。

```cpp
void ListNode::traversal(){
    Node* p = head->next;
    int i = 0;//这个双链表是环式结构
    while(i < this->size){
        cout<< "Node["<<i<<"]"<< "addrr = "<<p << " val = "<<p->m_data<<endl;
        p = p->next;
        i++;
    }
}
```



- 方法二:反向遍历，这里使用`head`值作为控制循环条件，从最后一个实体节点(尾节点，`head->prev`)遍历到头节点。

```cpp
void ListNode::traversal(){
    Node* p = head->prev;
    int i = 0;//这个双链表是环式结构
    while(p != head){
        cout<< "Node["<<i<<"]"<< "addrr = "<<p << " val = "<<p->m_data<<endl;
        p = p->prev;
        i++;
    }
}
```



## 销毁

销毁双向链表的方法就是一直采用**头删**或是**尾删**既可以将所有节点释放掉了

> 实现

```cpp
//释放所有节点内存
ListNode::~ListNode(){
    destroyList();
}

void ListNode::destroyList(){
    Node* p = head->next;
    cout<<"destory list"<<endl;
    while(this->size){
        removeHead();//通过头删删除所有节点
    }
    delete head;
}
```



## 总结

> **特点**

1. **双向遍历**：由于每个节点既有指向前驱节点的指针，又有指向后一个节点的指针，因此可以从任一方向遍历链表，使得操作更加灵活高效。
2. **快速插入和删除**：相比数组，在双向链表中插入和删除操作更加高效。只需修改相邻节点的指针即可，不需要像数组那样进行元素的移动。
3. **删除节点的能力**：双向链表允许直接删除指定节点，而单向链表只能通过遍历找到前驱节点，然后才能删除当前节点。
4. **空间占用**：相对于数组，双向链表需要更多的内存来存储额外的指针。



> **应用**

1. **缓存淘汰算法**：LRU算法中，双向链表可以用来维护最近访问的数据。当有新数据被访问时，将其插入链表头部；当内存不足需要淘汰数据时，可以快速删除链表尾部的节点。
2. **双向队列**：双向链表可以用来实现双向队列，即可以在队列的两端进行插入和删除操作。这给一些需要频繁在两端插入或删除元素的应用提供了便利，如任务调度、消息传递等。
3. **浏览器的前进后退功能**：浏览器中的浏览历史可以使用双向链表来实现。当用户点击后退按钮时，可以从当前页面节点沿着前指针遍历到上一个页面；当用户点击前进按钮时，可以从当前节点沿着后指针遍历到下一个页面



# 缓存算法

## 简介

**缓存**：是一种提高读取性能的技术，在硬件与软件中都有应用，如CPU缓存、浏览器缓存等，但是缓存的容量十分有限。当缓存满了之后，又要读取新的数据时，就需要考虑那么些数据该被清理，那些数据该保留？这就需要有缓存算法来决定

常见的缓存淘汰算法有先进先出`FIFO`、最少使用`LFU`(Least Frequently Used)、最近最少使用`LRU`(Least Recently Used)缓存淘汰算法。下文将介绍LRU与LFU

## LRU算法

> leetcode题目：[146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

双向链表适合实现LRU（Least Recently Used）算法的主要原因是它具有快速插入、删除和移动节点的能力。单链表也可以实现LRU，只不过效率和实现肯定没有双向链表好。

在LRU题目中**节点定义**如下：node保存的是键值对(可以理解为kv数据表)，意思是通过指定的键(key)可以访问到指定的值(vaule)。

```cpp
struct Node{
    struct Node* next;
    struct Node* prev;
    int key,val;
    Node():key(0),val(0),prev(nullptr),next(nullptr){}
    Node(int k,int v):key(k),val(v),prev(nullptr),next(nullptr){}
};
```



代码框架如下：

```cpp
class LRUCache {
public:
    LRUCache(int capacity) {
    }
    
    int get(int key) {
    }
    
    void put(int key, int value) {
    }
};
```

主要提供了两个接口

- `get`：输入键为参数，返回指定键所对应的值。

- `put`：输入键、值两个参数，将键值对作为一个节点保存到LRU缓存中。当缓存满后删除最不常用的键值节点。

本题比较困难的点就是要在O(1)的时间复杂度内实现：`put`与`get`操作

> 基本实现

要想在O(1)的时间复杂度内访问到指定的数据，那么我们肯定是需要使用哈希表的，所以我们可以定义一个`key_To_node;`数据成员，用于记录键与节点之间的**一对一映射**。其余的数据成员看下述代码注释：

```cpp
class LRUCache {
public:
	....
private:
    int size;  //当前节点数量
    int m_cap; //链表最大节点容量
    Node *head;//dummy node
    unordered_map<int, Node*> key_To_node;//哈希表
};
LRUCache::LRUCache(int cap){
    head = new Node();//空节点
    head->prev = head->next = head;
    this->size = 0;
    this->m_cap = cap;
}
```





> 接口实现：LRU算法

LRU实现的**基本思路**：只要访问了该节点的数据，就将这个节点移动到链表头部，缓存容量满后就删除链表尾部就可以了。具体可以细分为下

1. 访问的数据**已经缓存**：将节点移到链表头部，表示最近使用的数据。

2. 访问的数据**没有缓存**：存在两种情况，如下所示

   - 如果缓存**未满**，则将新数据插入**链表头部**，并更新缓存。

   - 如果缓存**已满**，则需要淘汰链表尾部的节点，即最久未使用的数据，然后将新数据插入**链表头部**。(实际上我们并不需要将节点删除，修改这个节点数据域，并移动到链表头部就可以了)



- `get`：通过`map`的`count`函数查询节点是否已经缓存，并使用`makeRcently`将访问的节点置于链表头部，反之返回-1。

```cpp
int LRUCache::get(int key) {
    if(key_To_node.count(key)){
        Node* p = key_To_node[key];
        makeRecently(p);
        return p->val;
    }
    return -1;
}
```



- `put`：put可以分为三种情况如下所示：	
  - 已缓存输入键值，修改指定节点的vaule值，调用`makeRecently`将节点置于链表头部。
  - 未缓存且链表**未满**：将新节点使用头插的方式插入到链表中。
  - 未缓存且链表**已满**：淘汰尾节点(`head->prev`)，在此我们只需要修改节点的value并且将这个节点移动到链表头部就可以了。

```cpp
void LRUCache::put(int key,int value) {
    if(key_To_node.count(key)){
        //cached key
        Node * p = key_To_node[key];
        p->val = value;
        makeRecently(p);
        return;
    }
    if(this->size < this->m_cap){
        insertHead(key,value);
    }else{
        Node * p = head->prev;
        key_To_node.erase(p->key);
        makeRecently(p);
        p->key = key;
        p->val = value;
        key_To_node[key] = p;
    }
}
```



- `makeRecently`：这个函数是实现LRU算法的关键，其原理就是，将指定节点从链表中移除`remove`(但是不删除)，在将这个节点移动了链表头部即可。这个函数在指定键被访问时需要调用改函数。

```c
void LRUCache::remove(Node *node){
    node->prev->next = node->next;
    node->next->prev = node->prev;
}


void LRUCache::moveTohead(Node * node){
    node->prev = head;
    node->next = head->next;
    head->next->prev = node;
    head->next = node;
}

void LRUCache::makeRecently(Node* node){
    if(node != head->next){//该节点不是链表第一节点
        //连接前后节点
        remove(node);
        //连接到头节点
        moveTohead(node);
    }
}
```



> 头插：

根据LRU的定义：我们只需要使用头插的方式，将新节点插入链表即可。之后就是将键与节点建立映射，并增加size字段。

```cpp
void LRUCache::insertHead(int key,int val){
    Node *node = new Node(key,val);
    //head insert O(1)
    head->next->prev = node;
    node->next = head->next;
    node->prev = head;
    head->next = node;
    //记入哈希表
    key_To_node[key] = node;
    this->size++;
}
```



> 完整的LRUCache类成员函数与成员定义

```cpp
//环形链表
class LRUCache{
public:
	LRUCache(int);
    //用户接口
    int get(int);
    void put(int,int);
private:
	void insert(int, int);
    void destroyList();
    void makeRecently(Node*);
    void moveTohead(Node*);//将节点移动到head->next
    void remove(Node*);    //断开指定节点，但不删除该节点
    int size;
    int m_cap;
    Node *head;//dummy node
    unordered_map<int, Node*> key_To_node;
};
```

在上述方法中没有设置析构函数，在可以参照上文实现析构函数以销毁链表以及节点。

## LFU算法

> leetcode题目：[460. LFU 缓存](https://leetcode.cn/problems/lfu-cache/)



## 简介

实现 `LFUCache` 类：

- `LFUCache(int capacity)` - 用数据结构的容量 `capacity` 初始化对象
- `int get(int key)` - 如果键 `key` 存在于缓存中，则获取键的值，否则返回 `-1` 。
- `void put(int key, int value)` - 如果键 `key` 已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量 `capacity` 时，则应该在插入新项之前，移除最不经常使用的项。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 **最久未使用** 的键。

为了确定最不常使用的键，可以为缓存中的每个键维护一个 **使用计数器** 。使用计数最小的键是最久未使用的键。

当一个键首次插入到缓存中时，它的使用计数器被设置为 `1` (由于 put 操作)。对缓存中的键执行 `get` 或 `put` 操作，使用计数器的值将会递增。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

> 

![image-20231005231247209](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20231005231247209.png)

```cpp
 

class LFUCache {
public:
    LFUCache(int capacity):m_cap(capacity) {
        head = new Node();
        head->next = head->prev = head;
    }
    
    int get(int key) {
        if(key_To_node.count(key)){
            Node* p =key_To_node[key];
            p->freq++;
            increFrequency(p->freq,p);
            return p->val;
        }
        return -1;
    }
    
    void put(int key, int value) {
        if(key_To_node.count(key)){
            //cached key
            Node *p = key_To_node[key];
            p->val = value;
            p->freq++;
            increFrequency(p->freq, p);
            return;
        }
        Node *node = nullptr;
        if(key_To_node.size() < this->m_cap){//not full
            node = new Node(key,value);
            insertTail(node);
        }else{ //full cache
            node = head->prev;//双向链表的最后一个节点
            key_To_node.erase(node->key);
            node->key = key;
            node->val = value;
            node->freq = 1;//重置频率
        }
        increFrequency(1,node);
        key_To_node[key] = node;
    }

private:
    void increFrequency(int freq,Node* node){
        Node *front = nullptr;
        if(freq_To_front.count(freq))
            //存在当前频率队列
            front = freq_To_front[freq];
        else if(freq_To_front.count(freq-1))
            //不存在当前频率队列，先将该节点放到上一频率的头部
            front = freq_To_front[freq-1];

        //升级频率队列的节点是上一级的队首，那么需要进行更改上级的freq_To_front节点
        if(freq_To_front.count(freq-1) && freq_To_front[freq-1] == node){
            if(node->next->freq == freq-1)
                //上一级队列还有剩余节点，修改freq_To_front
                freq_To_front[freq-1] = node->next;
            else
                //上一级队列没有剩余节点，删除节点
                freq_To_front.erase(freq-1);
        }
        if(front)//移动节点作为改频率队列队瘦
            makeRecently(front,node);

        //更新当前频率队列
        freq_To_front[freq] = node;
    }
    //连接前后节点，剔除该节点
    void remove(Node* node){
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }
    //插入node节点到front前
    void move(Node* front,Node* node){
        node->next = front;
        node->prev = front->prev;
        front->prev->next = node;
        front->prev = node;
    }

    void insertTail(Node *node){
        head->prev->next = node;
        node->prev = head->prev;
        node->next = head;
        head->prev = node;
    }

    void makeRecently(Node* front,Node* node){
        if(front != node){//将节点置于最前
            //断开当前节点
            remove(node);
            //插入到front之前
            move(front,node);
        }
    }   

    unordered_map<int,Node*> key_To_node;         //key  -> node
    //每一个freq级的最近最多使用的节点(most recently node)，作为该freq级队列的队首
    unordered_map<int,Node*>  freq_To_front;      //freq -> front
    Node* head;
    int m_cap;
};
```



运行结果：

![LFU](https://ember-img.oss-cn-chengdu.aliyuncs.com/LFU.png)