+++
date = '2025-05-02T14:37:32+08:00'
draft = false
title = 'Leetcode 49'
categories = [
  "Coding"
]
+++
[49. Group anagram](https://leetcode.com/problems/group-anagrams)

給予一個字串陣列，將相同的 angram 放在一個陣列中(為求方便，稱呼其為 angram 陣列)， 再將所有 angram 陣列放到一個陣列後回傳，可以以任何順序回傳。

## Pseudocode - 1
此題跟 [leetcode-242]({{< relref "post/leetcode-242.md" >}}) 很像。
嘗試呼叫 [leetcode-242]({{< relref "post/leetcode-242.md" >}}) 的 isAnaragram 來解決這題。
```
從第 0 個字串開始，逐一遍例輸入字串陣列的每一個字串：
    確認目前字串有無被確認為其他字串的 angram：
        跳下一個字串繼續
    從第一個迴圈取得的字串後的字串開始，逐一遍歷輸入字串陣列的每一個字串：
        確認取得的字串有無被確認為其他字串的 angram：
            跳下一個字串繼續
        確認取得字串是否是目前字串的 angram，是的話記錄下來
    將目前字串跟目前字串的 angram 都放入答案
回傳所有答案
```
時間複雜度: O(N^2)
這裡的時間複雜度不包含確認兩個字串是否是彼此的 angram。
假設 Ｌ 是字串長度的話（先不考慮所有字串的長度是否都相同為Ｌ），那麼計算兩個字串是否是彼此的 angram 的時間複雜度為: O(L)。
此解法的時間複雜度為: O(N ^ 2 * L)。
但是，直接 submit 會得到超時。


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
func groupAnagrams(strs []string) [][]string {
    find := map[int]bool{}
    answers := [][]string{}
    for index := 0 ; index < len(strs) ; index +=1{
        answer := []string{}
        if find[index]{
            continue
        }
        find[index] = true
        for second_index := index + 1 ; second_index < len(strs); second_index += 1{
            if find[second_index]{
                continue
            }else{
                if isAnagram(strs[index], strs[second_index]){
                    answer = append(answer,strs[second_index])
                    find[second_index] = true
                }
            }
        }
        answer = append(answer,strs[index])
        answers = append(answers,answer)
    }
    return answers
```
## Pseudocode - 2
既然超時的話，就要考慮到可以如何降低時間複雜度。
目前是兩兩比較是否是 angram，換個想法來說，其實是兩個字串在比較是否完全是由相同字元組成，但是字元順序可能不同。
那如果字元順序相同的話，是否可以直接比較字串是否相同，甚至是將字串作為 key 放入 hash table，借助 hash table O(1) ~ O(N) 的速度來判斷字串是否出現過就好，便可以縮短兩兩比較所需的 O(N^2) 時間了。
```
初始化 hash table "appear" key 是 sorted 過後的字串，value 是陣列，用來存放原本字串。
從第 0 個字串開始，逐一遍例輸入字串陣列的每一個字串：
    排序目前字串
    確認目前排序字串有無出現過，若無出現過：
        初始化 appear[排序目前字串] 為空陣列
    將目前字串加入 appear[排序目前字串]
將 appear 中的所有陣列提取出來，放入到一個陣列後回傳。
```
時間複雜度：O(N)
這裡的時間複雜度不包含sort字串。
假設 Ｌ 是字串長度的話（先不考慮所有字串的長度是否都相同為Ｌ），那 sort 字串的時間會再根據 sort 的方法不同而有不同。

假設都使用語言的 sort function:

* Python: O(L log L)
* GO: O(L Log L)

## Python
```
def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
    appear = {}
    for word in strs:
        sort_word = ''.join(sorted(word))
        if sort_word not in appear:
            appear[sort_word] = []
        appear[sort_word].append(word)
    answer = []
    for sort_word,words in appear.items():
        answer.append(words)
    return answer
```

## GO
```
import ("sort"
)
func groupAnagrams(strs []string) [][]string {
    appear := map[string][]string{}
    for index:= 0 ; index < len(strs) ; index += 1{
        words := []byte(strs[index])
        sort.Slice(words, func (i,j int) bool {
            return words[i] < words[j]
        })
        sort_words := string(words)
        if _,ok := appear[sort_words];!ok{
            appear[sort_words] = []string{}
        }
        appear[sort_words] = append(appear[sort_words],strs[index])
    }
    answer := [][]string{}
    for _,words := range appear{
            answer = append(answer,words)
    }
    return answer
}
```

## Reference
https://ithelp.ithome.com.tw/articles/10268906?sc=iThelpR