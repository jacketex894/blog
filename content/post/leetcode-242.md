+++
date = '2025-04-29T18:35:49+08:00'
draft = false
title = 'Leetcode 242'
+++
## 題目
[242. Valid Anagram](https://leetcode.com/problems/valid-anagram)

給予兩個字串 s 跟 t ， 判斷 t 是否是 s 的 anagram。
是的話回傳 true ， 不是的話，回傳 False。
anagram 指的是 t 字串的字元是否完全來自於 s ，並且每一個字元只被使用一次。

## Pseudocode
[leetcode-217]({{< relref "post/leetcode-217.md" >}}) 的延伸題。

根據題義可以判斷 s 跟 t 的長度必須相同，所以當長度不同便可直接回傳 false。
```
如果 s 的長度不等於 t 的長度:
    回傳 false

從第 0 個字元開始，逐一遍歷 s 的每一個字元:
    紀錄字元出現的頻率
從第 0 個字元開始，逐一遍歷 t 的每一個字元:
    比對字元是否在 s 出現過，以及是否超過 s 的頻率，若並未出現過或超出頻率:
        回傳 false
通過所有檢查，回傳 true
```
時間複雜度:O(N)

## Go
```
func isAnagram(s string, t string) bool {
    if len(s) != len(t){
        return false
    }
    appear := map[byte]int{}

    for index := 0 ; index < len(s) ; index += 1{
        if _,ok := appear[s[index]]; !ok{
            appear[s[index]] = 0
        }
        appear[s[index]] += 1
    }
    for index:=0 ; index < len(t) ; index += 1{
        if value,ok := appear[t[index]]; ok && value > 0{
            appear[t[index]] -= 1
        }else{
            return false
        }
    }
    return true
}
```