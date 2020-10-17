---
title: leetcode-array
tags:
  - LeetCode
  - Algorithm
mathjax: true
categories:
  - Algorithm
date: 2018-05-22 15:19:37
---

<!-- more -->

# 36 Valid Sudoku

先逐行检验，再逐列检验，最后分块检验

```python
class Solution36:
    def isValidSudoku(self, board):
        """
        :type board: List[List[str]]
        :rtype: bool
        """

        for i in range(9):
            used=[]
            for j in range(9):
                if self.check_used(board[i][j],used):
                    return False
            used=[]
            for j in range(9):
                if self.check_used(board[j][i],used):
                    return False
        for i in range(3):
            for j in range(3):
                used = []
                for row in range(i*3,i*3+3):
                    for col in range(j*3,j*3+3):
                        if self.check_used(board[row][col],used):
                            return False
        return True

    def check_used(self,value,used):
        if value=='.':
            return False
        if value in used:
            return True
        else:
            used.append(value)
            return False
```

# 42 Trapping Rain Water 

直观的解决方案是模拟雨水降落后从低往高升的过程，即从底向上逐层计算每一层能储存的雨水量。

后来又看了下别人的做法，不是从下到上，而是利用highest bar将数组分为两组，从而从左到highest、从右到highest各遍历一次即可，时间复杂度O(n)。

```python
class Solution42:
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        total = 0
        if len(height)<=1:
            return total
        highest = max(height)
        highest_index = height.index(highest)
        peak = 0
        for i in height[:highest_index]:
            if i > peak:
                peak = i
            else:
                total += peak-i

        peak = 0
        for i in height[highest_index+1:][::-1]:
            if i > peak:
                peak = i
            else:
                total += peak-i
        return total
    def trap_bottom2top(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        total = 0
        if len(height) <= 1:
            return total
        highest = max(height)
        for i in range(highest):
            left = self.bar_index(height)
            right = len(height)-self.bar_index(height[::-1])-1
            if right-left<=1:
                return total
            dec = min(height[left], height[right])
            total += (right-left-1)*dec
            for j in height[left+1:right]:
                if j>0:
                    total -= min(j,dec)
            height = [x-dec for x in height]
        return total
    def bar_index(self,bars):
        for index, i in enumerate(bars):
            if i >0:
                return index
        return len(bars)
```

# 48 Rotate Image

先转置，在水平翻转

```python
class Solution48:
    def rotate(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: void Do not return anything, modify matrix in-place instead.
        """
        # transpose
        m = len(matrix)
        n = len(matrix[0])
        for i in range(m):
            for j in range(i+1,n):
                matrix[i][j],matrix[j][i] = matrix[j][i],matrix[i][j]
        # vertical central line
        for i in range(m):
            for j in range(n//2,n):
                matrix[i][j],matrix[i][n-j-1] = matrix[i][n-j-1], matrix[i][j]


```

# 66 plusOne

```python
class Solution66:
    def plusOne(self, digits):
        """
        :type digits: List[int]
        :rtype: List[int]
        """
        ans = digits.copy()
        inc = 1
        for i in range(1,len(digits)+1):
            sum = digits[-i]+inc
            inc = sum//10
            new_last = sum%10
            ans[-i] = new_last
        if inc==1:
            ans.insert(inc,0)
        return ans
```

# 70 Climbing Stairs 

```python
class Solution70:
    def climbStairs(self, n):
        """
        每次可以一步也可以两步
        :type n: int
        :rtype: int
        """
        nm2 = 0
        nm1 = 1
        for i in range(1,n+1):
            temp = nm2+nm1
            nm2=nm1
            nm1=temp
        return nm1
```

