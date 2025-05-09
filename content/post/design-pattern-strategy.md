+++
date = '2025-05-09T11:28:40+08:00'
draft = false
title = 'Design Pattern Strategy'
+++
## 策略模式
將可互換的演算法封裝為獨立的類別，透過統一介面使用類別，使得變更行為時，不必因行為變化而重新修改類別本身。

以 python 舉個例子，假設我今天有個旅遊類別如下。
```
class Travel():
    def hotel(self):
        print('Staying at the Taiwan Hotel.')
    def transport(self,distance:int):
        print(f"Walk for {distance / 1} minutes.")
```
但是我目前有個需求是要新增一個新的交通方式:"開車"，他的時間計算跟原本的走路的計算方式不一樣。
那這時我們可以選擇:
* 繼承 Travel 類別，並複寫 transport function。
  * 需要為此撰寫一個新的類別，假設 hotel 不一樣也會需要複寫，當有幾種 hotel 跟幾種 transport 的組合，就需要寫多少類別，在維護上會是一大難題。
* 直接修改 transport
  * 透過參數來選擇使用哪種交通工具，但是會需要確認是否影響了既有功能，當功能是複雜功能時有可能造成無法預期的錯誤。

一開始使用策略模式設計類別就可以避免以上兩者的缺點，我們可以將 transport 會需要作的內容定義為介面。
```
from abc import ABC,abstractmethod

class TransportStrategy(ABC):
    @abstractmethod
    def transport(self,distance:int):
        pass
```
定義了每個 transport 都該有的行為後，就可以設計類別會直接執行行為而不用顧慮是哪種 transport。
可以重新定義類別為
```
class Travel():
    def __init__(self,transport_strategy:TransportStrategy):
        self.transport_handler = transport_strategy()
    def hotel(self):
        print('Staying at the Taiwan Hotel.')
    def transport(self,distance:int):
        self.transport_handler.transport(distance)
```
然後另外定義各種 TransportStrategy
```
class WorkTransport(TransportStrategy):
    def transport(self,distance:int):
        print(f"Walk for {distance / 1} minutes.")

class DriveTransport(TransportStrategy):
    def transport(self,distance:int):
        print(f"Walk for {distance / 10} minutes.")
```
這樣只需要在創建 Travel 類別的物件時，指定 TransportStrategy 就好，更甚至可以另外定義 function 去動態的修改 TransportStrategy。
可以在某些條件下從 work_transport 變為 drive_transport。
這樣子可以解決:
* 不必為了多種組合而去建立類別。
* 每個 TransportStrategy 都是獨立的，當我要新增新的 TransportStrategy 並不會影響到原有的 TransportStrategy 的程式碼。

```
from abc import ABC,abstractmethod

class TransportStrategy(ABC):
    @abstractmethod
    def transport(self,distance:int):
        pass
class WorkTransport(TransportStrategy):
    def transport(self,distance:int):
        print(f"Walk for {distance / 1} minutes.")

class DriveTransport(TransportStrategy):
    def transport(self,distance:int):
        print(f"Walk for {distance / 10} minutes.")

class Travel():
    def __init__(self,transport_strategy:TransportStrategy):
        self.transport_handler = transport_strategy()
    def hotel(self):
        print('Staying at the Taiwan Hotel.')
    def transport(self,distance:int):
        self.transport_handler.transport(distance)

if __name__ == '__main__':
    travel_handler = Travel(WorkTransport)
    travel_handler.transport(10)
    travel_handler = Travel(DriveTransport)
    travel_handler.transport(10)
```

## 參考:
* 深入淺出設計模式
* https://medium.com/bucketing/behavioral-patterns-strategy-pattern-483d074d046a
* https://medium.com/%E4%BA%BA%E7%94%9F%E7%9A%84%E5%90%84%E7%A8%AE%E5%8F%AF%E8%83%BD/%E6%B7%B1%E5%85%A5%E6%B7%BA%E5%87%BA%E8%A8%AD%E8%A8%88%E6%A8%A1%E5%BC%8F-design-pattern-%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F-1-strategy-pattern-8029f46659ef

