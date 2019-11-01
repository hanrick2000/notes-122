---
description: 手动总结题库
---

# JP Morgan OA Prep

## 输入输出的stdin stdout

要用的时候直接input 里提取。 另外所有input都是string，比如"4 1 7 2"， 所以如果有数学计算，或者数字排序的话 得先split，再convert成int。 有些input是“4，1，2，7\n" 这样的形式， 读取的时候记得把后面的换行符给去掉哟

```python
#转化为列表之后不能直接int(List)
#这是错的：n=int(sys.stdin.readline().strip())

#正确解法：需要对列表内每个数int();使用raw_input()不会读取回车，sys.stdin()会读取回车键
dat=[int(x) for x in input().strip().split()]
#这是如果输入是空格分割的话

#假如，.split(‘,’)
#另外还可以定义分割次数 ‘,’,1

#或者用 
import sys 
 sys.stdin.readline().strip('\n').split()
# 其实strip本身默认是用来去除字符串首位空格的 但也可以用户指定，比如在这里 指定去除最后的\n


#如果输入是 10 5 7 2 3 8 10 3 4 
#代表容量为10，第一件物品体积5，第二件物品体积7... 
c=dat[0]
p=[]
v=[]

m=len(dat)
for i in range(1, m, 2):
    p.append(dat[i])
for j in range(2, m, 2):
    v.append(dat[j])
    

for line in sys.stdin：
     input.append(line)
```

需要读多行的话

```python
import sys
msg = sys.stdin.readlines()
msged = [item.strip('\n').split() for item in msg]
msged = list(msged)
print msged 


input
1 2 3
4 5 6
7 8 9

output
[['1', '2', '3'], ['4', '5', '6'], ['7', '8', '9']]
```

```python
import sys

input = []

for line in sys.stdin:
  input.append(line)
  processed = [item.strip('\n').split() for item in input]
print input

效果和上面一样
```

```python
process = []
for inputaitem in inputa:
  process.append([int(i) for i in inputaitem])
然后这么转化一下
[[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

## DP/Back Tracking/Divide and Conquer

### 1. LeetCode 322： Coin Sum fewest\#

```text
You are given coins of different denominations and a total amount of money amount. 
Write a function to compute the fewest number of coins that you need to make up that amount. 
If that amount of money cannot be made up by any combination of the coins, return -1.

Example 1:

Input: coins = [1, 2, 5], amount = 11
Output: 3 
Explanation: 11 = 5 + 5 + 1
Example 2:

Input: coins = [2], amount = 3
Output: -1
Note:
You may assume that you have an infinite number of each kind of coin.

```

{% code-tabs %}
{% code-tabs-item title="solution 1 Recursion or DFS" %}
```python
class Solution(object):
  def coinChange(self, coins, amount):
    """
    input: int[] coins, int amount
    return: int
    """
    # write your solution here
    import sys 

    if (amount == 0): 
      return 0
	
	# Initialize result 
    res = sys.maxsize 
	
	# Try every coin that has smaller value than V 
    for i in range(0, len(coins)): 
      if (coins[i] <= amount): 
        sub_res = self.coinChange(coins, amount-coins[i]) 

			# Check for INT_MAX to avoid overflow and see if result can minimized 
        if (sub_res != sys.maxsize and sub_res + 1 < res): 
          res = sub_res + 1
    
    return res 

		
class Solution(object):
  def coinChange(self, coins, amount):
    """
    input: int[] coins, int amount
    return: int
    """
    # write your solution here
    if amount == 0 or amount < 0:
        return -1 
    
    if amount in coins:
        return 1
    
    minamount=float('inf')
    
    for i in [coin for coin in coins if coin<=amount]:
      coinamount = 1 + self.coinChange(coins, amount-i)
      minamount=min(minamount, coinamount)
    
    return minamount 

print(coinChange([1,5,10,25],63))
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Solution 2 Memoization 我也不知道算2还是3" %}
```python
class Solution(object):
  def coinChange(self, coins, amount):
    """
    input: int[] coins, int amount
    return: int
    """
    # write your solution here
    knownResults=[0]*(amount+1)
    minCoins = amount
    
    if amount in coins:
      knownResults[amount] = 1
      return 1
    
    elif knownResults[amount] > 0:
      return knownResults[amount]
    
    else:
      for i in [c for c in coins if c <= amount]:
        numCoins = 1 + self.coinChange(coins, amount-i)
        
        if numCoins < minCoins:
          minCoins = numCoins
          knownResults[amount] = minCoins
    
    return minCoins
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Solution 3 Tabulation \(Bottom Up\)" %}
```python
class Solution(object):
  def coinChange(self, coins, amount):
    """
    input: int[] coins, int amount
    return: int
    """
    # write your solution here
    MAX = float('inf')

    container=[0]+amount*[MAX]

    for i in range(1, amount+1):
      for coin in coins:
        if i-coin >= 0: 
          container[i] = min(container[i], 1 + container[i-coin])
    
    if container[amount]==MAX:  #or container[-1]
      return -1

    return container[amount]  #container[-1]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

上面的答案只有第三个可以通过所有的testcase 前两个，第一个有超时问题，第二个也是，而且没有考虑-1

### 变形题 LeetCode 518： Coin Sum \#combination

```text
You are given coins of different denominations and a total amount of money. 
Write a function to compute the number of combinations that make up that amount. 
You may assume that you have infinite number of each kind of coin.
 

Example 1:

Input: amount = 5, coins = [1, 2, 5]
Output: 4
Explanation: there are four ways to make up the amount:
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1
Example 2:

Input: amount = 3, coins = [2]
Output: 0
Explanation: the amount of 3 cannot be made up just with coins of 2.
Example 3:

Input: amount = 10, coins = [10] 
Output: 1
 

Note:

You can assume that

0 <= amount <= 5000
1 <= coin <= 5000
the number of coins is less than 500
the answer is guaranteed to fit into signed 32-bit integer
```

{% code-tabs %}
{% code-tabs-item title="solution 1 Recursion or DFS" %}
```python
def count(S, m, n ): 
 	# S[0...m-1] coins to get sum n 
 	# m is the coin left
 	# n is the value left
 
	# If n is 0 then there is 1 solution (do not include any coin) 
	if (n == 0): 
		return 1

	# If n is less than 0 then no solution exists 这一点需要和面试官argue 因为corner case
	if (n < 0): 
		return 0; 

	# If there are no coins and n is greater than 0, then no solution exist 
	if (m <=0 and n >= 1): 
		return 0

	# count is sum of solutions (i) including S[m-1] (ii) excluding S[m-1] 
	return count( S, m - 1, n ) + count( S, m, n-S[m-1] )
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
DP的关键点在于1\) Overlapping Subproblems 2\) Optimal Substructure

用dp\[i\] 来表示组成i块钱，需要最少的硬币数，那么

1. 第j个硬币我可以选择不拿 这个时候， 硬币数 = dp\[i\]
2. 第j个硬币我可以选择拿 这个时候， 硬币数 = dp\[i - coins\[j\]\] + 1

* 和背包问题不同， 硬币是可以拿任意个
* 对于每一个 dp\[i\] 我们都选择遍历一遍 coin， 不断更新 dp\[i\]
{% endhint %}

DP的做法 [https://github.com/azl397985856/leetcode/blob/master/problems/322.coin-change.md](https://github.com/azl397985856/leetcode/blob/master/problems/322.coin-change.md)

DFS的做法 答案比较接近combination [https://app.laicode.io/app/problem/73](https://app.laicode.io/app/problem/73)



### 类似题 Find sets of numbers that add up to a number 

这里需要注意的是 如果是interview 问面试官clarify这里没有negative number或者0， 也可以问是否sorted，是否有duplicates；如果给的是0，回复什么（其实应该回复0）

[https://www.youtube.com/watch?v=nqlNzOcnCfs](https://www.youtube.com/watch?v=nqlNzOcnCfs)  
  
  


### 2. LeetCode 64: Minimum Path Sum

[https://blog.csdn.net/happyaaaaaaaaaaa/article/details/51546526](https://blog.csdn.net/happyaaaaaaaaaaa/article/details/51546526) 这是java

[https://www.jianshu.com/p/19a57bd9f589](https://www.jianshu.com/p/19a57bd9f589) 这才是python

[https://blog.csdn.net/HappyRocking/article/details/85245979](https://blog.csdn.net/HappyRocking/article/details/85245979) 还有这才是python

[https://leetcode-cn.com/problems/minimum-path-sum/solution/](https://leetcode-cn.com/problems/minimum-path-sum/solution/) 

```text
Given a m x n grid filled with non-negative numbers, 
find a path from top left to bottom right which minimizes the sum of 
all numbers along its path. You can only move either down or right at any point in time.
Input: [
        [5, 1, 2, 4],
        [4, 1, 0, 1],
        [0, 3, 7, 6]
           ]    
Output: 14
```

{% code-tabs %}
{% code-tabs-item title="Bottom Up" %}
```python
class Solution:
    def minPathSum(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        n = len(grid)
        m = len(grid[0])
        
        for i in range(1,n):
            grid[i][0] + = grid[i-1][0]    #首先需要寻找左边界各点的路径总和
    
        for j in range(1,m):
            grid[0][j] + = grid[0][j-1]   #寻找上边界各点的路径总和
 
        for i in range(1,n):
            for j in range(1,m):
                grid[i][j] = min(grid[i-1][j] , grid[i][j-1]) + grid[i][j] 
                 #以边界处为依据一步步推出内部个点的路径总和

        return grid[n-1][m-1] #or grid[-1][-1]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 类似题LeetCode 62: Unique Path Sum

[https://blog.csdn.net/HappyRocking/article/details/85118081](https://blog.csdn.net/HappyRocking/article/details/85118081)

### 类似题LeetCode 63: Unique Path Sum

[https://blog.csdn.net/HappyRocking/article/details/85160326](https://blog.csdn.net/HappyRocking/article/details/85160326)

### 3. Knapsack problem \[40min\]

题目和答案 [https://www.geeksforgeeks.org/0-1-knapsack-problem-dp-10/](https://www.geeksforgeeks.org/0-1-knapsack-problem-dp-10/)

给n个物品 每一个物品有他的price和体积 问在购物车的size之内能达到的最大的value是多少

{% code-tabs %}
{% code-tabs-item title="DFS" %}
```python
def knapSack(W , wt , val , n): 
  
    # Base Case 
    if n == 0 or W == 0 : 
        return 0
  
    # If weight of the nth item is more than Knapsack of capacity 
    # W, then this item cannot be included in the optimal solution 
    if (wt[n-1] > W): 
        return knapSack(W , wt , val , n-1) 
  
    # return the maximum of two cases: 
    # (1) nth item included 
    # (2) not included 
    else: 
        return max(val[n-1] + knapSack(W-wt[n-1] , wt , val , n-1), 
                   knapSack(W , wt , val , n-1)) 
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Memoization 待确认" %}
```python
def KS(n, C):
    if arr[n][C]!=undefined:
        return arr[n][C]
    
    if n == 0 or C == 0:
        result = 0
    
    elif w[n] > C:
        result = KS(n-1, C)
    
    else:
        tmp1 = KS(n-1, C)
        tmp2 = v[n]+KS(n-1, C-w[n])
        result = max(tmp1, tmp2)
    
    arr[n][C] = result
    
    return result
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Bottom Up" %}
```python
def knapSack(W, wt, val, n): 
    K = [[0 for x in range(W+1)] for x in range(n+1)] 
  
    # Build table K[][] in bottom up manner 
    for i in range(n+1): 
        for w in range(W+1): 
            if i==0 or w==0: 
                K[i][w] = 0
            elif wt[i-1] <= w: 
                K[i][w] = max(val[i-1] + K[i-1][w-wt[i-1]],  K[i-1][w]) 
            else: 
                K[i][w] = K[i-1][w] 
  
    return K[n][W] #K[-1][-1]


x = np.array([int(i) for i in x.split()]).reshape(5,2)
L = x[0][0]
N = x[0][1]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

如果是要求恰好装满背包，那么在初始化时除了 f\[0\]为 0 其它 f\[1..W\]均设为-∞，这样就可以保证最终得到的 f\[N\]是一种恰好装满背包的最优解。如果并没有要求必须把背包装满，而是只希望价格尽量大，初始化时应该将 f\[0..W\]全部设为 0。  
 为什么呢?可以这样理解:初始化的 f 数组事实上就是在没有任何物品可以放入 背包时的合法状态。如果要求背包恰好装满，那么此时只有容量为 0 的背包可能被价值为 0 的 nothing“恰好装满”，其它容量的背包均没有合法的解，属于未定义的状态，它们的值就都应该是-∞了。如果背包并非必须被装满，那么任何 容量的背包都有一个合法解“什么都不装”，这个解的价值为 0，所以初始时状态的值也就全部为 0 了。  


#### 复杂度优化

时间和空间复杂度均为 O\(N\*W\)，其中时间复杂度基本已经不能再优 化了，但空间复杂度却可以优化到 O\(W\)。  
先考虑上面讲的基本思路如何实现，肯定是有一个主循环 i=1..N，每次算出来二维数组f\[i\]\[0..W\]的所有值。那么，如果只用一个数组 f\[0..W\]，能不能保证第 i 次循环结束后 f\[w\]中表示的就是我们定义的状态 f\[i\]\[w\]呢?f\[i\]\[w\]是由 f\[i-1\]\[w\]和 f\[i-1\]\[w-c\[i\]\]两个子问题递推而来，能否保证在推 f\[i\]\[w\]时\(也 即在第 i 次主循环中推 f\[w\]时\)能够得到 f\[i-1\]\[w\]和 f\[i-1\]\[w-w\[i\]\]的值呢? 事实上，这要求在每次主循环中我们以 v=V..0 的顺序推 f\[w\]，这样才能保证推 f\[v\]时 f\[v-w\[i\]\]保存的是状态 f\[i-1\]\[w-w\[i\]\]的值。改进后的代码如下：

```python
import numpy as np

def solve2(vlist,wlist,totalWeight,totalLength):
    resArr = np.zeros((totalWeight)+1,dtype=np.int32)
    for i in range(1,totalLength+1):
        for j in range(totalWeight,0,-1):
            if wlist[i] <= j:
                resArr[j] = max(resArr[j], resArr[j-wlist[i]]+vlist[i])
    return resArr[-1]

if __name__ == '__main__':
    v = [0,60,100,120]
    w = [0,10,20,30]
    weight = 50
    n = 3
    result = solve2(v,w,weight,n)
    print(result)
```

### 4. LeetCode17: Letter Combinations of a Phone Number \[90min\]

Back Tracking

[https://blog.csdn.net/puqutogether/article/details/45574903](https://blog.csdn.net/puqutogether/article/details/45574903)

### 5. LeetCode53: Find Max Sum Sub Sequence \[20min\]

```text
Given an integer array nums, find the contiguous subarray 
(containing at least one number) which has the largest sum and return its sum.

Example:

Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
Follow up:

If you have figured out the O(n) solution, 
try coding another solution using the divide and conquer approach, 
which is more subtle.
```

[https://www.youtube.com/watch?v=86CQq3pKSUw](https://www.youtube.com/watch?v=86CQq3pKSUw)

1. Brute Force Approach, check from the first,  $$O(n^{2})$$ 
2. Kadane’s Algorithm \(ideal one\), what’s the maximum subarray that ends in this index. $$O(n^{2})$$ , or run with a linear time$$O(n)$$ Compare the ‘current subarray’ with that ‘combining the previous maximum subarrays with this current’. 
3. Prefix Sum
4. 通过max-min优化Prefix Sum$$O(n)$$   
   定义函数 `S(i)` ，它的功能是计算以 `0（包括 0）`开始加到 `i（包括 i）`的值。

   那么 `S(j) - S(i - 1)` 就等于 从 `i` 开始（包括 i）加到 `j`（包括 j）的值。

   我们进一步分析，实际上我们只需要遍历一次计算出所有的 `S(i)`, 其中 `i = 0,1,2....,n-1。` 然后我们再减去之前的 `S(k)`,其中 `k = 0，1，i - 1`，中的最小值即可。 因此我们需要 用一个变量来维护这个最小值，还需要一个变量维护最大值。

#### 2种思路 max-min sum或者DP

```python
class Solution(object):
  def largestSum(self, array):
    """
    input: int[] array
    return: int
    """
    # write your solution here
    n = len(array)
    maxSum = array[0]
    minSum = sum = 0
    for i in range(n):
      sum += array[i]
      maxSum = max(maxSum, sum - minSum)
      minSum = min(minSum, sum)
            
    return maxSum

num = sys.stdin.readline().strip('\n').split()
dat = [int(n) for n in num]

print(largestSum(dat))
```

```python
class Solution:
    def maxSubArray(self, nums: List[int]) :
        max_sum_ending_curr_index = max_sum = nums[0]
        for i in range(1, len(nums)):
            max_sum_ending_curr_index = max(max_sum_ending_curr_index + nums[i], nums[i])
            max_sum = max(max_sum_ending_curr_index, max_sum)
            
        return max_sum

num = sys.stdin.readline().strip('\n').split()
dat = [int(n) for n in num]

print(largestSum(dat))
```

[https://github.com/azl397985856/leetcode/blob/master/problems/53.maximum-sum-subarray-cn.md](https://github.com/azl397985856/leetcode/blob/master/problems/53.maximum-sum-subarray-cn.md)

给一个string 比如说\[1,2,3,4\]的话，他给你的input是'1，2，3，4\n'

把string分割再变成list 注意注意 

### 类似题 Maximum Product Subarray

### 6. Reverse and add until a palindrome

题目和答案 [https://www.pd4cs.org/reverse-and-add-until-a-palindrome/](https://www.pd4cs.org/reverse-and-add-until-a-palindrome/)

```text
Given a positive integer, reverse the digits and add the resulting number to the original number. 
How many times does one have to repeat the process until one gets a palindrome? 
A palindrome is a number whose digits are the same when read left to right 
and right to left. 

For example, for 5280, we get a palindrome after 3 steps (each step reverses the string and add both)

Step 1: 5280 + 0825 = 6105

Step 2: 6105 + 5016 = 11121

Step 3: 11121 + 12111 = 23232

If we start executing the described step over and over again and we generate a palindrome, the algorithm terminates.  
However, if we don’t get a palindrome after 100 steps, after a 1000 steps, after 10,000 steps, what can we conclude?

We can stop and say “we have not generated a palindrome after 10000 iterations” 
but we cannot make any claim that a palindrome cannot be found later.  
Indeed, there exists no algorithm that generates the answer “no palindrome can be generated.”
```

```python
def reversDigits(num): 
    rev_num=0
    while (num > 0): 
        rev_num = rev_num*10 + num%10
        num = num/10
    return rev_num 
  
# Function to check whether the number is palindrome or not 
def isPalindrome(num): 
    return (reversDigits(num) == num) 
  
# Reverse and Add Function 
def ReverseandAdd(num): 
    rev_num = 0
    steps = 0
    
    while (steps <= 1000): 
        # Reversing the digits of the number 
        steps += 1
        rev_num = reversDigits(num) 
  
        # Adding the reversed number with the original 
        num = num + rev_num 
  
        # Checking whether the number is palindrome or not 
        if(isPalindrome(num)): 
            print steps, num 
            break
            
        else: 
            if (num > 4294967295): 
                print "No palindrome exist"

ReverseandAdd(168)
```

### 7. Reverse until Even

![](https://cdn.mathpix.com/snip/images/i6aL824ccsmHAGF05wUgekz-UF_-EGSjKBInmJd2DKQ.original.fullsize.png)

```python
def reversDigits(num): 
    rev_num=0
    while (num > 0): 
        rev_num = rev_num*10 + num%10
        num = num/10
    return rev_num 
  
# Function to check whether the number is palindrome or not 
def isEven(num): 
    while (num > 0): 
        digit = num%10
        num = (num-digit)//10
        if (digit %2!= 0):
          return False
    
    return True
  
# Reverse and Add Function 
def ReverseandAdd(num): 
    rev_num = 0
    steps = 0
    
    while (steps <= 1000): 
        # Reversing the digits of the number 
        steps += 1
        rev_num = reversDigits(num) 
  
        # Adding the reversed number with the original 
        num = num + rev_num 
  
        # Checking whether the number is palindrome or not 
        if(isEven(num)): 
            print steps, num 
            break
            
        else: 
            if (num > 4294967295): 
                print "No palindrome exist"

ReverseandAdd(168)
```

### 8. Balanced Smileys \[90min\]

[https://gist.github.com/mstepniowski/4660602](https://gist.github.com/mstepniowski/4660602)

Follow Up: 1）你的代码的复杂度； 2）Best case和worest case的复杂度; 3\) 这个问题能不能用dynamic programming来解决； 4）你的代码对于包含number的message work不  
30秒准备，录5分钟内的视频回答  


## 运用数据结构性质的问题

### 1. K largest/K smallest

**soln1:** sort   O\(nlogn\)

**soln2:** quick select O\(kn\)

**Soln3:** 维护一个小根堆  Space O\(n\)

Step 1: Heapify all elements    O\(n\)

Step 2: Call pop\(\) k times to get the k smallest elements. O\(klogn\)

Time Complexity Total: O\(n+klogn\)

```python
import heapq

def kSmallest(array, k):
    if not array:
        return []
    res = []
    heapq.heapify(array)
    for i in range(min(k,len(array))):
        res.append(heapq.heappop(array))
    return res
```

**Soln4:** 维护一个大根堆  Space O\(k\)

Step 1: Heapify the first k elements to form a max-heap of size k     O\(k\)

Step 2: Iterate over the remaining n-k elements one by one. 

Compare with the largest element of the previous smallest k candidates. 

case 1: new element&gt;=top: ignore

case 2: new element&lt;top: update \(top -&gt;new element\)              O\(\(n-k\)logk\)

Total O\(k+\(n-k\)logk\)

```python
def kSmallest2(array, k):
    if not array:
        return array
    if k>=len(array):
        return array
    res = [-elem for elem in array[0:k]]
    heapq.heapify(res)
    for i in range(k,len(array)):
        if array[i] < -res[0]
            heapq.heappop(res)
            heapq.heappush(res, -array[i])
    return [-elem for elem in res]
```

|  | O\(n+klogn\) | O\(k+\(n-k\)logk\) |
| :--- | :--- | :--- |
| k&lt;&lt;n | O\(c\*n\) | O\(nlogk\) |
| k~~n | O\(nlogn\) | O\(n\) |

## 

### 2. Merge K sorted array

Step 1: Create a min heap, put the first element of each array into the heap

Step 2: Each time pop an element from the heap, and then push the next element into the heap. 

```python
def mergek(arrays):
    if not arrays:
        return None
    heap = []
    for i in range (len()):
        if len(arrays[i]):
            heap.append((arrays[i][0], i, 0))
        heapq.heapify(heap)
        result=[]
        
        while heap:
            val, index_array, index_element = heapq.heappop(heap)
            result.append(val)
            if index_element+1<len(arrays[index_array]):
                heapq.heappush(heap, (arrays[index_array][index_element+1], 
                index_array, index_element+1))
                
        return result
```

Time: O\(2K+nlogk\)

k读入、k来heapify、nlogk  


### 3. LeetCode20：Valid Parenthesis

常用解法：stack存左括号 遇到右括号和栈顶判断一下是否匹配

[解析](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode%E7%AC%AC20%E5%8F%B7%E9%97%AE%E9%A2%98%EF%BC%9A%E6%9C%89%E6%95%88%E7%9A%84%E6%8B%AC%E5%8F%B7.md) 

```python
def isValid(seq):
    left_bracket = []
    matching_bracket = {'{':'}', '[':']', '(': ')'}
    for b in brackets:
        if b in matching_bracket:
            left_bracket.append(b)
        elif not left_bracket or matching_bracket[left_bracket[-1]]!=b:
            return False
        else:
            left_bracket.pop()
        return not reft_bracket
```

### 4. Count Value Frequency

```python
def words_to_frequencies (words):
    myDict={}
    for word in words:
        if word in myDict:
            myDict[word]+=1
        else:
            myDict[word]=1
    return myDict
```

[https://www.geeksforgeeks.org/count-frequencies-elements-array-o1-extra-space-time/](https://www.geeksforgeeks.org/count-frequencies-elements-array-o1-extra-space-time/)

[https://www.geeksforgeeks.org/counting-frequencies-of-array-elements/](https://www.geeksforgeeks.org/counting-frequencies-of-array-elements/)

[https://www.geeksforgeeks.org/count-number-of-occurrences-or-frequency-in-a-sorted-array/](https://www.geeksforgeeks.org/count-number-of-occurrences-or-frequency-in-a-sorted-array/)  
  


### 5. LeetCode202：Happy Number

```text
Write an algorithm to determine if a number is "happy".
A happy number is a number defined by the following process: 
Starting with any positive integer, replace the number by the sum of the squares of its digits, 
and repeat the process until the number equals 1 (where it will stay), 
or it loops endlessly in a cycle which does not include 1. Those numbers for which this process 
ends in 1 are happy numbers.

Example: 

Input: 19

Output: true

Explanation: 
1^2 + 9^2 = 82
8^2 + 2^2 = 68
6^2 + 8^2 = 100
1^2 + 0^2 + 0^2 = 1

Follow UP：5min, 30sec preparation
If happy numbers are generally prime (19, 79, 239), should it always end in a 9? 
Can u think of a happy number that doesn't end in 9? 

1
```

```python
def isHappy(n):
	seen = []
	while n != 1:
		n = sum(int(i) ** 2 for i in str(n))
		if n in seen:
			return False
		else:
			seen.append(n)
	else:
		return True
```

通过分析该问题，利用一些分析结果，可以对算法进行优化。比如利用10以内的happy number只有1和7，或者先求出100以内的所有happy number等。

代码二

```text
class Solution(object): 
    
 def isHappy(self, n): 
    """ :type n: int 
    :rtype: bool """ 
    
    happySet = set([1, 7, 10, 13, 19, 23, 28, 31, 32, 44, 49, 68, 70, 79, 82, 86, 91, 94, 97])
    
    while n>99:
        n = sum([int(x) * int(x) for x in list(str(n))])
    
    return n in happySet
```

三种解法[https://www.cnblogs.com/grandyang/p/4447233.html  
](https://www.cnblogs.com/grandyang/p/4447233.html)常见解法 用 HashSet 来记录所有出现过的数字，然后每出现一个新数字，在 HashSet 中查找看是否存在，若不存在则加入表中，若存在则跳出循环，并且判断此数是否为1，若为1返回true，不为1返回false

> From 1point3acre
>
> 在真实面试的时候，如果遇到这道题目，很多刚开始刷题的求职者上来就说想实现code，把答案写出来！ Hold on! Hold on!  
>   
> 推荐的解题逻辑：  
>   
> 1. 解读题目的含义与讲讲函数signature输入和输出是什么  
> 2. 讲解算法（非实现\)，依据算法需求选择数据结构  
> 3. 代码实现  
> 4. 找测试数据，验证code正确性  
> 5.分析算法复杂度与空间复杂复杂度  
>   
> 这里面涉及编程语言考点（java为例）  
>
>
> * hashset /treeset的区别；
> * 为什么选择hashset？
> * java最大的integer是多大？
> * 如何估计2^31-1 的大小？
> * 补码和反码 
>
> 关于这道题目背景与很多求职者交流情况  
>   
> 1.这道题目Uber曾经面试过，题目看起来很简单，但是全部都能答对不容易。这道题我问了大概50个人分析时间复杂度，没见过能分析特别明白的。  
>   
> 2. 依照这个题目，引申问了很多java的知识点，70%的人或多或少都有不会的。比如估计2^31-1 这个数的大小吧，很多转专业的人不知道在计算机里面有个很重要的估计方法2^10 = 1000，很能反映出求职者的基本素质。Apple曾经面试过一道题：2的最小多少次幂大于10万，对2^16= 65536 这种数字的记忆能反映出码农的基本素质。  
>   
> 3.大部分同学并没有建立起依据算法需求选择数据结构的逻辑链条：为什么要选择Hashset 而不选择treeset？为什么选择hashset 而不用hashmap？对基本数据结构掌握不牢固，将算法和数据结构混为一谈，是转专业同学的大问题。  
>   
> 4. 一道题目其实可以引申出很多计算机基本知识点，补码和反码什么的，overflow，这些都应该在刷题时候好好查一查，否则容易阴沟里翻船

\(unordered\_set\) O\(1\)

难点在于判断是否进入死循环，这里可以用 unordered\_set 来存之前已经算出过的数字。

计算每位数字的平方和 结果为1，则返回True 结果已出现过，说明陷入死循环，返回False 其他情况则重新计算每位数字的平方和，并把结果加入set 

时间复杂度 O\(1\)： 

\(1\) 首先看总共需要遍历多少个数，所有遍历的数最终会进入循环。进入环前会经过几次变换操作，每次数字 n 最多变成 81logn，所以大于1000的数很快会缩小\(n&gt;81logn\)， $$2^{31}-1$$ 操作四五次就会变到1000以内。考虑环内最大也是999，变化后为243，不会跳出环外，所以环内最多有1000个数。因此，环外有四五个数，环内有常数k个数。  
 \(2\) 由于用哈系表判重，所以每个数最多被遍历一次，总计算次数小于1000，所以时间复杂度是 O\(1\)。

空间复杂度 O\(n\)： 额外占用空间的是 unordered\_set。

作者：extrovert 链接: [https://www.acwing.com/solution/LeetCode/content/284/](https://www.acwing.com/solution/LeetCode/content/284/) 

### 类比题 LeetCode 141: Linked List Cycle

{% embed url="https://leetcode.com/problems/linked-list-cycle/" %}

上道题的环解Floyd Cycle detection algorithm. I believe that many people have seen this in the Linked List Cycle detection problem. The following is my code:

```python
class Solution(object):
    def isHappy(self, n):
        """
        :type n: int
        :rtype: bool
        """
        slow = fast = n
        while True:
            slow = self.squareSum(slow)
            fast = self.squareSum(fast)
            fast = self.squareSum(fast)
            if slow == fast:
                break
        return slow == 1

    def squareSum(self, n):
        sum = 0
        while(n>0):
            tmp = n % 10
            sum += tmp * tmp
            n /= 10
        return sum
```

把sum的值看成是一个链表，那么问题转换成链表是否有环。 用快慢指针判断链表中是否有环，slow和fast最后一定会收敛到某个数字。

时间复杂度 O\(1\)：   
\(1\) 首先看总共需要遍历多少个数，所有遍历的数最终会进入循环。进入环前会经过几次变换操作，每次数字 n 最多变成 81logn，所以大于1000的数很快会缩小\(n&gt;81logn\)， $$2^{31}-1$$ 操作四五次就会变到1000以内。考虑环内最大也是999，变化后为243，不会跳出环外，所以环内最多有1000个数。因此，环外有四五个数，环内有常数k个数。  
 \(2\) 然后看每个数最多遍历次数。快指针最多将每个数遍历2次就会和慢指针相遇。  
 \(3\) 每个数最多遍历3次，所以总计算次数小于3000，时间复杂度是 O\(1\)。

空间复杂度 O\(1\)：没有用额外的空间。

作者：extrovert 链接：[https://www.acwing.com/solution/LeetCode/content/284/](https://www.acwing.com/solution/LeetCode/content/284/) 

## 单纯的Array或字符串操作

### 1. 找出一个数组中最大的两数差值

```text
Given an array of numbers, write an algorithm to find the maximum difference 
between any two elements in the array.

Example:

Int [] a = {2, 8, 1, 6, 10, 4}
Output: Maximum difference= 9 (elements are 1 and 10)
Int [] b = {10, 30, 5, 16, 19};
Output:: Maximum difference= 25 (elements are 30 and 5)
```

```python
def maxdif(arr):
  maxnum=float('-inf')
  minnum=float('inf')

  for i in arr:
    maxnum=max(i, maxnum)
    #print maxnum
    minnum=min(i, minnum)
    #print minnum
  
  difference=maxnum-minnum

  return difference


import sys 
inputarr=sys.stdin.readline().strip('\n').split(',')
data = [int(x) for x in inputarr]

print(maxdif(data))
```

```text
larger element appears after the smaller number
```

```python
def maxDiff(arr, arr_size): 
    max_diff = arr[1] - arr[0] 
    min_element = arr[0] 
      
    for i in range( 1, arr_size ): 
        if (arr[i] - min_element > max_diff): 
            max_diff = arr[i] - min_element 
      
        if (arr[i] < min_element): 
            min_element = arr[i] 
    return max_diff 
```

### 2. Beautiful String \[20min\]

```text
Given a string s, little Johnny defined the beauty of the string 
as the sum of the beauty of the letters in it.
The beauty of each letter is an integer between 1 and 26, inclusive, 
and no two letters have the same beauty. Johnny doesn’t care about 
whether letters are uppercase or lowercase, so that doesn’t affect 
the beauty of a letter. (Uppercase ‘F’ is exactly as beautiful as 
lowercase ‘f’, for example.)
You’re a student writing a report on the youth of this famous 
hacker. You found the string that Johnny considered most beautiful. 
What is the maximum possible beauty of this string?
```

```python
import numpy as np

def beauty_string(x):

    x = np.array(list(x.lower())) 
    x_unique = np.unique(x) 
    count = []
    
    for i in range(len(x_unique)): 
        x_sum = np.sum(x == x_unique[i])
        count.append(x_sum)
    
    count=sorted(count, reverse=True)
    letter_beauty = [n for n in range(26,26-len(x_unique),-1)]
    result = np.sum(np.multiply(count,letter_beauty))
    
    return result

import sys

data=sys.stdin.readline().strip('\n')

print beauty_string(data)
```

```text
import sys
import collections
import string

for line in open(sys.argv[1]).readlines()[1:]:
  line = line.lower()
  l = collections.Counter(line)

  add = 26
  beauty = 0
  for key,value in l.most_common():
    if key in string.letters[:26]:
      beauty += value*add
      add -= 1

  print beauty
```

```text

```

### 3. LeetCode 38: Count and Say 

```text
Given a sequence of number: 1, 11, 21, 1211, 111221, …
The rule of generating the number in the sequence is as follow:
1 is "one 1" so 11.
11 is "two 1s" so 21.
21 is "one 2 followed by one 1" so 1211.
Find the nth number in this sequence.

Assumptions:
n starts from 1, the first number is "1", the second number is "11"
n is smaller than 30
```

```python
import numpy as np
def count_number(vector):
    vector_unique = [vector[0]]
   
    for i in range(1,len(vector)):
        if vector[i - 1] != vector[i]:
            vector_unique.append(vector[i])
    
    n = len(vector_unique)
    count = []
    
    for i in range(n):
        count_i = sum(np.array(vector) == vector_unique[i])
        count.append(count_i)
        count.append(vector_unique[i])
    
    count = " ".join(str(i) for i in count)
    
    return count

import sys
raw = sys.stdin.readline().strip('\n').split()
data = [int(i) for i in raw]
print count_number(data)
```

```text
def countAndSay(self, n):
        """
        :type n: int
        :rtype: str
        """
        res = "1"
        for i in range(n - 1):
            prev = res[0]
            count = 1
            ans = ""
            for j in range(1, len(res)):
                cur = res[j]
                if prev != cur:
                    ans = ans + str(count) + str(prev)
                    prev = cur
                    count = 0
                count += 1
            res = ans + str(count) + str(prev)
        return res
```

```python
class Solution(object):
    def countAndSay(self, n):
        """
        :type n: int
        :rtype: str
        """
        res = '1'
        for i in xrange(n-1):
            res = ''.join([str(len(list(group))) + digit for digit, group in itertools.groupby(res)])
        return res
```

### 类似题 Compres String

上一题的类似题 [https://www.cnblogs.com/grandyang/p/8742564.html](https://www.cnblogs.com/grandyang/p/8742564.html) 这是java

```text
Given a string, replace adjacent, repeated characters with the character followed by the
 number of repeated occurrences. If the character does not has any adjacent, repeated 
 occurrences, it is not changed.

Assumptions
The string is not null
The characters used in the original string are guaranteed to be ‘a’ - ‘z’

Examples
“abbcccdeee” → “ab2c3de3”
```

### 5. LeetCode121~123: 股票买卖

最后考的是maximum subarray 不是这些

[\[LC121\]](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode%E7%AC%AC121%E5%8F%B7%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B9%B0%E5%8D%96%E8%82%A1%E7%A5%A8%E7%9A%84%E6%9C%80%E4%BD%B3%E6%97%B6%E6%9C%BA.md)  
[\[LC122\]](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode%E7%AC%AC122%E5%8F%B7%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B9%B0%E5%8D%96%E8%82%A1%E7%A5%A8%E7%9A%84%E6%9C%80%E4%BD%B3%E6%97%B6%E6%9C%BAII.md)  
[\[LC123\]](https://github.com/MisterBooo/LeetCodeAnimation/blob/master/notes/LeetCode%E7%AC%AC123%E5%8F%B7%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B9%B0%E5%8D%96%E8%82%A1%E7%A5%A8%E7%9A%84%E6%9C%80%E4%BD%B3%E6%97%B6%E6%9C%BAIII.md)  


### 6. LeetCode 315 Count of Smaller Numbers After Self

number of inversions   


Given                 A: \["Carrot", "Cabbage", "Fish", "Meat"\], 

New Customers B: \["Cabbage", "Fish", "Meat", "Carrot"\] 

               C: \["Meat", "Fish", "Cabbage", "Carrot"\]

Return customer names sorted by number of inversions. In case of a tie, sort customer by name  


inversion的定义就是given i, j, if Si &gt; Sj but i &lt; j, \(S, S\[j\]\)就是一对inversion pair  


4,3,2,1 在这四个数里面分别index是0,1,2,3

inversion 就是每个index都和后面数字比 只要比后面大就是inversion pair

\[4,3,2,1\] 就是 \[4,3\], \[4,2\], \[4,1\], \[3,2\], \[3,1\], \[2,1\] 6个  


比较接近LeetCode493. Reverse Pairs

```python
def reversePairs(self, nums):
  return sum([nums[j] > nums[i] for i in range(len(nums)) for j in range(0 , i)])

input  
sortlist = sorted ([reversePairs(input), inverse=True)
```

```python
class Solution(object):
    def __init__(self):
        self.cnt = 0
    def reversePairs(self, nums):
        def msort(lst):
            # merge sort body
            L = len(lst)
            if L <= 1:                          # base case
                return lst
            else:                               # recursive case
                return merger(msort(lst[:int(L/2)]), msort(lst[int(L/2):]))
        def merger(left, right):
            # merger
            l, r = 0, 0                         # increase l and r iteratively
            while l < len(left) and r < len(right):
                if left[l] <= right[r]:
                    l += 1
                else:
                    self.cnt += len(left)-l     # add here
                    r += 1
            return sorted(left+right)           # I can't avoid TLE without timsort...

        msort(nums)
        return self.cnt
    
```

1. use the first line to get the weights of each item, for instance, 3, 2, 1; 3 pairs 
2. store those weights in a dict, so the next time meet abc from the following lines, and use this dict to get the weights of the rest of the lines
3. once I converted all the words to numbers, I can get inversion pairs each line in the reverse pairs a function
4. compare the inversion pairs and output the 2 inversion pars that have the largest inversion

{% embed url="https://zhengyang2015.gitbooks.io/lintcode/reverse\_pairs\_532.html" %}

{% embed url="https://leetcode.com/problems/reverse-pairs/discuss/97268/general-principles-behind-problems-similar-to-reverse-pairs" %}



### 换汤不换药再考一次

Implement Collaborative Filtering的算法来进行推荐

A Recommender System is capable of predicting the preferences of an active user for a set of items. For example, an online store can suggest a product to the shopper based on a history of purchases or page views.

One of the traditional approaches to construct such a system is to use Collaborative Filtering. It does not require any information about the items themselves and makes recommendations based on the past behavior of many users.

Usually, collaborative filtering can be reduced to two steps:

1.        Look for users with similar interests as the active user

2.        Use ratings from the other users identified above to make a prediction for the active user

Your task is to implement the first step using the number of inversions in the lists of user ratings as a numerical similarity measure.

An Inversion is a pair of elements\(Si,Sj\) of the sequence, such that i&lt;j and Si&gt;Sj. For example, sorted array \(1,2,3,4,5\) has zero inversions. Array \(5,1,2,3,4\) has four inversions \(5,1\), \(5,2\), \(5,3\), \(5,4\). Array \(1,3,5,2,4\) has three inversions \(3,2\), \(5,2\), \(5,4\). The maximum possible number of inversions in the array with n elements is .

Suppose we asked several people to rank three music genres. Now, we can form lists with ratings for each person from the most favorite genre to the least favorite. See the input description below for an example.

If a person in this set has identical preferences and ranks items exactly the same way as the active user, the number of inversions in the array would be zero. In general, the more inversions the array has, the more varied preferences are. In our example, Alice has 1 inversion compared to Bob. Meanwhile, John has 3 inversions compared to Bob.

So, Alice has more preferences in common with Bob and she is more suitable as the basis of a prediction.

Bob:  Rock, Blues, Jazz

Alice: Rock, Jazz, Blues

John: Jazz, Blues, Rock

输出：

Print the list of users to be considered for making a recommendation. The list must be sorted by the number of inversions in ascending order. If two users have the same count of inversions sort them alphabetically.

For example:

Alice, John

[https://www.cnblogs.com/grandyang/p/6657956.html](https://www.cnblogs.com/grandyang/p/6657956.html)

### 7. Normalize Levels

```text
Imagine a music studio has a set of recorded audio tracks they are mixing. The tracks each have their own range of audio levels. To mix them into a single track, the studio wants you to adjust them individually so they all fall into the same volume range.
Write a program that reads in a set of signal levels. The range of levels in each track can be any integer from 0 to 32767. Your program must return the same set of tracks but adjusted so they all peak at the same level of 100.
For example, consider these three tracks:
28,54,812,438
12,35,78,26
18,2,212,5
Your algorithm should:
1.        Determine the max value MM for each track. In the example, these would be 812, 78, and 212 respectively.
2.        Calculate the damping factor dd for each track as d = {100 \over M}d=M100
3.        Apply the weighting factor to each track's series of values as v \times dv×d where vv is the given value in the series
The results for each series should all have the same peak level.
For fractional values, round to the nearest whole number. Midpoint rounding should be upward/away from zero. For example, the value 3.5 should round to 4.

输入：
Each line of input will be a comma-separated series of positive integers 
representing the stream of sound levels for each track.
输出：
Print out a comma-separated series of the normalized values for each track, 
one track per line of output in the order they were input.
Using the example above, the correct output would be:
3,7,100,54
15,45,100,33
8,1,100,2
矩阵运算
在混音的过程中要把所有的track调整到统一的volume range，写一段程序，input是每个track的声音，需要把每个track统一成最大100的volume
Example：
28,54,812,438
12,35,78,26
18,2,212,5
结果：
3,7,100,54
15,45,100,33
8,1,100,2

d = 100/max(arr), 用arr每个数乘以d
给m个list，找到每个List的最大值，算出d = 100 / 最大值，然后把每个element转换成d * element
```

```python
import numpy as np
import math
inputdata = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
a = np.array(inputdata)
ratios = 100.0/np.max(a, axis = 1)
print ratios

for i in range(len(a)):
  for j in range(len(a[0])):
    a[i][j]=ratios[i]*a[i][j]
    print a[i][j]

print a
```

```text
import math
a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
ratios=[0]*len(a)
for i in range (len(a)):
  ratios[i] = 100.0/max(a[i])
print ratios

for i in range(len(a)):
  for j in range(len(a[0])):
    a[i][j]=int(ratios[i]*a[i][j])
    print a[i][j]

print a
```

* find max of each line 
* use 100 to divide the max to find the ratio to get the normalized value
* multiply each number in the array with the ratio to get the normalized levels
* numbers, sep by coma, matrix;
* 
### 8. Mth to Last Element from a Sequence

```python
def mth(vector):
    vector = list(vector)
    
    number = int(vector[len(vector) - 1])
    if number > len(vector) - 1:
        result = " "
        return result
    mth = vector[len(vector) - 1 - number]
    return mth

import sys
data = sys.stdin.readline().strip('\n').split()
print mth(data)
```

#### 

### 9. URL match 

[https://pypi.org/project/urlmatch/](https://pypi.org/project/urlmatch/)

#### ![](https://lh6.googleusercontent.com/5u_VvGye60sddvLDMc6houD_CoZvJfHe_BueU5Ss_v3sd0QrYlV_CZW32S1WTkafnz0zWTLFxlSisAI0Jwn8FvwVwQcWTl2b9Yu3LWSykrj2F_UMwvkfSyonJXnIRmzoZXTHxL21)

```python
link1, link2 = sys.stdin.readline().strip('\n').split(';')

from urlmatch import urlmatch
match_patter = link1

urlmatch(link1, link2)
```

### 10. 近似中位数解法QuickSelect 

Divide the list into sublists with k elements. The last sublist may contain fewer than k elements. Sort each sublist, get its median and append it to another list containing medians of sublists. If the medians list has a length more than k, compute its median recursively, proceeding with step 1. If the medians list has k elements or fewer, sort it and determine its median. Return it as a pivot. If an array is of odd length, the median is the middle element after the array has been sorted. If an array is of even length, there are two middle elements after it has been sorted. In this case, we will define the median as the left \(first\) of these two middle elements.  
recursion 

{% embed url="https://rcoh.me/posts/linear-time-median-finding/" %}

```python
import random

def quickselect_median(l, pivot_fn=random.choice):
    if len(l) % 2 == 1:
        return quickselect(l, len(l) / 2, pivot_fn)
    else:
        return 0.5 * (quickselect(l, len(l) / 2 - 1, pivot_fn) +
                      quickselect(l, len(l) / 2, pivot_fn))


def quickselect(l, k, pivot_fn):
    """
    Select the kth element in l (0 based)
    :param l: List of numerics
    :param k: Index
    :param pivot_fn: Function to choose a pivot, defaults to random.choice
    :return: The kth element of l
    """
    if len(l) == 1:
        assert k == 0
        return l[0]

    pivot = pivot_fn(l)

    lows = [el for el in l if el < pivot]
    highs = [el for el in l if el > pivot]
    pivots = [el for el in l if el == pivot]

    if k < len(lows):
        return quickselect(lows, k, pivot_fn)
    elif k < len(lows) + len(pivots):
        # We got lucky and guessed the median
        return pivots[0]
    else:
        return quickselect(highs, k - len(lows) - len(pivots), pivot_fn)
```

### 11. 均线法估值

A classic stock trading pattern happens when a 9-Day Moving Average \(9-DMA\) crosses the 50-Day Moving Average \(50-DMA\). This can be indicative of a bullish or a bearish setup, depending on the direction. When the 9-DMA crosses above the 50-DMA from below, it is Bullish. When the 9-DMA cross below the 50-DMA from above, it is Bearish. Write a program that reads in a series of dates and prices, calculates the 9-DMA and 50-DMA, then returns the dates of any bullish signals that occurred.

 datetime, moving average 如果短期average比长期average高了，就是牛市；如果短期比长期低了就是熊市。

1. 用pandas输入转换成DF，两列column，date，value
2. 用datetime转换日期变成天数，以第一天作为参考，另存一列叫day
3. 用np从第九天开始计算9-DMA，存一列9DMA，第50天开始计算50-DMA，存一列50DMA
4. 所以这两个DMA都有NA，把NA填成0 df.fillna\(0, inplace=True\)
5. filter = df\['9DMA'\]&gt;df\['50DMA'\] & df\['day'\]&gt;50
6. bulls = df.where\(filter, inplace=True\)
7. return bulls\['date'\].values

看看这个datetime处理吧  
[http://www.wklken.me/posts/2015/03/03/python-base-datetime.html](http://www.wklken.me/posts/2015/03/03/python-base-datetime.html)

```python
import pandas as pd
import numpy as np
from datetime import datetime

df=pd.DataFrame(data)

df.day = df.date.apply(lambda x: datetime.strptime(x, '%d/%m/%Y %H.%M.%S'))
df['9DMA'] = df.iloc[:,1].rolling(window=9).mean()
df['50DMA'] = df.iloc[:,1].rolling(window=50).mean()
df.fillna(0, inplace=True)

filter = df['9DMA']>df['50DMA'] & df['day']>50
bulls = df.where(filter, inplace=True)

return bulls['date'].values




#用np方法做的话
def movingaverage(values, window):
    weights = np.repeat(1.0, window)/window
    smas = np.convolve(values, weights, 'valid')
    return smas
```

## Behavior Question/Phone Interview Questions

1. 每道算法题后都1题解释自己的算法，分析时间空间复杂都，如果有时间的话再说改进思路
2. 你最近做的一个项目是什么？你在其中担任什么角色？你遇到了什么挑战？

* challenge: NYC crime 2GB 
* 1 person project for Big Data Course, Spark, eventually AWS
* random sampling 
* do it offline on databricks, so no need to install packages or learn complicated things
* Spark, trial & error, documentation
* learn AWS

1. Describe a time when you have worked as part of a successful team. What role did you play and what were the challenges you encountered?

* NYC CitiBike 3 member, got A and good demonstration in class
* cleaning data for the modeling process, bc not good at writing
* predict customer type
* but independent variables not much
* feature engineering \(binning datetime, using Google API for geo locations and eventually distance\), more features: time range; distance travelled; speed

1. 给没有quant背景的人介绍一个你做的quantitative project \(可能写作文可能口述\)
2. 介绍一个你用过的ml algorithm  为什么适合你的project

* K Means
* Clustering method, based on distance
* Random generate k centroids
* calculate distances from each datapoint to those k centroids 
* cluster to the closest centroids
* update new centroid based on the cluster's distances
* identify new centroids
* iterate
* although O\(kniteration\), easy to interpret the result

1. 作文题 Write and explain one of your previous project

Phone

1. why jpm, biggest challenge, career goal...etc
2. 45分钟的面试。先是问random forest，然后问别的bagging的模型，怎么做梯度下降，函数式编程的好处, ARIMA

## 前车之鉴

1. 其实总共有8个题，前四题都是选择问你会什么语言 没啥用 只有6个还是4个test case，都给在明面上的，应该没有隐藏的。做完一run就知道是不是都做对了，然后再submit。
2. 显示75min的实际用时

总共55min 但是它设置的时间大概得75min多

两道coding题，每题限时35min，两个简单的test cases  


如果是quant 分别是20min 40min 最后60min写作文  
  
  


1. BQ 三分钟的构思和improvement, 30秒准备时间，可以有两次机会 在做完题之后不用紧接着开始录视频，题和题之间可以休息，可以先想好了再点开。
2. 普遍反映的系统问题  

我第一题应该是读input有问题，test case也没过，但我之后放jupyter跑又是对的。第二题就读一个数所以没出问题  


不过输入要从文件读，valid parenthesis就被我搞出了点bug，因为这样读进来每一行都带一个换行符，我预处理会先处理掉奇数长度字符串就答案错误。。因为是屏幕输出所以debug不是很友好。  


问题的解决是很容易的，主要难点在于如何转换字符串并存储，特别是用C语言的话。

现在编程语言可选C, C++和Python。然后测试界面不够友好，万一出bug停不下来的话就直接关界面再进去，特别麻烦。  


Valid Parenthesis那题明明第一个test case "\(\)"竟然总输出False？？？到最后也没看出来到底哪儿能出错...在录的视频的最后解释一下，但愿面试官能谨慎对待吧。

p.s. 用的python，估计唯一的可能是sys.stdin获取input的时候出错了...  


发现valid parenthesis 没法一边读input一边进行compare，而是得先把所有input集合到一个string里 然后用LC上面的isValid（可能是我太菜或者漏掉了什么所以没法一边直接读一边compare）  


每道题的文字都超级长，基本上reading comprehension过关了就能写了。而且要自己读取input，process input，print output，超级麻烦



1. import numpy, pandas, urlmatch 都不可以；import heapq, math, datetime，random可以 
