---
title: leetcode1-20
tags:
  - LeetCode
  - Algorithm
categories: Algorithm
date: 2017-02-02 11:43:24
---

# 1. Two Sum

相似题目：15、16、18

给定一个数组，在输入一个指定的目标值后，返回数组中相加等于目标值的两个元素的下标。

可以假定每个输入都有一个解，并且不能重复使用同一个的元素。

<!-- more -->

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1]. 
```

## Python

```python
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        if len(nums)<=1:
            return False
        w_dict = {}
        for i in range(len(nums)):
            if (target - nums[i]) in w_dict:
                return [w_dict[target - nums[i]], i]
            else:
                w_dict[nums[i]] = i
```

## C++

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> hash;
        vector<int> result;
        for(int i=0; i<nums.size(); i++)
        {
            int want = target-nums[i];
            if (hash.find(want)!=hash.end())
            {
                result.push_back(hash[want]);
                result.push_back(i);
                return result;
            }
            else
            hash[nums[i]]=i;
        }
        return result;
    }
};
```

# 2. Add Two Numbers

给定两个非空链表，分别代表两个非负整数。其中的数字按逆序存储，每一个节点都只有一个数字。输入两个这样的链表，返回两数之和，同样用链表表示。

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
```

## Python

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        up = 0
        ans=temp = ListNode(0)
        while l1 or l2 or up:
            v1=v2=0
            if l1:
                v1 = l1.val
                l1 = l1.next
            if l2:
                v2 = l2.val
                l2 = l2.next
            up, val = divmod(v1+v2+up, 10)
            temp.next = ListNode(val)
            temp = temp.next
        return ans.next
```

## C++

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int up = 0;
        ListNode ans(0);
        ListNode *p = &ans;
        while (l1||l2||up)
        {
            int v1=0,v2=0;
            if (l1)
            {
                v1=l1->val;
                l1 = l1->next;
            }
            if (l2)
            {
                v2 = l2->val;
                l2 = l2->next;
            }
            int sum = v1+v2+up;
            up = sum/10;
            p->next = new ListNode(sum%10);
            p = p->next;
        }
        return ans.next;
    }
};
```

# 3.Longest Substring Without Repeating Characters

给定一个字符串，输出它的最长无重复字符子串的长度。

思路是哈希表加上双指针，进行遍历。

## Python

```python
class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        ans = 0
        old_start = 0
        record={}
        for i in range(len(s)):
            if s[i] in record:
                new_start = record[s[i]]+1
                for j in range(old_start,new_start):
                    del record[j]
                old_start = new_start
            ans = max(ans,i-old_start+1)
            record[s[i]]=i
        return ans
```

## C++

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int ans = 0, j=0;
        map<char,int> record;
        for(int i=0;i<s.length();i++)
        {
            if (record.find(s[i])!=record.end())
            {
                int temp = record[s[i]]+1;
                for (int ii=j;ii<temp;ii++)
                {
                    record.erase(record.find(s[ii]));
                }
                j = temp;
            }
            ans = max(ans, i-j+1);
            record[s[i]]=i;
        }
        return ans;
    }
};
```

# 4.Median of Two Sorted Arrays 

给定两个排好序的数组，寻找这两个数组的中位数。例如：

```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
```

```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

思路是按照奇偶性分别进行中位数查询，查询的策略是二分搜索，代码如下。

## Python

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        def binary_search(A, B, k):
            m = len(A)
            n = len(B)
            if m<n:
                return binary_search(B,A,k)
            if n==0:
                return A[k-1]
            if k==1:
                return min(A[0],B[0])
            b = min(n, k/2)
            a = k-b
            if A[a-1]<B[b-1]:
                return binary_search(A[a:],B,k-a)
            elif A[a-1]>B[b-1]:
                return binary_search(A,B[b:],k-b)
            else:
                return A[a-1]
        total = len(nums1)+len(nums2)
        if total%2==0:
            return (binary_search(nums1, nums2, total/2)+\
                 binary_search(nums1, nums2, total/2+1))/2.0
        else:
            return binary_search(nums1, nums2, total/2+1)
```

## C++

```c++
class Solution {
public:
    vector<int> forward_delete(vector<int> A, int k)
    {
        vector<int>::iterator it=A.begin();

        for(int i=0;i<k;i++)
        {
            it = A.erase(it);
        }
        return A;
    }
    double binary_search(vector<int>& A, vector<int>& B,int k)
    {
        int m = int(A.size());
        int n = int (B.size());
        if (m<n)
            return binary_search(B,A,k);
        if (n==0)
            return A[k-1];
        if (k==1)
            return min(A[0],B[0]) ;
        int b = min(k/2,n) ;
        int a = k-b;
        if (A[a-1]<B[b-1])
        {
            vector<int> C = forward_delete(A,a);
            return binary_search(C,B,k-a);
        }
        else if (A[a-1]>B[b-1])
        {
            vector<int> C = forward_delete(B,b);
            return binary_search(A,C,k-b);
        }
        else
            return A[a-1];

    }
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int total = int(nums1.size()+nums2.size());
        if (total%2==0)
        {
            double a = binary_search(nums1,nums2,total/2);
            double b=binary_search(nums1,nums2,total/2+1);
            return (a+b)/2.0;
        }
        else
        {
            return binary_search(nums1,nums2,total/2+1);
        }
    }
};
```

# 5.Longest Palindromic Substring

寻找给点字符串中的最长回文子串。

采用动态规划的方法，从左到右进行回文串判定以及是否为最长判定，如果是最长，就记录此时的left、right位置，最终输出s[left:right].

## Python

```python
class Solution(object):
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        n = len(s)
        dq = [[0 for ii in range(n)] for i in range(n)]
        length = 0
        left =0
        right =0
        for i in range(1,n):
            for j in range(i):
                if s[i]==s[j] and (i-j<=2 or dq[j+1][i-1]==1) :
                    dq[j][i]=1
                    if length<i-j+1:
                        length=i-j+1
                        left=j
                        right=i
        return s[left:right+1]
```

## C++

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        int length=0,left=0,right=0;
        int n=s.length();
        int dq[n][n]={0};
        for(int i=1;i<n;i++)
        {
            for (int j=0;j<i;j++)
            {
                if (s[j]==s[i] && (i-j<=2||dq[j+1][i-1])) 
                {
                    dq[j][i]=1;
                    if (length<i-j+1)
                    {
                        length=i-j+1;
                        left=j;
                        right=i;
                    }
                }
                
            }
        }
        return s.substr(left,right-left+1);
    }
};
```

# 6.ZigZag Conversion

把字符串先写成折线结构，例如字符串"PAYPALISHIRING"：

```
P   A   H   N
A P L S I I G
Y   I   R
```

然后输出横向读取的结果："PAHNAPLSIIGYIR"

## Python

自己想的是，先把字符串按照numRows的长度进行切片，没有的位置补上空格‘ ’，每个切片是折线方向的顺序，然后再把每个切片中第一的元素拼起来，第二个元素拼起来，就完成的横向连接。

```python
class Solution(object):
    def convert(self, s, numRows):
        """
        :type s: str
        :type numRows: int
        :rtype: str
        """
        left=0
        c = []
        if numRows<=1:
            return s
        while left<len(s):
            right = left+numRows
            if right>=len(s):
                c.append(s[left:len(s)]+' '*(right-len(s)))
                break
            else:
                c.append(s[left:right])
                left = right
                right += numRows-2
                if right>=len(s):
                    temp = ' '+s[left:len(s)]+' '*(right-len(s)) +' '
                else:
                    temp = ' '+s[left:right]+' '
                c.append(temp[::-1])
                left = right
        r = ''
        for i in range(numRows):
            r+=''.join([sec[i] for sec in c if sec[i]!=' '])
        return r
```

但是这种方法并不简洁，其实可以直接把原字符串按照从左到右的顺序存储在横向数组中，最后把横向数组的元素拼接即可。

```python
class Solution(object):
    def convert(self, s, numRows):
        """
        :type s: str
        :type numRows: int
        :rtype: str
        """
        if numRows==1:
            return s
        r = [''] * numRows
        i = 0
        while i < len(s):
            for j in range(numRows):
                if i >= len(s):
                    break
                r[j] += s[i]
                i += 1
            for j in range(numRows - 2, 0, -1):
                if i >= len(s):
                    break
                r[j] += s[i]
                i+=1
        ans =''
        for i in range(0,numRows):
            ans += r[i]
        return ans
```

## C++

```c++
class Solution {
public:
    string convert(string s, int numRows) {
        if (numRows == 1) return s;
        string r[numRows];
        int i =0;
        while (i < s.size())
        {
            for(int j=0; j<numRows && i<s.size(); j++) r[j]+=s[i++];
            for(int j=numRows-2;j>0 && i<s.size() ; j--) r[j]+=s[i++];
        }
        string ans="";
        for(i=0; i<numRows;i++)
        {
            ans+=r[i];
        }
        return ans;
        
    }
};
```
# 7.Reverse Integer

把一个整型数进行反转，例如123变成321。

需要注意的点事要能处理负数，同时100变成1而不是001。此外，翻转后的数字不能溢出，要满足有符号32位整型的范围。

## Python

```python
class Solution(object):
    def reverse(self, x):
        """
        :type x: int
        :rtype: int
        """
        c=0
        s=1
        if x<0:
            x=-x
            s=-1
        while x!=0:
            c = 10*c + x%10
            x = x/10
        return c*s if c<2147483648 and c>-2147483648 else 0
```

## C++

```c++
class Solution {
public:
    int reverse(int x) {
        int flag=1,c=0;
        if (isOverflow(x)) return 0;
        if(x<0)
        {
            x=-x;
            flag=-1;
        }
        while(x!=0)
        {
            c=c*10+x%10;
            x=x/10;
        }
        c=c*flag;
        if (c<INT32_MAX && c>INT32_MIN)
            return c ;
        else
            return 0;
    }
private:
    bool isOverflow(int x)
    {
        if(x/1000000000==0)
            return false;
        else if (x==INT32_MIN)
            return true;
        x = abs(x);
        for(int cmp=463847412; cmp!=0; cmp/=10,x/=10)
        {
            if (x%10>cmp%10)
                return true;
            else if (x%10<cmp%10)
                return false;
        }
        return false;
    }
};
```

# 8.String to Integer (atoi)

把字符串格式的数字转换成整型并输出。

需要注意的情况是：字符串前面的空格要去除；数字前面可能有正负号；数字后的其他字符作为截断位置，不需要再往后读取；如果数字超过INT_MAX或INT_MIN，那么输出INT_MAX或者INT_MIN。

## Python

```python
class Solution(object):
    def myAtoi(self, str):
        """
        :type str: str
        :rtype: int
        """
        pos = 0
        flag = 1
        ans=0
        while pos<len(str) and str[pos]==' ': pos+=1
        if pos<len(str) and str[pos]=='-':
            flag=-1
            pos+=1
        elif pos<len(str) and str[pos]=='+':
            pos+=1
        while pos<len(str):
            if str[pos]<='9' and str[pos]>='0':
                ans = ans*10 + ord(str[pos])-ord('0')
                pos+=1
            else:
                break
        ans = flag*ans
        if ans<-2147483648:
            return -2147483648
        elif ans>2147483647:
            return 2147483647
        else:
            return ans
```

## C++

```c++
class Solution {
public:
    int myAtoi(string str) {
        int  flag=1 ,pos =0;
        long long ans = 0;
        while(str[pos]==' '&& str[pos]!='\0') pos++;
        if (str[pos]=='-') flag=-1,pos++;
        else if (str[pos]=='+') pos++;
        for(; str[pos]!='\0';pos++)
        {
            if (str[pos]<='9'&&str[pos]>='0')
            {
                if (flag==1)
                {
                    ans = ans*10 + (str[pos]-'0');
                    if (ans>INT_MAX) return INT_MAX;
                }
                else
                {
                    ans = ans*10 -(str[pos]-'0');
                    if (ans<INT_MIN) return INT_MIN;
                }
            }
            else break;
        }
        return ans;
    }
};
```
# 9.Palindrome Number

给定一个整形数字，判断是否为回文，要求不能申请额外的空间，也就是空间复杂度是O（1）。

因为有空间要求，就不能转成字符串，然后反转看是否相等了。可以每次之比较开头和结尾，这样空间复杂度就是个常数。

## Python

```python
class Solution(object):
    def isPalindrome(self, x):
        """
        :type x: int
        :rtype: bool
        """
        if x<0:
            return False
        len = 1
        while(x/len>=10):
            len*=10
        while(len>1):
            left = x/len
            right = x%10
            if(left!=right):
                return False
            x = (x%len)/10
            len = len/100
        return True
```

## C++

```c++
class Solution {
public:
    bool isPalindrome(int x) {
        if (x<0)
        {
            return false;
        }
        int len = 1;
        while(x/len >= 10)
        {
            len*=10;
        }
        for(; len>1;len=len/100)
        {        
            int left = x/len;
            int right = x%10;
            if(left!=right) return false;
            x=(x%len)/10;
        }
        return true;
        
    }
};
```

# 10.Regular Expression Matching

实现正则表达，“*”表示匹配若干个和他其前面的字符相同的，“.”表示匹配该位置的任意字符。例如：

```
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a*") → true
isMatch("aa", ".*") → true
isMatch("ab", ".*") → true
isMatch("aab", "c*a*b") → true
```

## Python

```python
class Solution(object):
    def isMatch(self, s, p):
        """
        :type s: str
        :type p: str
        :rtype: bool
        """
        if not p:
            return not s
        if(len(p)>1 and p[1]=='*'):
            if(self.isMatch(s,p[2:])):
                return True
            lens=len(s)
            if lens==0:
                return self.isMatch(s,p[2:])
            for i in range(lens):
                if(s[i]!=p[0] and p[0]!='.'):
                    return False
                elif(self.isMatch(s[i+1:],p[2:])): 
                    return True
        else:
            if((not s) or (s[0]!=p[0] and p[0]!='.')):
                return False
            else:
                return self.isMatch(s[1:],p[1:])
        return False
```

## C++

```C++
class Solution {
public:
    bool isMatch(string s, string p){
        const char* a = s.c_str();
        const char* b = p.c_str();
        return isMatch(a,b);
    }
    bool isMatch(const char* a, const char* b) {
        if(a==NULL||b==NULL||*b=='*') return false;
        if(*b=='\0') return *a=='\0';
        if(*(b+1)=='*') {
            int lena = int(strlen(a));
            if (isMatch(a, b + 2)) return true;
            for (int i = 0; i < lena; i++) {
                if (*b != *(a + i) && *b != '.') return false;
                if (isMatch(a + i + 1, b + 2))
                    return true;
            }
        }
        else
        {
            if(*a=='\0')
                return false;
            if(*a==*b||*b=='.')
                return isMatch(a+1,b+1);
            else
                return false;
        }
		return false; //不加这行，leetcode不给编译通过！
    }
};
```

# 11. Container with most water

给定n个坐标，横坐标是1,2,3...n，纵坐标大于等于零，每个坐标向下做垂线，把这些线看做木板，试求两个木板再加上x轴组成的水桶，得到最大的装水容量是多少。

这个我觉得是贪心，从左右两边向中间移动，移动的规则是每次移动较矮的边，因为它限制了当前的容量， 如果移动后能找到更高的，则有可能得到更大的容量。

## Python

```python
class Solution(object):
    def maxArea(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        left=0
        right=len(height)-1;
        contain = 0
        while(left!=right):
            if(height[left]<height[right]):
                temp = (right-left)*height[left]
                if temp>contain:
                    contain = temp
                left+=1   
            else:
                temp = (right-left)*height[right]
                if temp>contain:
                    contain = temp 
                right-=1
        return contain
```

## C++

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;
        int right = int(height.size())-1;
        int contains = 0;
        while(left!=right)
        {
            if (height[left]<height[right])
            {
                int temp = height[left] * (right-left);
                if(temp>contains)
                {
                    contains = temp;
                }
                left++;
            }
            else
            {
                int temp = height[right]*(right-left);
                if(temp>contains)
                {
                    contains=temp;
                }
                right--;
            }
        }
        return contains;
    }
};
```

# 12 Integer to Roman

给定一个整数，转换成罗马数字，输入是1到3999。

I II III IV V VI VII VIII IX X   XI XII XIII XIV XV XVI XVII  XVIII XIX  XX 

1 2 3 4  5  6   7   8   9 10 11 12 13  14  15  16   17   18   19    20 

```
I(1)，V(5)，X(10)，L(50)，C(100)，D(500)，M(1000)
```

这道题不难，但是好麻烦啊，不想写PYTHON版本了。

## C++

```C++
class Solution {
public:
    string intToRoman(int num) {
        string ans = "";
        int M = num/1000;
        for(int i=0; i<M; i++)
            ans+="M";
        num = num%1000;
        if(num/900)
        {
            ans+="CM";
            num = num%900;
        }
        int D = num/500;
        for(int i=0; i<D; i++)
            ans+="D";
        num = num%500;
        if(num/400)
        {
            ans+="CD";
            num = num%400;
        }
        int C = num/100;
        for(int i=0; i<C; i++)
            ans+="C";
        num = num%100;
        if(num/90)
        {
            ans+="XC";
            num = num%90;
        }        
        int L = num/50;
        for(int i=0; i<L; i++)
            ans+="L";
        num = num%50;
        if(num/40)
        {
            ans+="XL";
            num = num%40;
        }  
        int X = num/10;
        for(int i=0; i<X; i++)
            ans+="X";
        num = num%10;
        if(num/9)
        {
            ans+="IX";
            num = num%9;
        }  
        int V = num/5;
        for(int i=0; i<V; i++)
            ans+="V";
        num = num%5;
        if(num/4)
        {
            ans+="IV";
            num = num%4;
        }
        int I = num/1;
        for(int i =0; i<I;i++)
            ans+="I";
        return ans;
    }
};
```

有一个简单的写法，换Python实现一下。

## Python

```python
class Solution(object):
    def intToRoman(self, num):
        """
        :type num: int
        :rtype: str
        """
        ans=''
        one=['','I','II','III','IV','V','VI','VII','VIII','IX']
        ten = ['','X','XX','XXX','XL','L','LX','LXX','LXXX','XC']
        hundred = ['','C','CC','CCC','CD','D','DC','DCC','DCCC','CM']
        thousand = ['','M','MM','MMM']
        rule = [one, ten, hundred, thousand]
        for t in rule:
            ans = t[num%10]+ans
            num = num/10
        return ans
```

# 13. Roman to Integer

上一题的逆过程。

## C++

```c++
class Solution {
public:
    int romanToInt(string s) {
        int sum=0;
        char roman[]={'I', 'V', 'X', 'L', 'C', 'D', 'M'};
        int integer[]={1, 5, 10, 50, 100, 500, 1000};
        std::map<char,int> roman2int; 
        int maplen = sizeof(roman)/sizeof(char);
        for(int i=0; i<maplen; ++i)
        {
            roman2int.insert(std::pair<char,int>(roman[i],integer[i]));
        }
        for(int i=0; i<s.length()-1;i++)
        {
            int present = roman2int[s[i]];
            int next = roman2int[s[i+1]];
            if(present<next)
                sum-=roman2int[s[i]];
            else
                sum+=roman2int[s[i]];
        }
        sum+=roman2int[s[s.length()-1]];
        return sum;
    }
};
```

## Python

```python
class Solution(object):
    def romanToInt(self, s):
        """
        :type s: str
        :rtype: int
        """
        roman = ['I', 'V', 'X', 'L', 'C', 'D', 'M']
        integer = [1, 5, 10, 50, 100, 500, 1000]
        r2i={}
        for i in range(len(integer)):
            r2i[roman[i]]=integer[i]
        sum=0
        for i in range(len(s)-1):
            if(r2i[s[i]]<r2i[s[i+1]]):
                sum-=r2i[s[i]]
            else:
                sum+=r2i[s[i]]
        sum+=r2i[s[-1]]
        return sum
```

# 14. Longest Common Prefix

给定一组string，寻找他们的最长公共前缀。

# 15. 3Sum 

给定一个数组，返回所有满足a+b+c=0的三元组，注意返回的元素彼此不能重复。

简单的思路是结合第一题，对数组中每个数字都进行一次TwoSum，寻找所有其他两个数之和是本数字的相反数，tip是先做好排序，然后当前数字统计好三元组后，下一个数字如果重复就可以跳过。同样，在统计二元组的子函数中，如果下个数字对和当前已经匹配了的数字对相同，也可以跳过，这样就能加快计算。

## Python

```python
class Solution(object):
    def twoSum(self,nums,target):
        L = 0
        R = len(nums)-1
        ans = []
        while L<R:
            temp = nums[L]+nums[R]+target
            if temp==0:
                ans.append([nums[L],nums[R],target])
                L+=1
                R-=1
                while L<R and nums[L-1]==nums[L]:L+=1
                while L<R and nums[R+1]==nums[R]:R-=1
            elif temp>0:
                R-=1
            else:
                L+=1
        return ans
    def threeSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        nums.sort()
        ans = []
        for i, num in enumerate(nums) :
            if i>0 and nums[i] == nums[i-1]: continue
            temp = self.twoSum(nums[i+1:],num)
            ans.extend(temp)
        return ans
```

## C++

```c++
class Solution
{
public:
    vector<vector<int>> threeSum(vector<int>& nums)
    {
        sort(nums.begin(),nums.end());
        vector<vector<int>> results;
        int n = int(nums.size());
        for(int i=0; i<n; i++)
        {
            if(nums[i]==nums[i-1] && i != 0) continue;
            int L = i+1 ;
            int R = n-1;
            while(L<R)
            {
                int ans = nums[L]+nums[R]+nums[i];
                if(ans == 0)
                {
                    vector<int> result;
                    result.push_back(nums[L]);
                    result.push_back(nums[i]);
                    result.push_back(nums[R]);
                    results.push_back(result);
                    L++;
                    R--;
                    while(L<R && nums[L] == nums[L-1]) L++;
                    while(R>L && nums[R] == nums[R+1]) R--;
                }
                else if(ans>0) R=R-1;
                else L=L+1;
            }
        }
        return results;
    }
};
```

# 16 ThreeSumClosest 

给定一个数组和一个目标值，返回数组中三个数的加和，满足这个加和是最接近目标值的。

##C++

```c++
class Solution
{
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(),nums.end());
        int n = int(nums.size());
        int result = nums[n-1]+ nums[n-3]+nums[n-2];
        int diff = nums[n-1]+ nums[n-3]+nums[n-2]-target;
        for (int i=0; i<n; i++)
        {
            if(i>0 && nums[i]==nums[i-1]) continue;
            int L = i+1;
            int R = n-1;
            while(L<R)
            {
                int sum = nums[L]+nums[i]+nums[R];
                int temp = sum-target;
                if (temp==0) return target;
                else if (temp>0) R--;
                else L++;
                if (abs(temp)<abs(diff)) {
                    diff = temp;
                    result = sum;
                }
            }
        }
        return result;
    }
};
```

# 17 Letter Combinations of a Phone Number

给定一个数字字符串如“234”，返回手机按键对应的所有可能的字母组合

## C++

```c++
class Solution {
public:
    unordered_map<char, string> prepare() {
        unordered_map<char, string> dict;
        dict.insert(pair<char, string>('2', "abc"));
        dict.insert(pair<char, string>('3', "def"));
        dict.insert(pair<char, string>('4', "ghi"));
        dict.insert(pair<char, string>('5', "jkl"));
        dict.insert(pair<char, string>('6', "mno"));
        dict.insert(pair<char, string>('7', "pqrs"));
        dict.insert(pair<char, string>('8', "tuv"));
        dict.insert(pair<char, string>('9', "wxyz"));
        return dict;
    }
    vector<string> letterCombinations(string digits) {
        vector<string> results;
        if (digits.length()==0) return results;
        results.push_back("");
        int n = int(digits.length());
        unordered_map<char, string> dict=prepare();
        for (int i=0; i<n; i++) {
            vector<string> temp;
            char num = digits[i];
            for (int j=0; j<results.size();j++) {
                for(int k=0; k<dict[num].length(); k++) {
                    temp.push_back(results[j]+dict[num][k]);
                }
            }
            results = temp;
        }
        return results;
    }
};
```



# 18 4Sum

给定一个数组，和一个目标值，寻找数组中所有满足四个数之和等于目标值的组合并返回。

思路是和2Sum，3Sum类似，只不过遍历的时候用两重循环固定前两个值，然后用双指针遍历查找满足条件的组合，tips是先排序，且重复项跳过，再有就是如果最大的四个加起来还不够，就continue。

## C++

```c++
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target)
    {
        sort(nums.begin(),nums.end());
        vector<vector<int>> results;
        int n = int(nums.size());
        for (int i=0; i<n;i++)
        {
            if(nums[i]==nums[i-1] && i!=0) continue; 
            if (nums[i]+nums[i+1]+nums[i+2]+nums[i+3]>target) break;
            if (nums[i]+nums[n-3]+nums[n-2]+nums[n-1]<target) continue;
            for(int j=i+1; j<n;j++)
            {
                if(nums[j]==nums[j-1] && j!=(i+1)) continue;
                int L = j+1;
                int R = n-1;
                while(L<R)
                {
                    int sum = nums[i]+nums[j]+nums[L]+nums[R];
                    if (sum == target)
                    {
                        int ans[] = {nums[i],nums[j],nums[L],nums[R]};
                        vector<int> result(begin(ans),end(ans));
                        results.push_back(result);
                        L++;
                        R--;
                        while(L<R && nums[L]==nums[L-1]) L++;
                        while(L<R && nums[R]==nums[R+1]) R--;
                    }
                    else if (sum<target) L++;
                    else R--;
                }

            }
        }
        return results;
    }
};
```

# 19 Remove Nth Node from End of List

先统计一下链表长度，然后知道该去除的是第几个节点，然后从链表头开始查，查到要去除的节点就跳过去(把next指针修改成next的next)

## C++

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        vector<ListNode*> store_p;
        vector<int> store_v;
        ListNode* ans = head;
        int num=1;
        while (ans->next!=NULL)
        {
            ans = ans->next;
            num++;
        }
      	//下表是expect的节点要跳过
        int expect = num-n;
        ListNode* temp = head;
        if (expect == 0) return head->next;
        for(int i=1;i<num;i++) {
            if (i!=expect) {
                temp = temp->next;
            }
            else {
                temp->next = temp->next->next;
                break;
            }
        }
        return head;
    }
};
```

# 20 Valid Parentheses 

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

The brackets must close in the correct order, `"()"` and `"()[]{}"` are all valid but `"(]"` and `"([)]"` are not.

## C++

```c++
class Solution {
public:
    bool isValid(string s) {
        stack<char> store ;
        for (char& c : s) {
            switch(c) {
                case '(':
                case '[':
                case '{': store.push(c); break;
                case '}': if (store.empty()||store.top()!='{') return false; else store.pop(); break;
                case ']': if (store.empty()||store.top()!='[') return false; else store.pop(); break;
                case ')': if (store.empty()||store.top()!='(') return false; else store.pop(); break;
                default: ;
            }
        }
        return store.empty();
    }
};
```

