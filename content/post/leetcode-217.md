+++
date = '2025-04-27T11:44:03+08:00'
draft = false
title = 'Leetcode 217'
+++
## 題目
[217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/description/)

給予一個數字陣列 nums，如果陣列中有重複的元素就回傳 True, 否則回傳 False。

## Pseudocode
這題是蠻簡單的題目，可以透過將元素存入類似 python 的 dict，這類的 hash table 來解決。
```
從第一個元素開始，逐一遍歷 nums 中的每一個元素:
    如果元素沒有出現過:
        將元素紀錄下來用於比對是否出現過
    如果元束出現過:
        回傳 True 值
當遍歷完 nums 依然沒有重複元素:
回傳 False 值
```

## Go
```
func containsDuplicate(nums []int) bool {
    appear := map[int]bool{}
    for i := 0 ; i < len(nums) ; i ++{
        if _,ok := appear[nums[i]]; ok{
            return true
        }else{
            appear[nums[i]] = true
        }
    }
    return false
}
```
主要使用了 map 來紀錄出現過的元素。
關於 map 請參考 [go-map]({{< relref "post/go-map.md" >}})