---
title: Isomorphic Strings
description: 本篇文章将介绍：保证字符串 s 和 t 的字符一一对应，即字符串 s 和 t 为同构字符串。使用两个哈希表来维护字符对应关系，时间复杂度 O(n)。
categories:
- Leet Code
---

> 胜者先胜而后求战，败者先站而后求胜

## 同构字符串

[进入 Leet Code 查看题目：https://leetcode-cn.com/problems/isomorphic-strings/](https://leetcode-cn.com/problems/isomorphic-strings/)

给定两个字符串 s 和 t ，判断它们是否是同构的。

如果 s 中的字符可以按某种映射关系替换得到 t ，那么这两个字符串是同构的。

每个出现的字符都应当映射到另一个字符，同时不改变字符的顺序。不同字符不能映射到同一个字符上，相同字符只能映射到同一个字符上，字符可以映射到自己本身。

**示例1：**

```
输入：s = "egg", t = "add"
输出：true
```

**示例2：**

```
输入：s = "foo", t = "bar"
输出：false
```

## 分析
只要保证 s 中的字符和 t 中的字符一一对应，那么字符串 s 和字符串 t 就是同构字符串。

**例一：**
`s = "egg", t = "add"`

其中 `e -> a` `g -> d`， 所以字符串 s 和字符串 t 是同构字符串。

**例二：**
`s = "foo", t = "bar"`

其中 `f -> b` `o -> a` `o -> r`， 字符 o 同时对应 a 和 r，所以字符串 s 和字符串 t 不是同构字符串。

利用哈希表存储 s 中每个字符对应的 t 中的字符，如果发现重复则不是同构字符串，同样利用哈希表存储 t 中的每个字符对应的 s 中的字符，如果发现重复则不是同构字符串。

[进入 Leet Code 查看：https://leetcode-cn.com/problems/isomorphic-strings/solution/tong-gou-zi-fu-chuan-ha-xi-biao-tu-jie-b-4jyt/](https://leetcode-cn.com/problems/isomorphic-strings/solution/tong-gou-zi-fu-chuan-ha-xi-biao-tu-jie-b-4jyt/)

## 图解
![同构字符串.gif](https://huapeiliang.github.io/assets/images/leetCode/isomorphicStrings.gif)

## 代码
```
class Solution {
    public boolean isIsomorphic(String s, String t) {
        HashMap<Character, Character> sMap = new HashMap<>();
        HashMap<Character, Character> tMap = new HashMap<>();
        for (int i=0; i<s.length(); i++){
            char sChar = s.charAt(i);
            char tChar = t.charAt(i);
            if ((sMap.containsKey(sChar) && sMap.get(sChar) != tChar) 
                || (tMap.containsKey(tChar) && tMap.get(tChar) != sChar)){
               return false;
            }
            sMap.put(sChar, tChar);
            tMap.put(tChar, sChar);
        }
        return true;
    }
}
```