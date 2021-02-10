---
description: 一些常见的算法模板，必须要滚瓜烂熟
---

# 常见算法模板与套路

## 1 递归模板

```text
public void recur(int level, int param) {
    // 终止条件
    if (level > MAX_LEVEL) {
        // 处理结果
        return;
    }
    
    // 处理当前逻辑
    process(level, param);
    
    // 下钻
    recur(level + 1, newParam);
    
    // 恢复状态
    // 有些也可以不处理。
}
```

思维要点：

* 不要人肉进行递归（最大误区）
* 找到最近最简方法，将其拆解成可重复解决的问题（重复子问题）
* 数学归纳法思维

## 2 分治模板

```text
public void divideConquer(problem, param...) {
    // 终止条件
    if (problem == null) {
        printResult();
        return;
    }
    // 处理数据
    data = prepareData(problem);
    problem[] subProblems = splitProblem(problem, data);
    subResult[] subResults = new subResult[subProblems.length];
    // 分别处理子问题
    for (problem subProblem : subProblems) {
        subResult = divideConquer(subProblem, param...);
        subResults.add(subResult);
    }
    # 合并子结果，生成最终结果
    result = processResult(subResults);
}
```

## 3 回溯

核心步骤：

> 采用试错的思想，尝试分步去解决一个问题。在分步解决问题的过程中，当它通过尝试发现现有的分步答案不能得到有效的正确解答的时候，它将取消上一步甚至上几步的计算，再通过其他的可能的分步解答再次尝试寻找问题的答案。

会存在如下两种情况：

1. 找到一个可能存在的正确答案
2. 在尝试了所有可能的分步方法后宣告该问题没有答案。

## 4 深度优先搜索 

递归写法

```text
Set visited = new HashSet();
void dfs(Node node, Set visited) {
    // 终止条件
    if (visited.contain(node)) {
        return
    }
    visited.add(node)
    // 处理当前node
    for (Node childNode: node.children()) {
        if (!visited.contain(childNode)) {
            dfs(childNode, visited);
        }
    }
}

```

非递归写法

```text
void dfs(tree) {
    if (tree.root == null) {
        return new Node[];
    }
    Set visited = new HashSet();
    Stack stack = new Stack {};
    stack.push(tree.root);
    
    while (stack.size == 0) {
        Node node = stack.pop();
        visited.add(node);
        // 处理当前节点
        process(node);
        Node[] nodes = node.children();
        stack.push(nodes);
    }
}
```

## 5 广度优先算法

```text
void BFS(tree.root) {
    Queue queue = new ArrayDeque();
    Set visited = new HashSet();
    queue.append(tree.root);
    visited.add(tree.root);
    
    while(queue.isNotEmpty()) {
        Node node = queue.pop();
        visited.add(node);
        // 处理当前节点
        process(node);
        Nodes[] nodes = node.children();
        queue.push(nodes);
    }     
    
    // 其他处理
    // ...
}
```

## 6 贪心算法

> 贪心算法是一种在每一个选择中都采取在当前状态下最好或者最优的选择，从而希望导致结果是全局最好的算法。

贪心与动态规划的区别：  


> 贪心对每个问题都作出选择，不能回退；而动态规划则会保存以前的运算结果，并根据以前的结果对当前进行选择，有回退功能。



## 7 二分查找

前提：

> 1、目标函数单调性（单调递增或者递减）
>
> 2、存在上下界
>
> 3、能够通过索引访问

模板：

```text
int binarySearch(int[] arr, int target) {
    int left = 0;
    int  right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (target == arr[mid]) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    // 没找到
    return -1;
}
```

## 8 动态规划

动态规划 和 递归，或者分治 没有根本上的区别。

共性：  找到重复子问题。

差异性：最优子结构，中途可以淘汰次优解。

![](../.gitbook/assets/image%20%284%29.png)







