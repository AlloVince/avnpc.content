---  
title: 用 Javascript 写排序算法  
s: sorting-algorithms-by-javascript  
date: 2019-07-15 22:28:00  
published: true
tags:  
 - Sorting  
 - Algorithms  
 - 排序  
 - 算法  
 - Javascript  
---  
  
这是个人算法知识复习及梳理的第一篇，用 [Javascript 实现主要的排序算法](https://avnpc.com/pages/sorting-algorithms-by-javascript)，一直觉得算法是自己的弱项，因此写的时候也是诚惶诚恐，希望尽量做到好的代码可读性，同时少犯错误。

为了方便自己也帮到更多人，所有的排序算法都会简单说一下自己的思路，并附上自己能找到的最容易理解的可视化动画。

至于为什么选择用 Javascript，则是因为我觉得 Javascript 是最方便运行和调试的，只需要复制代码粘贴到浏览器的控制台就可以了，我为所有的算法附上了测试用例，通过引入 [Mocha](https://mochajs.org/) 就可以在浏览器中显示用例的通过情况。

因此我将所有的算法都整理为 CodePen 形式的在线演示，点击下文每段代码后的「在线运行」，就可以在浏览器中直接运行所有测试用例，你如果想自己进行调试或修改，只需要点击在线演示右上角的「Edit On CodePen」，就可以 Fork 一个到自己的 CodePen 修改为自己想要的版本了。如果你也曾经跟我一样对算法感到头疼，相信这是能最大限度降低上手门槛的方法。

所有 CodePen 在线演示地址可查看 [Sorting Algorithms](https://codepen.io/collection/nNvpBx/)， 以下所有算法及描述按有小到大排序。


## 冒泡排序 (Bubble Sort)

冒泡排序应该是大多数人的入门算法了，其实个人觉得「冒泡」一词有误导之嫌，叫「交换排序」更符合实际情况。因为从字面理解「冒泡」好像是选中第一数，将其一直移动到最后，但实际上一次「冒泡」过程中，可能数组中的每个元素位置都会被改变。通过不断交换相邻元素的方式，让最大的元素排到最后，个人觉得这样更容易记忆。算法描述如下

1.  比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2.  对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3.  针对所有的元素重复以上的步骤，除了最后一个。
4.  持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

[动画演示冒泡排序](http://www.algomation.com/player?algorithm=58b307b11adc730400031082)

```js
function bubbleSort(nums) {
  // 外循环：一共有 i 个元素要排序
  for (let i = 0; i < nums.length - 1; i++) {
    // 内循环：不断交换相邻元素, 终止条件 -i 是最后 i 个元素已经排好序
    for (let j = 0; j < nums.length - 1 - i; j++) {
      if (nums[j] > nums[j + 1]) {
        //js 中交换两个元素可以写成 [a, b] = [b, a] 来省去中间变量
        [nums[j], nums[j + 1]] = [nums[j + 1], nums[j]];
      }
    }
  }
  return nums;
}
```

[在线运行冒泡排序](https://codepen.io/AlloVince/pen/BgGMqY)

- 最坏时间复杂度 $O(n^2)$
- 最优时间复杂度 $O(n)$
- 平均时间复杂度 $O(n^2)$
- 最坏空间复杂度 $O(n)$

## 选择排序 (Selection Sort)

选择排序应该是最符合人类思维的排序方式了，算法描述也很简单

1. 在未排序的元素中找到最小元素，存放到排序序列的起始位置
2. 再从剩余未排序元素中继续寻找最小元素，然后放到已排序序列的末尾。
3. 以此类推，直到所有元素均排序完毕。

[动画演示选择排序](https://upload.wikimedia.org/wikipedia/commons/9/94/Selection-Sort-Animation.gif)

```js
function selectionSort(nums) {
  for (let i = 0; i < nums.length; i++) {
    let minIndex = i;
    let min = nums[i];
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[j] < min) {
        min = nums[j];
        minIndex = j;
      }
    }
    [nums[i], nums[minIndex]] = [nums[minIndex], nums[i]];
  }
  return nums;
}
```

[在线运行选择排序](https://codepen.io/AlloVince/pen/gNQJdG)

- 最坏时间复杂度 $O(n^2)$
- 最优时间复杂度 $O(n^2)$
- 平均时间复杂度 $O(n^2)$
- 最坏空间复杂度 $O(n)$

## 插入排序 (Insertion Sort)

插入排序也是非常直观的一种排序，其实和玩扑克牌抽牌的过程非常相似，牌堆相当于未排序的部分，手牌相当于已排序的部分，当我们从牌堆拿起一张牌时，会根据手牌的排序，将牌插入到相应的位置，算法描述如下

1.  从第一个元素开始，该元素可以认为已经被排序
2.  取出下一个元素，在已经排序的元素序列中从后向前扫描
3.  如果该元素（已排序）大于新元素，将该元素移到下一位置
4.  重复步骤 3，直到找到已排序的元素小于或者等于新元素的位置
5.  将新元素插入到该位置后
6.  重复步骤 2~5

[动画演示插入排序](https://zh.wikipedia.org/wiki/File:Insertion-sort-example-300px.gif)

```js
function insertionSort(nums) {
  for (let i = 1; i < nums.length; i++) {
    let j = i;
    while (nums[j - 1] !== undefined && nums[j - 1] > nums[j]) {
      [nums[j - 1], nums[j]] = [nums[j], nums[j - 1]];
      j--;
    }
  }
  return nums;
}
```

[在线运行插入排序](https://codepen.io/AlloVince/pen/EBOJzO)

## 堆排序 (Heap Sort)

堆排序借助了堆（Heap）这个数据结构，堆是一种树形结构，普遍都采用二叉树实现。堆如果满足根节点始终是堆的最小节点，则称为最小堆（Min Heap）；如果根节点始终是最大节点，则称为最大堆（Max Heap）。

在理解了堆的基础上，堆排序的算法就非常简单，先将数据构造为最小堆，由于根节点始终是当前堆中最小的元素，所以不停的取根节点就可以了。

[动画演示堆排序](http://www.algomation.com/player?algorithm=58b44c8f0e406f04000c713d)

由于 Javascript 中并未内置堆这个数据结构，因此在代码演示中手动实现了一个，但在此不再赘述。你也可以自行引入其他第三方实现的堆。

```js
function heapSort(nums) {
  const minHeap = new MinHeap();
  //使用数组构建堆
  minHeap.inserts(nums);
  const res = [];
  //如果堆中还有元素
  while (!minHeap.isEmpty()) {
    //取出根节点放入已排序数组的末尾
    res.push(minHeap.remove());
  }
  return res;
}
```

[在线运行堆排序](https://codepen.io/AlloVince/pen/dBrNgN)


- 最坏时间复杂度 $O(n*log(n))$
- 最优时间复杂度 $O(n*log(n))$
- 平均时间复杂度 $O(n*log(n))$
- 最坏空间复杂度 $O(n)$


## 快速排序 (Quick Sort)

快排体现的是分治的思想，从算法描述上来看也非常简单，但老实说以人类（至少对于我）的思维习惯来说，由于存在递归，快排的算法显得并非那么直观，特别是对于快排的原地(In Place)排序版本更是如此。建议先从简单版本开始理解，多看动画演示。快排的算法描述如下

1.  挑选基准值：从数列中随机挑出一个元素（本例选择最后一个元素），称为“基准”（pivot）。
2.  分割：所有比基准值小的元素摆在基准前面，大于等于基准值的元素摆在基准后面。
3.  递归排序子序列：递归的将 1~2 步应用于小于基准值元素的子序列和大于基准值元素的子序列。

[动画演示快速排序](http://www.algomation.com/algorithm/quick-sort-visualization)

快排的简单版本

```js
function quickSort(nums) {
  if (nums.length <= 1) {
    return nums;
  }
  const pivot = nums.pop();
  const left = [];
  const right = [];
  while (nums.length > 0) {
    const n = nums.pop();
    n < pivot ? left.push(n) : right.push(n);
  } 
  return [...quickSort(left), pivot, ...quickSort(right)];
}
```

[在线运行快速排序简单版本](https://codepen.io/AlloVince/pen/GbwPzp)


快排的原地(In Place)排序版本

```js
function partition(nums, left, right) {
  const pivot = nums[right];
  let partitionIndex = left;
  for (let i = left; i < right; i++) {
    if (nums[i] < pivot) {
      [nums[i], nums[partitionIndex]] = [nums[partitionIndex], nums[i]];
      partitionIndex++;
    }
  }
  [nums[right], nums[partitionIndex]] = [nums[partitionIndex], nums[right]];
  return partitionIndex;
}

function quickSort(nums, left, right) {
  left = left === undefined ? 0 : left;
  right = right === undefined ? nums.length - 1 : right;
  if (left >= right) {
    return nums;
  }
  
  const partitionIndex = partition(nums, left, right);
  quickSort(nums, left, partitionIndex - 1);
  quickSort(nums, partitionIndex + 1, right);
  return nums;
}
```

- 最坏时间复杂度 $O(n^2)$
- 最优时间复杂度 $O(n*log(n))$
- 平均时间复杂度 $O(n*log(n))$
- 最坏空间复杂度 $O(log(n))$

## 归并排序 (Merge Sort)

归并排序也是非常能提现分治思想的排序算法，同时也比较符合人的思维方式，算法描述如下，可能看描述还比较抽象，看动画演示就非常直观了。

1.  分割：递归地把当前序列平均分割成两半。
2.  集成：在保持元素顺序的同时将上一步得到的子序列集成到一起（归并）。

[动画演示归并排序](http://www.algomation.com/player?algorithm=58bb32885b2b830400b05123)

```js
function merge(left, right) {
  const sorted = [];
  while (left.length && right.length) {
    let min = left[0] < right[0] ? left.shift() : right.shift();
    sorted.push(min);
  }
  return sorted.concat(left, right);
}

function mergeSort(nums) {
  if (nums.length <= 1) {
    return nums;
  }
  const mid = Math.floor(nums.length / 2);
  return merge(mergeSort(nums.slice(0, mid)), mergeSort(nums.slice(mid)));
}
```

[在线运行归并排序](https://codepen.io/AlloVince/pen/XLyGOM)

- 最坏时间复杂度 $O(n*log(n))$
- 最优时间复杂度 $O(n*log(n))$
- 平均时间复杂度 $O(n*log(n))$
- 最坏空间复杂度 $O(n)$
