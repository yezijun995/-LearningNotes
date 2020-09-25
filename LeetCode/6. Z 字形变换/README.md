# 6. Z 字形变换

[题目链接](https://leetcode-cn.com/problems/zigzag-conversion/)

## 题目要求

- 将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。

  比如输入字符串为 "LEETCODEISHIRING" 行数为 3 时，排列如下：

  ````txt
  L   C   I   R
  E T O E S I I G
  E   D   H   N
  ````


  之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："LCIRETOESIIGEDHN"。

  请你实现这个将字符串进行指定行数变换的函数：

  string convert(string s, int numRows);
  示例 1:

  ````txt
  输入: s = "LEETCODEISHIRING", numRows = 3
  输出: "LCIRETOESIIGEDHN"
  ````


  示例 2:

  ````txt
  输入: s = "LEETCODEISHIRING", numRows = 4
  输出: "LDREOEIIECIHNTSG"解释:
  ````

  ````txt
  L     D     R
  E   O E   I I
  E C   I H   N
  T     S     G
  ````

## 解题思路

### 方法：



numRows=3 

```txt
L   C   I   R           0   4   8     12   
E T O E S I I G         1 3 5 7 9  11 13 15
E   D   H   N           2   6   10    14
```

numRows=4

````txt
L     D     R          0     6       12
E   O E   I I          1   5 7    11 13
E C   I H   N          2 4   8 10    14
T     S     G          3     9       15
````

从上面可以看出是有规律的，当中的间隔都是每行的间隔都是2 * numRows - 2，中间的元素与前一个或是后一个元素的总和都是2 * numRows - 2。

#### 代码

````java
public String convert(String s, int numRows) {
    if(s.isEmpty() ||numRows < 1) return "";
    if(numRows == 1) return s;
    StringBuilder sb = new StringBuilder();
    int index = 0, deviation = 0, count = 0;
    int tmp = 2 * numRows -2;
    for (int i = 0; i < numRows; i++) {
        index = i;
        while (index < s.length()) {
            sb.append(s.charAt(index));
            if (deviation == 0 || deviation == tmp) {
                index += tmp;
            }else {
                if (count % 2 == 0) index = index + tmp - deviation;
                else if (count % 2 != 0) index = index + tmp - (tmp - deviation);
                count++;
            }
        }
        count = 0;
        deviation += 2;
        if (deviation == tmp) deviation = 0;
    }

    return sb.toString();
}
````
