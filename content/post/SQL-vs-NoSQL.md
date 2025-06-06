+++
date = '2025-05-22T11:17:31+08:00'
draft = false
title = 'SQL vs NoSQL'
categories = [
  "System Design"
]
+++
## SQL (Structured Query Language)

SQL 是指一種查詢語言，用來跟關聯式資料庫(RDBMS,Relational Database Management System)互動。

### RDBMS (Relational Database Management System)

使用 Table 來儲存管理資料，Table 中會以表格的方式儲存資料，如下所示。

| user_id     | user_name          | email |
|----------|---------------|------|
| 0     | Jack    | example@test.com   |

同一欄位的資料，會規定是要相同的資料型態。

每個表格會有一個 Primary Key，用來唯一識別每條資料。

表格跟表格之間可以建立關聯(join)，表示表格可以特過特定的欄位資料來跟另一個表格相互關聯。

以以下兩張表來做說明

users
| user_id     | user_name          | email |
|----------|---------------|------|
| 0     | Jack    | example@test.com   |

user_detail
| user_name     | Age       |
|----------|---------------|
| Jack     | 28    |

假如想要查詢使用者姓名、電子郵件、年齡，可以透過 SQL 語法關聯。
```
SELECT users.user_name, users.email, user_details.Age
FROM users
INNER JOIN user_details
ON users.user_name = user_details.user_name;
```

| user_id     | user_name          | email |
|----------|---------------|------|
| 0     | Jack    | example@test.com   |

RDBMS 具備 ACID 特性，在資料庫中，使用一筆交易(transaction)來指一組資料庫操作，以下以交易來解釋 ACID。
- 原子性(Atomicity):一筆交易中只要有任何操作失敗，就必須回滾(roll back)為交易前的狀態。
- 一致性(Consistency):交易前後，資料庫必須保持一致的狀態，交易不會破壞資料庫的完整系。e.g. A 與 B 一共有 200 元，A 轉帳給 B，轉完帳後 A 與 B 依然一共有 200 元。
- 隔離性(Isolation):交易之間不會互相干擾。每個交易的執行不應該被其他交易影響，isolation 可以再分為四個級別。
- 持久性(Durability):只要資料寫入資料庫，即便是系統崩潰也不會丟失。

### index
RDBMS 透過 B-Tree 資料結構來建立索引，並且可以依照排序來儲存索引。

## NoSQL(Not only sql)

NoSQL 通常指的是不使用表格關聯方式來儲存資料的資料庫，儲存方式多元，以我自己常用的 mongodb 來講，是以 Json 的方式來儲存資料。
```
[
  {
    "user_id": 1,
    "user_name": "Alice",
    "email": "alice@example.com",
    "age": 30,
  },
  {
    "user_id": 2,
    "user_name": "Bob",
    "email": "bob@example.com",
    "age": 25,
  },
]
```
mongodb 一開始並不支援 join，但是後來新增了 $lookup ，其行為類似於 SQL 的 JOIN。

NoSQL 資料庫通常是用於在分散式系統，所以需要考慮 CAP。

### CAP

CAP 理論指的是在一個分散式系統中，只能保證以下三者其二。
- 一致性(Consistency):每次讀取操作都能取得系統中最新寫入的資料。
- 可用性(Availability):每次對系統的請求都能得到回覆。
- 分區容錯性(Partition Tolernce):當系統的不同節點支籤無法通訊，系統仍然能保持運作。
### index
NoSQL 使用 Hash function 將資料轉換為 Hash value，但也因此 nosql 的索引是無序的。
## 比較

### 縱向擴展 (Vertical Scaling)
縱向擴展指的是透過增加硬體提升效能(e.g.增加 CPU、memory...)，來提升單個伺服器的效能。

SQL : 支援，對於 SQL 資料庫來說垂直擴展是最能增加效能的方法，隨著資料量增加，對於硬體的需求也會逐漸提高。
  
NoSQL : 支援
  
### 橫向擴展 (Horizontal Scaling)
橫向擴展指的是增加更多伺服器或節點來提升效能。

SQL:支援但不擅長，因為要維持 ACID，而 ACID 在不同 server 之間要維持原子性會非常困難。

NOSQL:支援，並且由於資料間沒有關聯性，NOSQL 能更好的運作在分散式系統。

### 寫入

SQL:為了保持ACID，當在處理高併發的大量寫入請求時會消耗更多資源並影響效能。

NOSQL:NOSQL 在處理高併發的大量寫入請求時，表現會比 SQL 來的好，原因是因為 NOSQL 的資料在寫入時不必立刻同步到所有 server，而是最終資料一致就好。

### 查詢

SQL:SQL 擅長處理複雜的查詢，特別是涉及多表 Join 以及複雜的過濾條件時。

NOSQL: 單一紀錄的查詢快，但是複雜的查詢，特別是涉及多表的查詢會相較 SQL 慢。

### 結論

SQL:適合需要 ACID 來保證資料正確，複雜查詢的應用。

NOSQL:適合高效能寫入，需要在分散式系統運作，可接受最終一致性的應用。

## 補充
### isolation level

isolation 分成 四個 level，分別是為了解決不同狀況的需求。

先解釋資料庫在儲存時會遇到那些狀況。
- Dirty Read : 一個交易向資料庫寫入資料時，還沒 commit，另一個交易卻讀取了尚未 commit 的資料
  - A 交易在一次交易中讀取了兩次資料，但是在第一次資料讀取跟第二次資料讀取之間，B交易將資料改寫了，導致兩次讀取資料的數值不同。
- Non-repeatable reads : 同一個交易中讀取了多次資料，但是卻取得了不同的結果
  - A 交易在一次交易中讀取了多次資料，但是在資料讀取間，B交易改寫了資料並 commit，導致了A讀取了相同資料卻取得不同結果。
  - Dirty Read 也屬於 Non-repeatable reads 差別在於有沒有 commit
- Phantom reads: 當在同一個交易中，連續讀取資料取得的比數卻不同。

為了解決這些狀況，於是將 isolation 分成 4 個 level
-  Read Uncommited : 可以讀取為 commit 的資料。
-  Read Commited: 只能讀取到已 commit 的資料，解決 Dirty Read。
-  Repeatable Read: 同一交易中，只要搜尋條件相同，便能取得相同結果，解決 Non-repeatable reads。
-  Serializable: 多個 transaction 執行時，只要順序相同，得到的結果便相同，但是因為資料庫會確保 transaction 依序完成，會造成效能降低，解決 Phantom reads。
## 參考:
https://medium.com/appdev-ooops/%E8%B3%87%E6%96%99%E5%BA%AB-%E4%BD%A0%E7%9C%9F%E7%9A%84%E7%94%A8%E5%B0%8D%E4%BA%86%E5%97%8E-sql-vs-nosql-9621a05180d4
https://www.codegym.tech/blog/sql-vs-nosql
https://realnewbie.com/basic-concent/database/sql-vs-nosql-differences-beginner-guide/
https://oldmo860617.medium.com/%E5%88%9D%E6%AD%A5%E8%AA%8D%E8%AD%98%E5%88%86%E6%95%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E8%88%87-nosql-cap-%E7%90%86%E8%AB%96-a02d377938d1
https://www.mysql.tw/2024/05/mysql-innodb-isolation-level.html