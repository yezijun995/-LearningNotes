# 剑指 Offer 64. 求1+2+…+n

[题目链接](https://leetcode-cn.com/problems/qiu-12n-lcof/)

## 题目要求

- 求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

   

  示例 1：

  输入: n = 3
  输出: 6
  示例 2：

  输入: n = 9
  输出: 45


  限制：

  1 <= n <= 10000

## 解题思路

### 方法1：递归

本题中，无法使用关键字，与判断语句，那么可以用逻辑运算符的短路特性来做

#### 代码

````java
public int sumNums(int n) {
    int sum = n;
    boolean flag = n > 0 && (sum +=sumNums(n - 1)) > 0;
    return sum;
}
````

### 方法2：

#### 代码

````txt
public int sumNums(int n) {
    return (1 + n) *n /2;
}
````
