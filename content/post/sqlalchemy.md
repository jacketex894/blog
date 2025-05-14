+++
date = '2025-05-13T00:23:14+08:00'
draft = false
title = 'Sqlalchemy'
+++
## 前言
學到了 connection pool，想說要來 optimize 自己的 side project。

沒想到研究了半天，原本使用的 Sqlalchemy 就已經有內建有做了。

因此做個筆記紀錄一下，也給自己一個警惕，比起請 AI 直接開始改寫，不如先去讀一下官方文件，養成閱讀官方文件的習慣。

在使用 AI 上也要更小心自己不夠了解的部分，避免被幻覺誤導。

## Connection pool
每當跟資料庫要求 CRUD 操作時，都需要跟資料庫建立連線、進行操作、關閉連線。
建立連線跟關閉連線的動作對資料庫來說是很消耗資源的，connection pool 就是為了解決消耗資源的問題。

connection pool 相當於連線的快取，程式會事先建立固定的連線數量，當要跟資料庫要求執行操作時，會從 connection pool 中取得已事先建立好的連線，完成操作後再歸還。

透過重複使用已建立的連線，來減少每次操作都建立連線再關閉連線所消耗的資源。

以下會先事先建立好資料庫，用於比較有 connection pool 跟沒有 connection pool
查詢的差異。

## mysql.connector
mysql.connector 是由 MySQL 官方提供的 lib。
會先利用此 lib 的方法來連接資料庫做查詢。

mysql.connector 的使用方法
- 透過 mysql.connector.connect 來設定基本設定以及建立連線，這裡不設定 port 是因為在測試範例中使用的是 mysql 的 default port 3306。
- 透過 connection.cursor() 建立 cursor，透過 cursor 跟 DB 溝通(傳遞 SQL 指令跟取回結果)。
### without_pool
每次查詢都會建立連線查詢，再關閉連線。
```
def query_with_out_pool():
    connection = None
    cursor = None
    try:
        #建立連線
        connection = mysql.connector.connect(
            host=DB_HOST,
            user=DB_USER,
            passwd=DB_PASSWORD,
            database=DB_NAME
        )
        if connection.is_connected():
            cursor = connection.cursor()
            query = f"""SELECT name 
            FROM {TABLE_NAME}
            WHERE name = 'test_55'"""
            cursor.execute(query)
            records = cursor.fetchall()
    except Error as e:
        print(f"Database operation error{e}")
    finally:
        if connection and connection.is_connected():
            if cursor:
                cursor.close()
            connection.close()
```
我使用 line_profile 來逐行檢視執行100次所消耗的資源。
會得到以下的輸出
```
Wrote profile results to query.py.lprof
Timer unit: 1e-06 s

Total time: 2.9325 s
File: query.py
Function: query_with_out_pool at line 16

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    16                                           @profile
    17                                           def query_with_out_pool():
    18       100         56.4      0.6      0.0      connection = None
    19       100         24.8      0.2      0.0      cursor = None
    20       100         15.0      0.2      0.0      try:
    21       200    2733927.1  13669.6     93.2          connection = mysql.connector.connect(
    22       100         28.4      0.3      0.0              host=DB_HOST,
    23       100         21.5      0.2      0.0              user=DB_USER,
    24       100         17.5      0.2      0.0              passwd=DB_PASSWORD,
    25       100         25.9      0.3      0.0              database=DB_NAME
    26                                                   )
    27       100      29763.2    297.6      1.0          if connection.is_connected():
    28       100      31832.3    318.3      1.1              cursor = connection.cursor()
    29       200         99.7      0.5      0.0              query = f"""SELECT name
    30       100         62.7      0.6      0.0              FROM {TABLE_NAME}
    31                                                       WHERE name = 'test_55'"""
    32       100      54132.4    541.3      1.8              cursor.execute(query)
    33       100       3652.7     36.5      0.1              records = cursor.fetchall()
    34                                               except Error as e:
    35                                                   print(f"Database operation error{e}")
    36                                               finally:
    37       100      29637.8    296.4      1.0          if connection and connection.is_connected():
    38       100         59.0      0.6      0.0              if cursor:
    39       100        622.1      6.2      0.0                  cursor.close()
    40       100      48518.0    485.2      1.7              connection.close()

Total time: 2.9357 s
File: query.py
Function: test at line 84

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    84                                           @profile
    85                                           def test():
    86       101         51.7      0.5      0.0      for i in range(0,100):
    87       100    2935643.6  29356.4    100.0          query_with_out_pool()
```
可以發現有 93.2% 的時間都在建立連線。

### with_pool
mysql.connector 也有提供建立 pool 的方法，可以先建立連線，要使用連線的時候再從 pool 取。
```
#事先建立好連線
connection_pool = mysql.connector.pooling.MySQLConnectionPool(
            host=DB_HOST,
            user=DB_USER,
            passwd=DB_PASSWORD,
            database=DB_NAME,
            pool_size = 5,
        )

def query_with_pool():
    connection = connection_pool.get_connection()
    cursor = None
    try:
        if connection.is_connected():
            cursor = connection.cursor()
            query = f"""SELECT name 
            FROM {TABLE_NAME}
            WHERE name = 'test_55'"""
            cursor.execute(query)
            records = cursor.fetchall()
    except Error as e:
        print(f"Database operation error{e}")
    finally:
        if connection and connection.is_connected():
            if cursor:
                cursor.close()
            connection.close()
```
轉用 pool 後可以發現總消耗時間減少了，並且取用連線的時間也比建立連線短。
```
Wrote profile results to query.py.lprof
Timer unit: 1e-06 s

Total time: 0.212316 s
File: query.py
Function: query_with_pool at line 49

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    49                                           @profile
    50                                           def query_with_pool():
    51       100      21758.1    217.6     10.2      connection = connection_pool.get_connection()
    52       100         32.5      0.3      0.0      cursor = None
    53       100         15.4      0.2      0.0      try:
    54       100      19519.5    195.2      9.2          if connection.is_connected():
    55       100      21222.9    212.2     10.0              cursor = connection.cursor()
    56       200         86.4      0.4      0.0              query = f"""SELECT name
    57       100         40.6      0.4      0.0              FROM {TABLE_NAME}
    58                                                       WHERE name = 'test_55'"""
    59       100      40642.8    406.4     19.1              cursor.execute(query)
    60       100       2980.9     29.8      1.4              records = cursor.fetchall()
    61                                               except Error as e:
    62                                                   print(f"Database operation error{e}")
    63                                               finally:
    64       100      19098.2    191.0      9.0          if connection and connection.is_connected():
    65       100         54.1      0.5      0.0              if cursor:
    66       100        522.8      5.2      0.2                  cursor.close()
    67       100      86341.6    863.4     40.7              connection.close()

Total time: 0.214138 s
File: query.py
Function: test at line 84

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    84                                           @profile
    85                                           def test():
    86       101         35.0      0.3      0.0      for i in range(0,100):
    87       100     214103.5   2141.0    100.0          query_with_pool()
```
## sqlalchemy
sqlalchemy 是一個 ORM (Object Relational Mapper) library，讓使用者透過物件導向的方式去編寫程式碼，ORM 會再轉譯成 SQL 指令去執行。

ORM 可以防止 SQL 注入攻擊，但由於轉譯的關係，使用 ORM 也會比原生的 SQL 速度要更慢。

sqlalchemy default 就有 connection pool。

以下是一個簡單的 sqlalchemy 範例
```
from sqlalchemy import Column, Integer, String,create_engine
from sqlalchemy.orm import declarative_base
from sqlalchemy.orm import sessionmaker

#建立一個 engine 透過 pymysql 驅動連線
engine = create_engine(f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{3306}/{DB_NAME}")

# 建立一個 ORM 的基礎類別，所有資料表對應的類別都要繼承這個 Base
Base = declarative_base()

# 建立一個 Session 類別，用來建立與資料庫互動的 session（會話）
Session = sessionmaker(bind=engine)

# 定義一個 ORM 類別 User，對應資料庫中的 'test_member' 資料表
class User(Base):
    __tablename__ = 'test_member'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(255),nullable=False)

def query_with_pool_sqlalchemy() -> User|None:

        # 建立一個資料庫會話
        session = Session()
        query_user = session.query(User).filter(User.name == "test_55").first()

        # 關閉 session（釋放資源）
        session.close()
        return query_user
```
sqlalchemy 在使用 pool 的方式比較不同，並非是預先建立好連線。

而是首次使用的時候才建立連線，而 session.close 則是將資源歸還給 pool，並非關閉連線。

可以發現使用 sqlalchemy 的時間與 mysql.connector 使用 pool 的時間接近，但稍慢一點，這部分的差異可能就是在於轉譯 SQL 指令所花費的時間。

可以發現 63.3% 的時間都在 query_uesr 設定 fliter 條件去搜尋這段。
```
Wrote profile results to query.py.lprof
Timer unit: 1e-06 s

Total time: 0.229665 s
File: query.py
Function: query_with_pool_sqlalchemy at line 78

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    78                                           @profile
    79                                           def query_with_pool_sqlalchemy() -> User|None:
    80       100       3421.0     34.2      1.5          session = Session()
    81       100     145461.8   1454.6     63.3          query_user = session.query(User).filter(User.name == "test_55").first()
    82       100      80724.5    807.2     35.1          session.close()
    83       100         57.7      0.6      0.0          return query_user

Total time: 0.231337 s
File: query.py
Function: test at line 84

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    84                                           @profile
    85                                           def test():
    86       101         29.0      0.3      0.0      for i in range(0,100):
    87       100     231308.2   2313.1    100.0          query_with_pool_sqlalchemy()
```
sqlalchemy 雖然在預設就有使用 pool，但也有提供使用者自行設定 pool 的參數。
    * pool_size    : pool 內有多少連線需要保持
    * max_overflow : 設定 pool 中最多可以有多少連線
    * pool_recycle : 設定多少秒後回收連線
    * pool_timeout : 設定多少秒後放棄從 pool 中取得連線
max_overflow 的用意是當連線都在被使用的時候，最多可以多開到多少連線。
以下是使用例子，作為 `create_engine` 的參數傳入即可。 
```
engine = create_engine(f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{3306}/{DB_NAME},pool_size=5, max_overflow=10")
```
更詳細的使用設定可以參考[官方文件](https://docs.sqlalchemy.org/en/20/core/engines.html#sqlalchemy.create_engine)
## Reference  
https://en.wikipedia.org/wiki/Connection_pool
https://vocus.cc/article/5f800406fd89780001365d17
https://dev.mysql.com/doc/connector-python/en/connector-python-example-cursor-select.html
https://www.runoob.com/python3/python-mysql-connector.html
https://dev.mysql.com/doc/connector-python/en/connector-python-connection-pooling.html
https://github.com/pyutils/line_profiler
https://reginapanpan.medium.com/sqlalchemy-%E6%98%AF%E4%BB%80%E9%BA%BC-%E8%88%87-orm-%E7%9A%84%E9%97%9C%E4%BF%82-a57b16053aec
https://www.explainthis.io/zh-hant/swe/orm-intro
https://docs.sqlalchemy.org/en/20/core/pooling.html
https://docs.sqlalchemy.org/en/20/core/engines.html#sqlalchemy.create_engine