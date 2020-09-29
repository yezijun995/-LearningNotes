# 面试题 01.07. 旋转矩阵

[题目链接](https://leetcode-cn.com/problems/rotate-matrix-lcci/submissions/)

## 题目要求

给你一幅由 `N × N` 矩阵表示的图像，其中每个像素的大小为 4 字节。请你设计一种算法，将图像旋转 90 度。

不占用额外内存空间能否做到？

**示例1：**

```java
给定 matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

原地旋转输入矩阵，使其变为:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```

**示例2：**

```java
给定 matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

原地旋转输入矩阵，使其变为:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```

## 解题思路

### 方法1:

比较简单的方法，对数组进行旋转。

```java
class Solution {
    public void rotate(int[][] matrix) {
        int matrixTmp[][] = new int[matrix.length][matrix.length];
        for(int i = 0; i < matrix.length; i++){
            for(int j = 0; j < matrix.length; j++){
                matrixTmp[j][matrixTmp.length - i -1] = matrix[i][j];
            }
        }

        for (int i = 0; i < matrixTmp.length; i++) {
            for (int i1 = 0; i1 < matrixTmp[0].length; i1++) {
             matrix[i][i1] = matrixTmp[i][i1];   
            }
        }
    }
}
```

### 方法2

先对对称轴进行旋转

```java
比如:
[1,2,3],
[4,5,6],
[7,8,9]
对称轴旋转
[1,4,7],
[2,5,8],
[3,6,9]
```

在对各行进行旋转

```java
[1,4,7],
[2,5,8],
[3,6,9]
各行旋转
[7,4,1],
[8,5,2],
[9,6,3]  
```

**代码**

```java
class Solution {
    public void rotate(int[][] matrix) {
        int s = matrix.length;
        int tmp;
        //对称轴旋转
        for (int i = 0; i < s - 1; i++) {
            for (int j = i + 1; j < s; j++) {
                tmp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = tmp;
            }
        }

        //行旋转
        int mid = s / 2;
        for (int i = 0; i < s; i++) {
            for (int j = 0; j < mid; j++) {
                tmp = matrix[i][j];
                matrix[i][j] = matrix[i][s - j - 1];
                matrix[i][s - j -1] = tmp;
            }
        }
    }
}
```

