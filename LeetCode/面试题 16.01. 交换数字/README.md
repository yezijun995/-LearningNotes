# 面试题 16.01. 交换数字

[题目链接](https://leetcode-cn.com/problems/swap-numbers-lcci/)

## 题目要求

编写一个函数，不用临时变量，直接交换numbers = [a, b]中a与b的值。

示例：

````txt
输入: numbers = [1,2]
输出: [2,1]提示：
````

- numbers.length == 2

## 解题思路

### 方法1：

#### 代码

````java
public int[] swapNumbers(int[] numbers) {
    numbers[0] += numbers[1];
    numbers[1] = numbers[0] - numbers[1];
    numbers[0] -= numbers[1];
    return numbers;
}
````

### 方法2：异或

#### 代码

````txt
public int[] swapNumbers(int[] numbers) {
    numbers[0] ^= numbers[1];
    numbers[1] ^= numbers[0];
    numbers[0] ^= numbers[1];
    return numbers;
}
````
