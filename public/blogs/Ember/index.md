# 八大排序

> 简单排序：时间复杂度为$O(n^2)$

- [冒泡排序](##冒泡排序)

- [插入排序](##插入排序)

  - 直接插入排序

  - 希尔排序

- [选择排序](##选择排序)
  - 简单选择排序
  - 堆排序

> 分治思想+排序：时间复杂度为$O(nlogn)$

- [快速排序](##快速排序)
- [归并排序](##归并排序)
  - 递归版
  - 非递归版



> 线性排序

- [桶排序](##桶排序)
- [计数排序](##计数排序)
- [基数排序](##基数排序)



|          | 原地排序 | 稳定 | 最差时间复杂度 | 平均时间复杂度 |    空间复杂度    |
| -------- | -------- | ---- | -------------- | -------------- | :--------------: |
| 冒泡排序 | 是       | 是   | $ O(N^2) $     | $ O(N^2) $     |      $O(1)$      |
| 插入排序 | 是       | 是   | $ O(N^2) $     | $ O(N^2) $     |      $O(1)$      |
| 选择排序 | 是       | 否   | $ O(N^2) $     | $ O(N^2) $     |      $O(1)$      |
| 希尔排序 | 是       | 否   | $O(N^2) $      | $ O(NlogN) $   |      $O(1)$      |
| 归并排序 | 否       | 是   | $O(NlogN)$     | $O(NlogN)$     |      $O(N)$      |
| 快速排序 | 是       | 否   | $ O(N^2) $     | $O(NlogN)$     | $O(logN)$ $O(N)$ |
| 桶排序   |          |      |                |                |                  |



# 简单排序

## 冒泡排序

> 思路：

​	从头开始遍历，两两元素比较进行交换，最终将最大的数固定到第N-1位(数组大小为N)，这个过程像鱼吐泡泡一样就被称之为冒泡，接着逐渐将第2、3、4...大的数固定n-2、n-3、...，1，0位，这样就排好序了。

![img](https://ember-img.oss-cn-chengdu.aliyuncs.com/v2-1543c0b97237bb55063e033959706ca0_b.gif)



> 实现

```c
void bubble_sort(int arr[],int n){
	for(int i = 0;i < n;++i){
		int flag = 0;//标记数组是否发生交换
		for(int j = 1;j < n - i;++j){
			if(arr[j-1] > arr[j]){
				int temp = arr[j-1];
				arr[j-1] = arr[j];
				arr[j] = temp;
				flag = 1;
			}
		}
		if(!flag) break;//没有发生交换说明已经排好序了
	}
}
```



## 选择排序

> 思路：

​	比较相邻两个元素的大小，如果顺序不正确就进行**交换**(swap)，直到整个序列按照要求排序为止。

![img](https://ember-img.oss-cn-chengdu.aliyuncs.com/20190309160955666.gif)

> 实现：

```c
void select_sort(int arr[],int n){
	for(int i = 0;i < n;++i){
		int min_idx = i;//记录最小元素的索引
		for(int j = i + 1;j < n;++j){
			if(arr[j] < arr[min_idx])
				min_idx = j;
		}
		int temp = arr[min_idx];
		arr[min_idx] = arr[i];
		arr[i] = temp;
	}
}
```



## 插入排序

> 思路：

​	每次选择一个元素，并且将这个元素和整个数组中的所有元素进行比较，然后插入到合适的位置。

![img](https://ember-img.oss-cn-chengdu.aliyuncs.com/v2-f87ad7d8ad54379dd81f02fcf9b91f49_b.gif)

> 实现：



```c
void insert_sort(int arr[],int n){
	for(int i = 0;i < n;++i){
		int temp = arr[i];
		int j = i - 1;
		for(;j >= 0;--j){
            //移动数组元素
			if(arr[j] > temp)
				arr[j+1] = arr[j];
			else //找到合适的位置
				break;	
		}
		arr[j+1] = temp;
	}
}
```



## 希尔排序

> 思路：

​	是基于插入排序的排序算法，它通过将一组元素分割成多个子序列来改进插入排序的性能。希尔排序的时间复杂度取决于间隔序列的选择。

```c
void shell_sort(int arr[],int n){
	for (int gap = n/2; gap > 0; gap /= 2) 
    { 
		for(int i = 0;i < n;++i){
			int temp = arr[i];
			int j = i - 1;
            //移动数组元素
			for(;j >= 0;--j){
				if(arr[j] > temp)
					arr[j+1] = arr[j];
				else 
					break;	
			}
			arr[j+1] = temp;
		}
	}
}
```



# 分治排序

## 归并排序

> 思路：

​	先把左半边数组排好序，再把右半边数组排好序，然后把两半数组合并。



![在这里插入图片描述](https://ember-img.oss-cn-chengdu.aliyuncs.com/f2c61238bbe745e19301b20cf103d059.gif)



![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/gibkIz0MVqdEYjsa02Q7unU9ErV4p4Ly0FELvuVd1ia6GXofXJVibyqkSpMhVGEDsPmjHgBUJW5Mg9FGN6gt7SuCg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1)

### 分治

```c
/*分治-->递归 */
//也可以选择使用函数重载
void merge_sort_c(int arr[],int l,int r){
	if(l >= r)
		return;
	//中间位置	
	int m = l + (r-l)/2 ;//等同于 (r+l)/2 这里是为了防止整数相加溢出 
	
	//分治递归 与二叉树的遍历十分相似 
	merge_sort_c(arr,l,m);
	merge_sort_c(arr,m+1,r);
	
	//开始合并时，也就说明遍历到达叶子节点 
	merge(arr,l,m,r); 
}

void merge_sort(int arr[],int n){
	merge_sort_c(arr,0,n-1);
} 
```



### 数组合并

​	归并少不了的步骤肯定是有合并操作的，下述leetcode88题，考察的就是两个有序数组的合并操作。

> [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

```c
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
    int i1 = m - 1,i2 = n - 1,idx = m + n - 1;

    while(i1 >= 0 && i2 >= 0)
        if(nums1[i1] > nums2[i2])
            nums1[idx--] = nums1[i1--];
        else
            nums1[idx--] = nums2[i2--];

    while(i2 >= 0)
        nums1[idx--] = nums2[i2--];
}
```



那么根据上述的实现代码，我们只需要稍加修改便能够实现归并排序的**merge**函数



```c
void merge(int arr[], int l, int m, int r) 
{ 
	
    int n1 = m - l + 1; 
    int n2 =  r - m; 
	//申请两个临时数组，分为mid左方和右方 
    int *l_arr = malloc(n1 * sizeof(int)),*r_arr = malloc(n2 * sizeof(int));
    
    //初始化临时数组 
    for(int i = 0;i < n1;++i)
    	l_arr[i] = arr[l + i];
    for(int j = 0;j < n2;++j)
 	   	r_arr[j] = arr[m+ j+ 1];
 	   	
 	int i = n1 - 1, j = n2 - 1, idx = r;
	
	while(i >= 0 && j >= 0) 
	    if (l_arr[i] > r_arr[j]) 
	        arr[idx--] = l_arr[i--]; 
	    else
	        arr[idx--] = r_arr[j--]; 
     
	while(i >= 0)
		arr[idx--] = l_arr[i--]; 
		
	while(j >= 0)
		arr[idx--] = r_arr[j--]; 
		
    free(l_arr);free(r_arr);
}
```



## 快速排序

> 思路：

​	从数组中选取一个数作为一个轴(pivot)，将数组大于pivot的数放到它的右边，小于它的则放到它的左边，那么这个数就已经拍好序了，这个过程称为分区(partition)。再通过分治的思想将剩下的数进行分区，如此就是排好序了。

​	简单来说就是**先将一个元素排好序，然后再将剩下的元素排好序**。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/gibkIz0MVqdGZy8ttAE2M0GxYNH54ibyAfn2jDiaSNzia8GYnnIkeJTLuKMO6VFLMjrUrRXw5v9RDVG6awlNjPl0xA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://ember-img.oss-cn-chengdu.aliyuncs.com/v2-c411339b79f92499dcb7b5f304c826f4_b.gif)





```c
void quick_sort_c(int arr[],int l,int r){
    if(l >= r) return;

    int p = partition(arr,l,r);

    quick_sort_c(arr,l,p-1);
    quick_sort_c(arr,p+1,r);
}
void quick_sort(int arr[],int n){
    quick_sort_c(arr,0,n-1);
}
```





```c
void swap(int *a,int *b){
    int temp = *a;
    *a = *b;
    *b = temp;
}

int partition(int arr[],int l,int r){
    int pivot = arr[r];
    int i = l;
    for(int j = l;j < r;j++){
        if(arr[j] < pivot){
            swap(&arr[j],&arr[i]);
            i++;
        }
    }
    swap(&arr[i],&arr[r]);
    return i;
}
```





### 数组中的第K个最大元素

[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

```cpp
class Solution {
public:
    int quickSelect(vector<int>& a, int l, int r, int index) {
        int q = randomPartition(a, l, r);
        if (q == index) {
            return a[q];
        } else {
            return q < index ? quickSelect(a, q + 1, r, index) : quickSelect(a, l, q - 1, index);
        }
    }

    inline int randomPartition(vector<int>& a, int l, int r) {
        int i = rand() % (r - l + 1) + l;
        swap(a[i], a[r]);
        return partition(a, l, r);
    }

    inline int partition(vector<int>& a, int l, int r) {
        int x = a[r], i = l - 1;
        for (int j = l; j < r; ++j) {
            if (a[j] <= x) {
                swap(a[++i], a[j]);
            }
        }
        swap(a[i + 1], a[r]);
        return i + 1;
    }

    int findKthLargest(vector<int>& nums, int k) {
        srand(time(0));
        return quickSelect(nums, 0, nums.size() - 1, nums.size() - k);
    }
};
```



## 对比



> 稳定性

对于序列中的相同元素，如果排序之后它们的相对位置没有发生改变，则称该排序算法为「稳定排序」，反之则为「不稳定排序」。

如果单单排序 int 数组，那么稳定性没有什么意义。但如果排序一些结构比较复杂的数据，那么稳定性排序就有更大的优势了。

比如说你有若干订单数据，已经按照订单号排好序了，现在你想对订单的交易日期再进行排序：

如果用稳定排序算法（比如归并排序），那么这些订单不仅按照交易日期排好了序，而且相同交易日期的订单的订单号依然是有序的。

但如果你用不稳定排序算法（比如快速排序），那么虽然排序结果会按照交易日期排好序，但相同交易日期的订单的订单号会丧失有序性。

**在实际工程中我们经常会将一个复杂对象的某一个字段作为排序的 `key`，所以应该关注编程语言提供的 API 底层使用的到底是什么排序算法，是稳定的还是不稳定的，这很可能影响到代码执行的效率甚至正确性**。



> 参考：

- [快速排序的正确理解方式及运用](https://mp.weixin.qq.com/s/8ZTMhvHJK_He48PpSt_AmQ)
- [归并排序的正确理解方式及运用](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247495989&idx=1&sn=30e34ac75dd1c724205e9c8b0f488e35&scene=21#wechat_redirect)



