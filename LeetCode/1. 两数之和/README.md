# 1. 两数之和

[题目链接](https://leetcode-cn.com/problems/two-sum/)

## 题目要求

- 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

  你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

   

  示例:

  给定 nums = [2, 7, 11, 15], target = 9

  因为 nums[0] + nums[1] = 2 + 7 = 9
  所以返回 [0, 1]

## 解题思路

### 方法1：暴力

遍历每个元素，target-x相等的数

#### 代码

````java
public int[] twoSum(int[] nums, int target) {

    int[] sums = new int[2];

    for (int i = 0; i < nums.length; i++) {
        for (int j = nums.length - 1; j > i; j--) {
            if (nums[i] + nums[j] == target) {
                sums[0] = i;
                sums[1] = j;
                return sums;
            }
        }
    }
    return null;
}
````

### 方法2：哈希表

进行哈希表映射，每次先查找哈希标中是否存在target-num[i]的元素，如果有直接返。

#### 代码

````txt
public int[] twoSum(int[] nums, int target) {
    Map<Integer,Integer> map = new HashMap<>();
    for(int i = 0; i < nums.length;i++){
        if(map.containsKey(target - nums[i])){
            return new int[]{map.get(target - nums[i]),i};
        }
        map.put(nums[i],i);
    }
    return null;
}
````
