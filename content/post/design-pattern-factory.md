+++
date = '2025-05-09T17:11:07+08:00'
draft = false
title = 'Design Pattern Factory'
categories = [
  "Design Pattern"
]
+++
## 工廠模式
封裝建立物件的邏輯，讓使用者無須在意物件創建的相關邏輯(e.g. 讀取設定檔...)。

在工廠模式中，會將封裝建立物件邏輯的類別稱為工廠，而實際建立的物件稱為產品，就像普通人並不會知道工廠運作的邏輯，但知道工廠生產什麼產品一樣。
工廠模式的目的是讓使用者(需要物件的function或邏輯)，無需在意這些物件建立的相關邏輯。 

以下會以漫畫(comic)作為產品 來舉例說明，假設我有兩種漫畫，分別是 funny 跟 adventure。
我想要取得漫畫物件最簡單的方法是透過 if 來判斷該創立哪種物件。
```
from abc import ABC, abstractmethod
class Book(ABC):
    @abstractmethod
    def content(self) -> str:
        pass
    @abstractmethod
    def cost(self) -> int:
        pass

class FunnyComics(Book):
    def content(self) -> str:
        return 'This is funny comic'
    def cost(self) -> int:
        return 4
class AdventureComics(Book):
    def content(self) -> str:
        return 'This is adventure comic'
    def cost(self) -> int:
        return 3

def buy_book(book_type:str) -> Book:
    if book_type == "Funny Comic":
        book = FunnyComics()
    elif book_type == "Adventure Comic":
        book = AdventureComics()
    print(book.content())
    print(book.cost())

if __name__ == '__main__':
    buy_book("Funny Comic")
    buy_book("Adventure Comic")
```
但是會發現當漫畫的種類越來越多的時候，`buy_book` 內的邏輯會越來越多，並且會一直需要改動 `buy_book` 這個 `function`。
### 簡單工廠
所有的產品物件類別都由一個共同的工廠類別或介面來管理。

其實是將 if 等相關判斷式另外包進去一個類別做管理
```
class SimpleBookFactory():
    def create_book(self,book_type:str) -> Book:
        if book_type == "Funny Comic":
            book = FunnyComics()
        elif book_type == "Adventure Comic":
            book = AdventureComics()
        return book

def buy_book(book_type:str):
    book_factory = SimpleBookFactory()
    book = book_factory.create_book(book_type)
    print(book.content())
    print(book.cost())
```
這樣做的好處在於對於 `buy_book` 來說，不用在意 `FunnyComics` 跟 `AdventureComics` 這兩個類別的實作細節。

只需透過傳遞參數給 `SimpleBookFactory` 即可。

但是這樣做依然有當新增類別時需要修改到 `SimpleBookFactory` 這個問題。

```
from abc import ABC, abstractmethod
class Book(ABC):
    @abstractmethod
    def content(self) -> str:
        pass
    @abstractmethod
    def cost(self) -> int:
        pass

class FunnyComics(Book):
    def content(self) -> str:
        return 'This is funny comic'
    def cost(self) -> int:
        return 4
class AdventureComics(Book):
    def content(self) -> str:
        return 'This is adventure comic'
    def cost(self) -> int:
        return 3

class SimpleBookFactory():
    def create_book(self,book_type:str)-> Book:
        if book_type == "Funny Comic":
            book = FunnyComics()
        elif book_type == "Adventure Comic":
            book = AdventureComics()
        return book
    
def buy_book(book_type:str):
    book_factory = SimpleBookFactory()
    book = book_factory.create_book(book_type)
    print(book.content())
    print(book.cost())

if __name__ == '__main__':
    buy_book('Funny Comic')
    buy_book('Adventure Comic')
```
### 工廠方法
設計工廠介面，產生產品物件由根據工廠介面建立的子類別來管理。

首先先來將簡單工廠的設計轉為工廠模式的設計。

先定義一個工廠類別的介面，並且都應該返回一個產品類別的物件。
```
class BookFactory(ABC):
    @abstractmethod
    def create_book(self) -> Book:
        pass
```
根據這個介面，來修改原本的 `buy_book`，對於 `buy_book` 來說，只需知道傳遞進來的參數是 `BookFactory` 類別即可。

所以除非是更改到了 `BookFactory` 這個介面，否則不會因為新增了工廠或是產品類別而需要更改到 `buy_book`。
```
def buy_book(factory:BookFactory):
    book_factory = factory()
    book = book_factory.create_book()
    print(book.content())
    print(book.cost())

```
接著設計工廠類別
```
class FunnyBookFactory(BookFactory):
    def create_book(self)-> Book:
        return FunnyComics()

class AdventureBookFactory(BookFactory):
    def create_book(self)-> Book:
        return AdventureComics()
```
相對於簡單工場類別，假設我今天需要新增一個新的 `Book` 類型，我只須新增一個工廠類別與產品類別就可以。

不必擔心新增了 `Book` 類型需要去改動到原本的工廠類別，不像簡單工場新增一個產品就需要更改到  `SimpleBookFactory` 類別。

`buy_book` 一樣不用在意如何去建立產品物件，因為這些都由各自對應的工廠負責。

e.g. 假設要新增一個 love comic，只需新增如下
```
class LoveComics(Book):
    def content(self) -> str:
        return 'This is love comic'
    def cost(self) -> int:
        return 6

class LoveComicsFactory(BookFactory):
    def create_book(self)-> Book:
        return LoveComics()
```
不會更改到 `buy_book`，甚至先前建立的 `FunnyComicsFactory` 或是 `AdventureComicsFactory` 類別。

```
from abc import ABC, abstractmethod
class Book(ABC):
    @abstractmethod
    def content(self) -> str:
        pass
    @abstractmethod
    def cost(self) -> int:
        pass

class FunnyComics(Book):
    def content(self) -> str:
        return 'This is funny comic'
    def cost(self) -> int:
        return 4
    
class AdventureComics(Book):
    def content(self) -> str:
        return 'This is adventure comic'
    def cost(self) -> int:
        return 3
    
class LoveComics(Book):
    def content(self) -> str:
        return 'This is love comic'
    def cost(self) -> int:
        return 6
    
class BookFactory(ABC):
    @abstractmethod
    def create_book(self,book_type:str) -> Book:
        pass

class FunnyBookFactory(BookFactory):
    def create_book(self)-> Book:
        return FunnyComics()

class AdventureBookFactory(BookFactory):
    def create_book(self)-> Book:
        return AdventureComics()
    
class LoveBookFactory(BookFactory):
    def create_book(self)-> Book:
        return LoveComics()

def buy_book(factory:BookFactory):
    book_factory = factory()
    book = book_factory.create_book()
    print(book.content())
    print(book.cost())

if __name__ == '__main__':
    buy_book(FunnyBookFactory)

    buy_book(AdventureBookFactory)

    buy_book(LoveBookFactory)
```
### 抽象工廠
設計工廠介面用於建立一系列相關的產品。

假設當類別變多了，例如今天每種書又都分為電子書跟實體書，前面使用的工廠方法所需設計的類別就可能會變很多。

這時就可以使用抽象工廠來解決這個問題。

先設計抽象類別，每個工廠會分別建立實體書跟電子書。
```
class BookFactory(ABC):
    @abstractmethod
    def create_book()-> Book:
        pass
    @abstractmethod
    def create_e_book()-> Book:
        pass
```

擴充電子書產品類別(`Book`)

```
class FunnyComicsEBook(Book):
    def content(self) -> str:
        return 'This is funny comic e-book'
    def cost(self) -> int:
        return 3
class AdventureComicsEBook(Book):
    def content(self) -> str:
        return 'This is adventure comic e-book'
    def cost(self) -> int:
        return 2
```

根據 `BookFactory` 設計子類別
```
class FunnyBookFactory(BookFactory):
    def create_book(self)-> Book:
        return FunnyComics()
    def create_e_book(self)-> Book:
        return FunnyComicsEBook()

class AdventureBookFactory(BookFactory):
    def create_book(self)-> Book:
        return AdventureComics()
    def create_e_book(self)-> Book:
        return AdventureComicsEBook()
```

```
from abc import ABC, abstractmethod

class Book(ABC):
    @abstractmethod
    def content(self) -> str:
        pass
    @abstractmethod
    def cost(self) -> int:
        pass

class FunnyComics(Book):
    def content(self) -> str:
        return 'This is funny comic'
    def cost(self) -> int:
        return 4
    
class FunnyComicsEBook(Book):
    def content(self) -> str:
        return 'This is funny comic e-book'
    def cost(self) -> int:
        return 3
    
class AdventureComics(Book):
    def content(self) -> str:
        return 'This is adventure comic'
    def cost(self) -> int:
        return 3

class AdventureComicsEBook(Book):
    def content(self) -> str:
        return 'This is adventure comic e-book'
    def cost(self) -> int:
        return 2

class BookFactory(ABC):
    @abstractmethod
    def create_book() -> Book:
        pass
    @abstractmethod
    def create_e_book()-> Book:
        pass

class FunnyBookFactory(BookFactory):
    def create_book(self)-> Book:
        return FunnyComics()
    def create_e_book(self)-> Book:
        return FunnyComicsEBook()

class AdventureBookFactory(BookFactory):
    def create_book(self)-> Book:
        return AdventureComics()
    def create_e_book(self)-> Book:
        return AdventureComicsEBook()

def buy_book(factory:BookFactory):
    book_factory = factory()
    book = book_factory.create_book()
    print(book.content())
    print(book.cost())
    e_book = book_factory.create_e_book()
    print(e_book.content())
    print(e_book.cost())

if __name__ == '__main__':
    buy_book(FunnyBookFactory)

    buy_book(AdventureBookFactory)
```
相對於工廠方法，抽象工廠其實更注重於產出彼此相關的產品。
工廠方法則是專門產生一個產品的工廠。

雖然不同的抽象工廠可能生產的產品具備相同功能，但它們的具體實作會根據工廠有所不同。

例如本篇例子`FunnyBookFactory` 跟 `AdventureBookFactory` 底下的產品都分成實體書跟電子書。
但是 `content` 會根據是  `FunnyBookFactory` 還是 `AdventureBookFactory` 而不同。

## 參考:
* 深入淺出設計模式
* https://skyyen999.gitbooks.io/-study-design-pattern-in-java/content/simpleFactory.html