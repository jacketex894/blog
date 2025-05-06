+++
date = '2025-05-06T21:07:33+08:00'
draft = false
title = 'Leetcode 347'
+++
## 題目
[347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements)

給予一個數字陣列 nums 跟數字 k ，回傳這個數字陣列中出現的數字頻率前 K 多的數字。
題目有保證 answer 是 unique ， 所以我認為不用考慮到頻率有相等，但剛好在第 K 跟第 K + 1 的狀況。

## Pseudocode

```
初始化一個 frequency map 用來記錄元素出現頻率
從第 0 個元素開始遍歷 nums:
    如果元素沒有出現過:
        將元素作為 key 在 frequency 中對應 0
    將 frequency 中的元素對應的 value + 1，紀錄出現頻率 + 1 
根據 frequency 內的 value 去排序對應的 key 出現頻率的多寡
將排序結果中前 K 大的 key 回傳
```
時間複雜度會根據排序的演算法而有不同，如果是 Python 的 sorted 或 go 的 sort是O(n log n)

## Python
```
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        frequency = {}
        for num in nums:
            if num not in frequency:
                frequency[num] = 0
            frequency[num] += 1
        
        answer = sorted(frequency.keys(),key = lambda x: frequency[x],reverse=True)
        return answer[:k]
```
### Go
```
import (
    "fmt"
    "sort"
)

func topKFrequent(nums []int, k int) []int {
    frequency :=  map[int]int{}
    for index:=0;index < len(nums);index += 1{
        if _,ok := frequency[nums[index]]; !ok{
            frequency[nums[index]] = 0
        }
        frequency[nums[index]] += 1
    }
    answer := []int{}
    for number := range frequency{
        answer = append(answer,number)
    }

    sort.Slice(answer,func(i,j int) bool{
        return frequency[answer[i]] > frequency[answer[j]]
    })
    return answer[:k]
}
```