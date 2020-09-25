# 20. 有效的括号

[题目链接](https://leetcode-cn.com/problems/valid-parentheses/)

- 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

  有效字符串需满足：

  左括号必须用相同类型的右括号闭合。
  左括号必须以正确的顺序闭合。
  注意空字符串可被认为是有效字符串。

  示例 1:

  输入: "()"
  输出: true
  示例 2:

  输入: "()[]{}"
  输出: true
  示例 3:

  输入: "(]"
  输出: false
  示例 4:

  输入: "([)]"
  输出: false
  示例 5:

  输入: "{[]}"
  输出: true

## 解题思路

### 方法：栈

利用栈先入后出的特点，刚好与本题括号特点一样，都是最后的左括号先出栈，并且当全部遍历完成后，栈应该为空。

#### 代码

````java
public boolean isValid(String s) {
    if (s.length() % 2 != 0) return false;
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if(c =='(') stack.push(')');
        else if(c == '[') stack.push(']');
        else if(c == '{') stack.push('}');
        else if (stack.isEmpty() ||c != stack.pop()) return false;
    }
    return stack.isEmpty();
}
````

#### 
