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



