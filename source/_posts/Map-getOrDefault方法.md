---
title: Map.getOrDefault方法
date: 2020-11-27 09:19:11
tags:
	- Java
	- Map
---

源码如下：

```java
default V getOrDefault(Object key, V defaultValue) {
        V v;
        //存在此key的值或者存在key，返沪key对应的value，否则返回你设置的defaultValue
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }
```

用法：

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Map<Integer, Integer> countAB = new HashMap<Integer, Integer>();
        for (int u : A) {
            for (int v : B) {
                //存在就取值加一，否则返回defaultValue = 0
                countAB.put(u + v, countAB.getOrDefault(u + v, 0) + 1);
            }
        }
        int ans = 0;
        for (int u : C) {
            for (int v : D) {
                if (countAB.containsKey(-u - v)) {
                    ans += countAB.get(-u - v);
                }
            }
        }
        return ans;
    }
}

```

