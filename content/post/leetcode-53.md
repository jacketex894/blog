+++
date = '2025-05-07T16:46:07+08:00'
draft = false
title = 'Leetcode 53'
+++
## 題目
[53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

給予一個數字陣列 nums, 找到該陣列的子集合中，相加最大的合，並且回傳合。

## Pseudocode
這題我一開始的想法是應該是 DP 相關題目，直接暴力O(N^2)解會超時。
我的思路是在計算和的時候，要想辦法沿用先前的計算。
最大和的值，從 nums 的第0個元素開始，因為只有一個元素也是子集合。
```
初始化 max_answer 為 nums[0] 用來記錄最大和
初始化 temp = 0 用來記錄當前的和

#以下用 max(A,B) 表示 A,B 中取較大者
從第 0 個 元素開始逐一遍歷 nums:
    當 temp + 元素 大於 max_answer:
        # 當元素比 temp + 元素還大的時候，就應該要從元素開始計算了。
        temp = max(元素,temp + 元素)
        max_answer = temp
    否則:
        temp = max(元素,temp + 元素)
        max_answer = max(max_answer,num)
#確認到最後一個元素的連續和有沒有比目前的答案還大
max_answer = max(temp,max_answer)

回傳 max_answer
```
其實我第一版的思路有一些重複計算的地方，因為我最開始的思路是`當 temp + 元素 大於 max_answer` 時，應該要重置 temp，所以以上 Pseudocode 的內容是邊 debug 得出的。
仔細想可以發現，其實無論如何當 temp + 元素比元素本身還要小的時候，就不該繼續計算合了。

舉以下兩個例子
```
目前迴圈都走到 index = 1

[4,-1,2,1]
4 + -1 > -1
所以繼續計算下去 3 比從 -1 開始從頭計算還要大

[-5,1,6,2]
-5 + 1 < 1
所以繼續計算下去 -4 比從 1 開始從頭計算還要小
```
而 max answer 在每次更新 temp 時確認是否有比較大就好，所以可以將程式碼簡化為

```
初始化 max_answer 為 nums[0] 用來記錄最大和
初始化 temp = 0 用來記錄當前的和

#以下用 max(A,B) 表示 A,B 中取較大者
從第 0 個 元素開始逐一遍歷 nums:
    temp = max(元素,temp + 元素)
    max_answer = max(max_answer,num)

回傳 max_answer
```

時間複雜度一樣是 O(N)，但後者的程式碼意義應該更為清楚。


## Python
```
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        max_answer = nums[0]
        temp = 0
        for num in nums:
            temp = max(num,temp + num)
            max_answer = max(max_answer,temp)
        return max_answer
```

## Go
```
func max(A,B int)int {
    if (A > B){
        return A
    }
    return B
}
func maxSubArray(nums []int) int {
    max_answer := nums[0]
    temp := 0
    for index := 0 ; index < len(nums) ; index ++{
        temp = max(nums[index],temp + nums[index])
        max_answer = max(max_answer,temp)
    }
    return max_answer
}
```