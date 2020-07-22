# 面试题 01.01. 判定字符是否唯一

[题目链接](https://leetcode-cn.com/problems/is-unique-lcci/)

## 题目要求

实现一个算法，确定一个字符串 s 的所有字符是否全都不同。

示例 1：

输入: s = "leetcode"
输出: false 
示例 2：

输入: s = "abc"
输出: true
限制：

0 <= len(s) <= 100
如果你不使用额外的数据结构，会很加分。

## 解题思路

### 方法1：哈希表

创建一个数组相对应的哈希表，Key为数字，Value为出现的的次数。当Value为1时返回

#### 代码

````java
class Solution {
    public boolean isUnique(String astr) {
        Map<Character, Integer> map = new HashMap<>();
        for (int i = 0; i < astr.length(); i++) {
            if (!map.containsKey(astr.charAt(i)))
                map.put(astr.charAt(i), 1);
            else return false;
        }
        return true;
    }
}
````

### 方法2：暴力破解

对每个字符进行循环破解。

#### 代码

````txt
class Solution {
    public boolean isUnique(String astr) {
        for (char ch: astr.toCharArray()){
        if (astr.indexOf(ch) != astr.lastIndexOf(ch)) {
        	return false;
        }
    }
    return true;
    }
}
````

### 方法3：位运算

题目限制如果不使用数据结构可以额外加分，那么可以使用位运算，二进制位当为1时候表示已经出现过，而当0的时候表示还没出现过。

#### 代码

````java
class Solution {
    public boolean isUnique(String astr) {
        long lStr = 0, index;
        for (char c : astr.toCharArray()) {
            index = 1L << c;
            if ((lStr & index) != 0) {
                return false;
            }
            lStr |= index;
        }
        return true;
    }
}
````

