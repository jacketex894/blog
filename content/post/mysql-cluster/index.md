+++
date = '2025-06-14T14:19:51+08:00'
draft = false
title = 'Mysql Cluster'
categories = [
  "System Design"
]
+++
# Ｍysql Cluster
## Cluster
利用多台獨立的伺服器或是電腦(節點,Node)，組成集群(cluster)，共同計算或提供一項服務。
### 優點
1.高可用性(Availability):資料會複製多份在不同的機器上，即便其中一台節點損壞，也可以持續提供服務。

2.擴展性(Scalability）): mysql cluster 可以透過新增一個新的節點(可能是一台新的電腦或是伺服器)，來分散流量負擔。

3.資料一致性(Consistency): mysql cluster 可以維持 ACID 來保證資料一致性。

### Replication
mysql cluster 透過同步(synchronous) 的方法來保持資料一致性備份資料到其他節點。

而與先前筆記的 mysql 的主從式資料庫有所不同的點在於，mysql 的主從式資料庫是透過非同步的方式來備份資料到從資料庫。

從寫入一筆資料到資料庫來分別兩者的區別

* mysql cluster : 會需要等待所有節點都寫入完成後，寫入才完成。

    若有節點寫入不完成，所有節點會一起 roll back。

* mysql 主從式資料庫 : 當主資料庫寫入後，對於主資料庫來說資料寫入已經完成，並會將該事件記錄到 bin log，再由從資料庫取得 bin log 後 replay 事件同步資料。

    若是從資料庫寫入失敗，主資料庫並不會因此 roll back，而從資料庫則會停止運作，資料維持在寫入失敗的這一刻，不會再更新。
    可以參考([database-repliaction](/post/database-replication/))沒有備份主資料庫的資料就設定了主從式會發生的狀況。

mysql cluster 透過 two-phase commit 的機制來保證資料寫入所有節點。

#### two-phase commit

two-phase commit 是被設計用來處理所有參與節點是否 commit 或是 abort(roll back) transaction 的一種演算法。

必須滿足以下條件：
* 每個節點都要有穩定的儲存空間提供給 write-ahead log。
    * write-ahead log 指的是所有對 DB 的操作都必須先寫入在此log，才能對 DB 進行變動。
* 沒有節點會永遠處在崩潰狀態。
* 所有寫在  write-ahead log 的資料不會因為 node 的崩潰，而遺失或是損壞。
* 任意兩個節點可以彼此溝通。

1. 需要有一個協調者(coordinator process)，向所有參與節點提出請求，詢問是否要 commit 或是 abort transaction，並根據回應來決定動作。 
    
    1-1. 協調者向所有參與節點發出請求確認是否commit transaction，並等待所有參與節點回應。

    1-2. 參與節點執行協調者詢問是否 commit 的 transaction，並記錄到 undo log 以及 redo log。(即便執行了，參與節點也並沒有在執行完 release 資源或lock)

    1-3. 所有參與節點根據執行成功或失敗，回覆是要 commit 或是 abort。

2. 協調者根據回應結果決定要 commit 還是 abort transaction，並向所有節點發出 notify。所有節點收到後根據投票決定的結果執行。
    
    * success:
    1. 協調者收到所有參與節點都決定 commit，並向所有參與節點送出訊息要求 commit。
    2. 所有參與節點完成操作後，release 所有資源跟 lock。
    3. 所有參與節點回傳 acknowledgement 給協調者。
    4. 協調者收到所有 acknowledgement 完成 transaction。

    * fail:
    當任一節點回覆 abort 或是超時沒有回應
    1. 協調者向所有參與節點發出 roll back 請求
    2. 所有參與節點根據 undo log undo 後，release 資源跟 lock。
    3. 所有參與節點回傳 acknowledgement 給協調者。
    4. 協調者收到所有 acknowledgement 後 undo transaction

## 參考:
https://en.wikipedia.org/wiki/MySQL_Cluster
https://en.wikipedia.org/wiki/Two-phase_commit_protocol
https://en.wikipedia.org/wiki/Write-ahead_logging
https://www.cc.ntu.edu.tw/chinese/epaper/0037/20160620_3707.html