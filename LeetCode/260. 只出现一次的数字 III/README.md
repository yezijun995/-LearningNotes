# 只出现一次的数字III

[题目链接](https://leetcode-cn.com/problems/single-number-iii/)

## 题目要求

给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

示例 :

输入: [1,2,1,3,2,5]
输出: [3,5]
注意：

结果输出的顺序并不重要，对于上面的例子， [5, 3] 也是正确答案。
你的算法应该具有线性时间复杂度。你能否仅使用常数空间复杂度来实现？

## 结题思路

### 方法1：哈希表

创建一个数组相对应的哈希表，Key为数字，Value为出现的的次数。当Value为1时返回

#### 代码

````java
public int[] singleNumber(int[] nums) {
    ArrayList<Integer> list = new ArrayList<>();
    Map<Integer,Integer> map = new HashMap<>();
    for (int num : nums) {
        if(!map.containsKey(num)){
            map.put(num,1);
        }else{
            map.put(num,map.get(num) + 1);
        }
    }
    for (Map.Entry<Integer, Integer> set : map.entrySet()) {
        if(set.getValue() == 1){
            list.add(set.getKey());
        }
    }
    int[] sums = new int[list.size()];
    for(int i = 0; i < list.size();i++){
        sums[i] = list.get(i);
    }
    return sums;
}
````

### 方法2：异或

题目限定一个数组中只有两个数字出现一次，我们通过异或可以得到两个数字的异或值。

比如int a = {1,3,2,3,2,5}

1⊕ 3⊕ 2⊕ 3⊕ 2⊕ 5 = (3⊕ 3)⊕ (2⊕ 2)⊕ 1⊕ 5 = 4 = 100B

两个不同的数字其中肯定有为1，根据此位为1的可以分为两组

````txt
1 = 001&100 第一组
3 = 011&100 第一组
2 = 010&100 第一组
3 = 011&100 第一组
2 = 010&100 第一组
5 = 101&100 第二组
````

此时就会被分为两组，然后对两组进行异或运算就能获得最后的答案。

#### 代码

````java
public int[] singleNumber(int[] nums) {
    int xor = 0,post = 1,num1 = 0,num2 = 0;
    for (int num : nums) {
        xor ^= num;
    }

    while((xor & 1) == 0){
        xor >>= 1;
        post <<= 1;
    }
    for (int num : nums) {
        if((num & post) == 0){
            num1 ^= num;
        }else{
            num2 ^= num;
        }
    }
    return new int[]{num1,num2};
}
````

