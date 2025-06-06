+++
date = '2025-06-03T17:05:01+08:00'
draft = false
title = 'Database Replication'
+++
## 資料庫複寫(Database Replication)
指的是將資料庫內的資料，複製到不同的資料庫實體上。 

等於是說將資料備份到不同的資料庫上，這樣做的好處是，當一台機器壞掉時，還有其他台機器存有資料。

例如異地備份，就是將資料存放到不同地點的機器上，來避免天災、人禍導致的資料損失。
另外是可以分配讀取查詢到複數的機器上，提高讀取效率。

但是分配讀取到不同的資料庫，就需要考慮到寫入是否只能限定哪幾台資料庫，否則可能提升讀取效率卻也會造成資料不一致的問題。

## 主從機制(Master Slave)
主從機制是常見的一種資料庫架構模式，將一個資料庫伺服器設為主資料庫(Master)，將其他多個資料庫伺服器設定為從資料庫(Slave)。


### 主資料庫(Master):
* 負責所有的資料寫入。
* 將資料變更記錄到log(e.g. mysql binlog)。
  * 以 mysql 的 binlog 來說，主資料庫是將變更資料的事件寫到 binlog，從資料庫再向主資料庫 pull binlog ，並根據 binlog 中的事件 replay 操作來更新從資料庫內的資料。

### 從資料庫(Slave):
* 不能進行資料寫入。
* 用於資料讀取，分散資料讀取的負擔。
  
### 優點:
1. 分散讀取的查詢到不同的資料庫，可以提高查詢的效率。
2. 當其中一台資料庫出現問題時，可以不用擔心資料遺失，或長時間無法使用服務。
   * 主資料庫出現問題時，可以將從資料庫轉換為主資料庫。
   * 從資料庫出現問題，當只有一台時，可以先將讀取操作導向主資料庫。
   * 若有多台從資料庫，則先導向其他正常資料庫。

### 缺點:
1. 根據使用的機制不同，如有多個資料庫寫入，需考慮資料同步的問題。
2. 主資料庫錯誤的話，需要手動將從資料庫轉為主資料庫。

## 使用 mysql 建立主從式架構

可以參考我自己建立的[例子](https://github.com/jacketex894/System-design/tree/main/database_replication)

以下會以我自己的範例結構做說明。
```
├── docker-compose.yml
├── master-database
│   ├── Dockerfile
│   ├── init.sql
│   ├── my.cnf
└── slave-database
    ├── Dockerfile
    ├── my.cnf
```
這個結構中將主資料庫放在 master-database，從資料庫放在 slave-database，master-datase 跟 slave-database 會各自建立容器，並使用 docker compose 做管理。


可以先 clone 下來後，執行 docker compose up 後在跟著以下說明進行。

### docker-compose.yml
```
services:
  mysql-master:
    build: ./master-database # 使用指定目錄建構映像
    container_name: mysql-master # 指定容器名稱
    ports:
      # 指定port，要注意的是 mysql default 3306，這裡是將 mysql 修改後的 port 映射到主機的3307
      - "3307:3307" 
    environment:
      # 設定 mysql 密碼為環境變數，需搭配 master-database/Dockerfile 
      MYSQL_ROOT_PASSWORD: test 
    # 設定掛載資料夾
    # mysql 目的是為了重啟容器不遺失資料
    # init.sql 是初始化資料庫時建立 user_db 跟 table users
    # replication 為了備份資料用
    volumes:
      - ./master-database/mysql:/var/lib/mysql
      - ./master-database/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./master-database/replication:/replication
    # 加入 docker 網路跟 slave 溝通
    networks:
      - mysqlnet

  mysql-slave:
    build: ./slave-database # 使用指定目錄建構映像
    container_name: mysql-slave # 指定容器名稱
    # 指定port，要注意的是 mysql default 3306，這裡是將 mysql 修改後的 port 映射到主機的3308
    ports:
      - "3308:3308"
    environment:
      # 設定 mysql 密碼為環境變數，需搭配 slave-database/Dockerfile
      MYSQL_ROOT_PASSWORD: test
    # 設定掛載資料夾
    # mysql 目的是為了重啟容器不遺失資料
    # replication 為了放 master 的備份資料用
    volumes:
      - ./slave-database/mysql:/var/lib/mysql
      - ./slave-database/replication:/replication
    # 加入 docker 網路跟 slave 溝通
    networks:
      - mysqlnet

networks:
  mysqlnet:
    driver: bridge
```

### init.sql

設想可能在建立主從式架構前，都是只使用一個資料庫來存取資料，但是不夠應付系統需求了，需要建立主從式架構來分散流量。

所以在我的例子中，可以在 master-database 資料夾中，看到 init.sql。

```
CREATE DATABASE user_db ;
USE user_db;
CREATE TABLE users(
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    user_name VARCHAR(255)  UNIQUE NOT NULL,
    mail VARCHAR(255) UNIQUE NOT NULL
);
```

目的是為了模擬在設立主從式架構前就有建立 table。

在設定主從式架構前，如果有資料的話，需要主動備份到從資料庫。

因為從資料庫不會同步設定主從式架構前的資料。

這有可能導致 master 有 table users，並且插入了一筆資料。

但是 slave 並沒有 table users，卻跟著 binlog 中的事件 replay，導致錯誤。

master 跟 slave 的 replication 掛載資料夾就是為了使用在將主資料庫的資料先備份到從資料庫用的。

要如何備份會在後續步驟講解，也可以試試沒有備份時直接設定主從，查看在主資料庫插入一筆資料後，從資料庫會報什麼錯誤。

可以閱讀 [Replication Implementation](https://dev.mysql.com/doc/refman/9.3/en/replication-implementation.html) 了解 replication 更詳細的內容。

### my.cnf
以 master-database/my.cnf 內容來說明
```
[mysqld]
port=3307
```
將 default 的 port 3306 改成 3307，也可以不設定，default 會為 3306
### Dockerfile
以 master-database/Dockerfile 內容來說明
```
FROM mysql:8.0
#將 docker compose 設定環境的密碼設定為 sql 的 密碼
ENV MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD

#將設定套用到容器內的 mysql  資料庫
COPY my.cnf /etc/mysql/my.cnf
```

## Set up
### 1. 設定主資料庫

  首先需要設定 my.cnf，新增以下內容，或著也可以直接使用範例中的 my.cnf 移除掉註解。
  ```
  # 設定 server 唯一id
  server-id=1
  # 設定 binlog 檔案最大容量
  max_binlog_size=100M
  # 設定記錄檔案名稱
  log_bin = mysql-bin
  ```

  將修改後的內容新增到容器內，因為範例建立的容器沒有 vi 或是 nano。
  ```
  docker cp ./my.cnf mysql-master:/etc/mysql/my.cnf
  ```

  重啟容器套用設定。
  ```
  docker restart mysql-master
  ```

  為了繼續設定，要進入到容器內。
  ```
  docker exec -it mysql-master /bin/bash
  ```

  進入到 mysql 內。
  ```
  mysql -u root -p
  ```
  然後輸入密碼，如果是使用範例的話，密碼是 test。

  以下指令 mysql> 表示進入 mysql 內下達的指令，實際輸入時請忽略。

  此時執行 
  ```
  mysql> SHOW DATABASES;
  ```
  應該會看到，事先建立好的user_db。
  ```
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | user_db            |
  +--------------------+
  ```

  建立給從資料庫使用的使用者，並指定使用 SHA-256 來加密及驗證密碼
  ```
  mysql> CREATE USER 'second'@'%' IDENTIFIED WITH caching_sha2_password BY 'test';
  ```

  設定使用者的權限。
  ```
  mysql> GRANT REPLICATION SLAVE ON *.* TO 'second'@'%';
  ```

  重新載入權限，mysql 才能套用上面所設定的使用者權限。
  ```
  mysql> FLUSH PRIVILEGES;
  ```

  確認設定是否生效。
  ```
  mysql> SHOW MASTER STATUS;
  ```
  應該會看到類似以下的輸出結果。
  ```
  +------------------+----------+--------------+------------------+-------------------+
  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +------------------+----------+--------------+------------------+-------------------+
  | mysql-bin.000001 |      860 |              |                  |                   |
  +------------------+----------+--------------+------------------+-------------------+
  ```
  請記下 File 跟 position 的內容，會在從資料庫的設定使用到。

### 2. 備份主資料庫資料到從資料庫
  先將 DB 設定為 read only，如果是正在線上的系統，此舉會影響到所有寫入功能，務必小心。 
  ```
  mysql> FLUSH TABLES WITH READ LOCK;
  ```

  退出 mysql 到 master-sql 的容器下達指令，備份資料。
  ```
  mysqldump -uroot -p  --all-databases > replication/master.sql
  ```
  會要求輸入密碼，密碼是 test。

  在容器外，將 master-database 內的 replication 的備份資料複製到 slave-database 底下。

  ```
  sudo cp master-database/replication/master.sql slave-database/replication/
  ```

  Q: 為什麼要額外掛載 replication?

  A: 額外掛載 replication 是為了備份資料的移轉，也可以透過其他的方式將檔案移道從資料庫底下，並不限制一定只能照此方法執行。

  進入到從資料庫的容器內。
  ```
  docker exec -it mysql-slave /bin/bash
  ```
  
  將備份的資料匯入到從資料庫內，也可以看一下 master.sql 的內容，會發現裡面其實是 SQL 指令的腳本，所以這個動作其實是在從資料庫上 replay 主資料庫的操作。 
  ```
  mysql -u root -p --default-character-set=utf8 < replication/master.sql
  ```
  會要求輸入密碼，密碼是 test。

  進入到 mysql 內。
  ```
  mysql -u root -p
  ```
  然後輸入密碼，如果是使用範例的話，密碼是 test。

  此時執行 
  ```
  mysql> SHOW DATABASES;
  ```
  會看到，成功備份了主資料庫的 user_db。
  ```
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | user_db            |
  +--------------------+
  ```

### 3. 設定從資料庫

  首先需要設定 my.cnf，新增以下內容，或著也可以直接使用範例中的 my.cnf 移除掉註解。

  ```
  # 設定 server 唯一id
  server-id=2
  # 設定只能讀取
  read_only=ON
  # 設定連 root 也只能讀取
  super_read_only=ON
  ```
  `super_read_only` 這項設定端看使用者需求，如果只設定 `read_only` 只會阻擋普通使用者的寫入，並不會阻擋 root 的寫入。

  將修改後的內容新增到容器內，因為範例建立的容器沒有 vi 或是 nano。
  ```
  docker cp ./my.cnf mysql-slave:/etc/mysql/my.cnf
  ```

  重啟容器套用設定。
  ```
  docker restart mysql-slave
  ```

  進入到 mysql 內。
  ```
  mysql -u root -p
  ```
  然後輸入密碼，如果是使用範例的話，密碼是 test。
  
  根據 `1. 設定主資料庫` 紀錄的內容設定
  ```
  mysql> CHANGE MASTER TO
  MASTER_HOST='mysql-master',
  MASTER_PORT=3307,
  MASTER_USER='second',
  MASTER_PASSWORD='test',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=860,
  GET_MASTER_PUBLIC_KEY = 1;
  ```
  * MASTER_HOST : 是設定主資料庫的ip，但這裡是使用docker-compose內的網路，所以根據 service 名稱設定。
  * MASTER_PORT : 設定主資料的port，如果沒有設定，預設是走3306，所以如果沒有修改主資料庫的 port 是不用設定此項。
  * MASTER_USER : 在 `1. 設定主資料庫` 的主資料庫中設定給從資料庫使用的使用者。
  * MASTER_PASSWORD : 在 `1. 設定主資料庫` 的主資料庫中設定給從資料庫使用的使用者密碼。
  * MASTER_LOG_FILE : 在 `1. 設定主資料庫` `SHOW MASTER STATUS` 顯示的 File，指定從哪個 binary log 檔案開始同步。
  * MASTER_LOG_POS : 在 `1. 設定主資料庫` `SHOW MASTER STATUS` 顯示的 Position，指定 binlog 中的 offset 位置，從該位置開始複製資料。
  * GET_MASTER_PUBLIC_KEY : 設定啟用 RSA 公鑰來交換密碼。

  更詳細的設定可以參考 : [CHANGE MASTER TO Statement](https://dev.mysql.com/doc/refman/8.0/en/change-master-to.html)

  啟動從伺服器的同步機制
  ```
  mysql> START SLAVE;
  ```
  要注意的是 SQL 版本 如果是 8.0.22 以上的話，`START SLAVE` 被改為 `START REPLICA`，所以需要注意 SQL 版本。

  [詳細請參考](https://dev.mysql.com/doc/refman/8.0/en/start-slave.html)

  確認從伺服器的狀態
  ```
  mysql> SHOW SLAVE STATUS\G;
  ```

  看到以下內容顯示 YES 表示成功了。
  ```
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
  ```

### 4. 設定主資料庫

  記得要解除 `2. 備份主資料庫資料到從資料庫` 主資料庫 的 read only。

  ```
  mysql> UNLOCK TABLES;
  ```

  可以嘗試在主資料庫 insert 一筆資料，確認從資料庫是否也有更新。

  ```
  mysql> USE user_db;
  INSERT INTO users (user_name, mail) VALUES ('test', 'test@mail.com');
  ```

## 參考:
* 內行人才知道的系統設計面試指南
* https://homuchen.com/posts/what-and-why-database-replication-advantage-and-disadvantage/
* https://ithelp.ithome.com.tw/m/articles/10267454
* https://medium.com/dean-lin/%E6%89%8B%E6%8A%8A%E6%89%8B%E5%B8%B6%E4%BD%A0%E5%AF%A6%E4%BD%9C-mysql-master-slave-replication-16d0a0fa1d04
* https://blog.toright.com/posts/5062/mysql-replication-%E4%B8%BB%E5%BE%9E%E5%BC%8F%E6%9E%B6%E6%A7%8B%E8%A8%AD%E5%AE%9A%E6%95%99%E5%AD%B8
* https://ithelp.ithome.com.tw/articles/10255421
* https://dev.mysql.com/doc/refman/9.3/en/replication-implementation.html
* https://dev.mysql.com/doc/refman/8.0/en/replication-howto.html
* https://dev.mysql.com/doc/refman/8.0/en/change-master-to.html
* https://dev.mysql.com/doc/refman/8.4/en/caching-sha2-pluggable-authentication.html
* https://dev.mysql.com/doc/refman/8.0/en/start-slave.html