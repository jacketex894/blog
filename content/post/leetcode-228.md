+++
date = '2025-04-21T16:52:48+08:00'
draft = false
title = 'Leetcode 228'
+++

## 題目
[228. Summart Ranges](https://leetcode.com/problems/summary-ranges/)

給予一個 sorted 過的數字陣列，要求依照給定的數字陣列中的連續數字輸出字串陣列。
e.g. 
input : [0,1,2,4,5,7] 
output : ["0->2","4->5","7"]

數字陣列中連續數字分別為 [0,1,2] [4,5] [7]。
所以根據題意可以拆解成三個區間字串 "0 -> 2" , "4 -> 5" , "7"

根據題目說明，數字陣列長度可能為 0 ，所以需要考慮此特殊狀況。
input 的每一個元素都是獨立的，並且從小排到大。

## Pseudocode
首先，先將應實作邏輯整理出來，好讓後面可以使用不同語言實作。
目的是為了邊學習 go 語言的基本操作。

先針對特例做處理，當收到一個空陣列的時候，應該回傳一個空陣列，而不是繼續其他操作。
```
如果 input 是 空陣列:
    回傳空陣列
```

接下來處理連續數字的邏輯:
1. 當遍歷 input 的時候，需要確認遍歷的元素是否跟前一個元素是連續的。
2. 如果不連續的話，便產生一個字串放入需回傳的答案陣列。

這裡需要兩個陣列:
1. temp_arr : 用來存放與確認目前遍歷的元素是否是連續數字所使用。
2. answer : 用來存放要回傳的答案。

```
# 從第 0 個元素開始存放，迴圈從第 1 個元素開始跟前一個元素比對
初始化 temp_arr 為 [input[0]]
初始化 answer 為空陣列

從第 1 個元素開始，逐一遍歷 input 中的元素:
    如果 當前元素 不等於 temp_arr 最後一個元素 + 1:
        # 表示不連續，需結束目前區間
        將 temp_arr 根據題義轉為區間字串並加入 answer
        初始化 temp_arr 為 [當前元素]
    否則:
        將當前元素加入 temp_arr

# 處理最後一段尚未加入的區間
將 temp_arr 根據題義轉為區間字串並加入 answer

回傳 answer
```

根據題義產生區間字串的過程，由於在遍歷與遍歷完成後都需要使用到，所以另外寫成 function，方便修改。
根據題義產生區間字串會需要連續數字的第 0 個數字 + "->" + 最後一個數字。
但是如果如果不連續，只有單一個數字的話，則只要輸出該數字即可。
```
字串 = temp_arr 中的第0個元素
如果 temp_arr 的大小超過 1:
    字串加上 "->" 以及 最後一個元素
回傳字串
```


### 子函式：產生區間字串
Input : temp_arr，一個包含連續整數的子區段
Output : 符合題目格式的字串表示，例如 "1->3" 或 "5"
```
初始化 字串 為 temp_arr 的第一個元素（轉成字串）

如果 temp_arr 的長度大於 1:
    將字串加上 "->" 和 temp_arr 的最後一個元素（轉成字串）

回傳字串
```
Note:
* 如果只有一個數字，直接輸出該數字。
* 如果是一段連續區間（例如 [1,2,3]），輸出 "1->3"。
### 主函式
Input : 一個遞增排序的整數陣列 nums（無重複）
output : 將所有連續的整數區段，以字串格式表示的陣列
```
如果 input 是 空陣列:
    回傳空陣列

# 從第 0 個元素開始存放，迴圈從第 1 個元素開始跟前一個元素比對
初始化 temp_arr 為 [input[0]]
初始化 answer 為空陣列

從第 1 個元素開始，逐一遍歷 input 中的元素:
    如果 當前元素 不等於 temp_arr 最後一個元素 + 1:
        # 表示不連續，需結束目前區間
        將 temp_arr 根據題義轉為字串並加入 answer
        初始化 temp_arr 為 [當前元素]
    否則:
        將當前元素加入 temp_arr

# 處理最後一段尚未加入的區間
將 temp_arr 根據題義轉為字串並加入 answer

回傳 answer
```
## Python
用熟悉的 python 實作，驗證邏輯
```
class Solution:
    def get_answer_string(self,temp_arr: List[int]) -> str:
        answer_string  = f"{temp_arr[0]}"
        if len(temp_arr) > 1:
            answer_string += f"->{temp_arr[-1]}"
        return answer_string
    def summaryRanges(self, nums: List[int]) -> List[str]:
        if not len(nums):
            return []
        temp_arr = [nums[0]]
        answer = []
        for index in range(1,len(nums)):
            if nums[index] - 1 != temp_arr[-1]:
                answer_string = self.get_answer_string(temp_arr)
                temp_arr = [nums[index]]
                answer.append(answer_string)
            else:
                temp_arr.append(nums[index])
        answer_string = self.get_answer_string(temp_arr)
        answer.append(answer_string)
        return answer
```

## GO
```
import "strconv"

func get_answer_string(temp_arr [] int) string{
    answer_string := strconv.Itoa(temp_arr[0])
    if len(temp_arr) > 1{
        answer_string += "->" + strconv.Itoa(temp_arr[len(temp_arr) - 1])
    }
    return answer_string
}
func summaryRanges(nums []int) []string {
    if len(nums) == 0{
        return []string{}
    }
    var temp_arr = []int{}
    var answer = []string{}
    temp_arr = append(temp_arr,nums[0])
    for index := 1 ; index < len(nums); index ++{
        if nums[index] - 1  != temp_arr[len(temp_arr) - 1]{
            answer_string := get_answer_string(temp_arr)
            answer = append(answer,answer_string)
            temp_arr = []int{}
        }
        temp_arr = append(temp_arr,nums[index])
    }
    answer_string := get_answer_string(temp_arr)
    answer = append(answer,answer_string)
return answer
}
```