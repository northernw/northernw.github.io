---
title: ZigZag Conversion | Medium
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-23 14:51:08
---
leetCode: [6. ZigZag Conversion](https://leetcode.com/problems/zigzag-conversion/)

## Description
```
The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"

Write the code that will take a string and make this conversion given a number of rows:

string convert(string s, int numRows);
Example 1:

Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
Example 2:

Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:

P     I    N
A   L S  I G
Y A   H R
P     I
```

## Solution

### First
心里想着是有数学规律的解答方案的，奈何数学太渣

Runtime: 56 ms, faster than 26.43% of Java online submissions for ZigZag Conversion.

```
class Solution {
    public String convert(String s, int numRows) {
        if (numRows == 1) {
            return s;
        }
        int n = s.length();
        int cnt = numRows + numRows - 2;
        int numCols = (n / cnt + 1) * (numRows - 1);

        char[][] flag = new char[numRows][numCols];

        int i = 0, j = 0, k = 0;
        while (k < n) {
            flag[i][j] = s.charAt(k);
            k++;
            if (j % (numRows - 1) == 0) {
                if (i == numRows - 1) {
                    i--;
                    j++;
                } else {
                    i++;
                }
            } else {
                i--;
                j++;
            }
        }

        StringBuilder ans = new StringBuilder();
        for (i = 0; i < flag.length; i++) {
            for (j = 0; j < flag[i].length; j++) {
                if (flag[i][j] != '\0') {
                    ans.append(flag[i][j]);
                }
            }
        }

        return ans.toString();

    }
}

```

## Recommended

### Sort By Row[走Z方案]
思路相同，写法可比自己的巧妙多了

34 ms
* goingDown的布尔比较巧妙
* StringBuilder+List，节省空间，也省掉遍历
```
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        List<StringBuilder> rows = new ArrayList<>();
        for (int i = 0; i < Math.min(numRows, s.length()); i++)
            rows.add(new StringBuilder());

        int curRow = 0;
        boolean goingDown = false;

        for (char c : s.toCharArray()) {
            rows.get(curRow).append(c);
            if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
            curRow += goingDown ? 1 : -1;
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) ret.append(row);
        return ret.toString();
    }
}
```

### Visit By Row [找规律]

传说中的数学方案
每个循环的字符个数cycLen = 2*(numRows)-2，k为循环个数
* 0行的字符索引为 cycLen * k, k = 0,1,2...
* numRows-1行的字符索引为 cycLen * k + (numRows - 1), 比0行的索引加（numRows-1）个
* 其他每行，在每个循环中都有2个字符，索引分别为cycLen*k+i和cycLen*(k+1)-i，即cycLen*k+(cycLen-i)

总结起来
* 每行中都有一个索引cycLen * k + i 的字符
* 非0行和numRows-1行，还有一个 cycLen * k + (cycLen - i)的字符
见下方实现

19 ms
j表示那一行内，位于竖着的那些列的字符所在索引
即，[A] [S] [G]
```
 P       I       N
[A]   L [S]   I [G]
 Y  A    H  R
 P       I
```
 
```
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;
        
        int n = s.length();
        int cyclen = numRows * 2 - 2;

        StringBuilder ans = new StringBuilder();
        // i表示每行
        // j表示每个分块
        for(int i = 0; i < numRows; i++){
            for(int j = 0; j+i <n; j += cyclen ){
                ans.append(s.charAt(j+i));
                if(i!=0 && i!= numRows-1 && j+cyclen-i<n){
                    ans.append(s.charAt(j+cyclen-i));
                }
            }
        }
        return ans.toString();
    }
}
```
## Note
