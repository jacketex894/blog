+++
date = '2025-04-29T14:37:14+08:00'
draft = false
title = 'Go Map'
categories = [
  "Coding"
]
+++
## map
map 是 Go 中的哈希表（hash table），與 Python 中的 dict 類似。不過，Go 的 map 需要事先定義 key 和 value 的資料型別，並且 key 和 value 必須是相同資料型別的集合。

map 的型態如下
```
map[KeyType]ValueType
```

### 宣告
```
var m map[string][int]
m = make(map[string]int)
```
在 Go 中，使用 var 宣告一個 map 時，會得到一個 nil map，也就是尚未初始化的狀態。
若在未經初始化的情況下對 map 進行寫入操作（例如新增鍵值對），會得到錯誤訊息:  panic: assignment to entry in nil map
因此，必須使用 make 函數來初始化 map 才能進行寫入。

```
m := map[string]int{}
```
也可以使用 := 來直接建立空 map，透過此方式建立的 map 可以直接進行寫入。

### 修改/新增
```
m["apple"] = 3
```
map 可以直接透過 key 來新增或修改元素。

### 查詢
```
m["apple"]
i := m["apple"]
```
map 可以直接透過 key 來查詢值，若是 key 不存在，會得到值 0。

```
value,ok := m["apple"]
```
或著也可以透過兩個 assignment 的方法來確認 key 是否存在在 map。
value 會存放 key 所對應的值，若是 key 不存在於 map 則會是0。
ok 是布林值， true 表示 key 存在於 map， false 表示 key 不存在於 map。

```
_,ok := m["apple"]
```
如果希望在不檢索值的狀況下確認 key，可以使用 _ 。
這樣可以避免宣告了 value 卻沒使用，導致 go 編譯時報錯。
### 刪除
```
delete(m,"apple")
```
可以透過 delete 來刪除 key，delete 不會回傳任何東西，若是指定的 key 不存在 map ，則不會做任何事。

### 遍歷
map 是無序的，所以不能保證每次遍歷都可以是相同的順序。
```
package main

import "fmt"

func main() {
    m := map[string]int{
        "apple":  5,
        "banana": 2,
        "cherry": 7,
    }
    for key, value := range m {
        fmt.Println(key, value)
    }
}
```

可以透過以上的程式碼多跑幾次去測試。
第一次執行可能是
```
apple 5
banana 2
cherry 7
```
但第二次執行可能就變成
```
banana 2
cherry 7
apple 5
```
當然也有可能連續兩次執行都是相同順序，所以當有需要 map 以特定順序遍歷的時候，最好是另外儲存 key ，再對 key 進行 sort。
## 參考
https://go.dev/blog/maps
