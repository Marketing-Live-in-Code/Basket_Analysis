# 購物籃分析-part2【附Python程式碼】

在前篇文章「購物籃分析，最清楚概念-part1」中，已經介紹購物籃分析中，最重要的三個變數，接下來課程將以實作的方式，並且帶領您管理意涵。

## 實做案例
資料共有12,406筆，裡面只有購買清單（order_id）與產品名稱（product_name）兩個欄位，如下圖所示。
![資料內容](https://i.imgur.com/CoSg7sY.png)
apyori套件所要餵進去的格式非常簡單，只要將所有買過得商品整理成雙重陣列即可，例如範例：
```python
[
    ['啤酒','尿布','水果','餅乾'],
    ['水果','尿布','奶粉'],
    ['啤酒','尿布','水果','餅乾'],
    ['尿布']
]
```
因此利用for迴圈，以購買清單為單位，將每個購買清單所購買的商品放在同一個陣列，在將所有陣列集中在一個大陣列中，形成雙重陣列的形式，如下圖所示。
![整理完成的陣列](https://i.imgur.com/S25tuUH.png)
```python
record=[]
for i in tqdm(order_products['order_id'].value_counts().index):
    member = order_products[order_products['order_id']==i]
    record.append(member['product_name'].values.tolist())
```

## 購物籃分析
apriori套件中有兩個參數必須要設定，分別就是最小支持度（min_support）與最小提昇度（min_lift），雖然min_lift沒有強制要設定，但沒有設定的話分析出來的結果沒有意義。

min_support是設定大於多少的支持度要挑選出來，這個數字如果太小，會造成資料過多，相對資料太大，可能會沒有商品組合可以符合這個條件。min_lift就是之前所提到的，必須要設定大於1，否則商品就沒有正相關。
```python
association_rules = apriori(record, min_support=0.01, min_lift=1.0000001)association_results = list(association_rules)
```

以min_support設定為0.01，min_lift設定為1.0000001，出來的商品組合總共有25筆（如圖3所示），本文拆解第一個挑選出來的商品組合（如圖所示），來做詳細講解。

```python
#挑出第一筆資料
list1 = association_results[0]
```
![購物籃分析出來的結果](https://i.imgur.com/saor0fn.png)
![第一筆結果資料內容](https://i.imgur.com/z2rF7z0.png)

RelationRecord這個資料格式的取資料方式類似於陣列，因此使用中括號加上編號，即可取出想要的內容，如範例（圖下所示）中取出商品組合「Bag of Organic Bananas」+「Large Lemon」的支持度約為0.012。

![商品組合與支持度](https://i.imgur.com/nd7tNVY.png)

```python
print('商品組合： ')
print(list1[0])print('這個組合的支持度： ')
print(list1[1])
```

相信您還記的前面範例中，從不同的商品角度，得到的信心度與提昇度都不一樣，因此可以先取得Bag of Organic Bananas商品的角度，呈現的兩個數值為何（下圖所示）。
![以Bag of Organic Bananas產品為出發點的信賴度與提昇度](https://i.imgur.com/MaMbPo5.png)

```python
print('以Bag of Organic Bananas產品為出發點的信賴度： ')
print(list1[2][0][2])print('以Bag of Organic Bananas產品為出發點的提昇度： ')
print(list1[2][0][3])
```
換成從Large Lemon產品為出發點，得到的結果與前者完全不同（下圖所示）。
![以Large Lemon產品為出發點的信賴度與提昇度](https://i.imgur.com/vbXFr9i.png)
```python
print('以Large Lemon產品為出發點的信賴度： ')
print(list1[2][1][2])print('以Large Lemon產品為出發點的提昇度： ')
print(list1[2][1][3])
```

## 管理意含
回到我們最前面所討論的，我們最重要的是要討論「買A商品時買B商品的機率為多少」，也就是信賴度的部份，因此就以實際範例中的商品組合「Bag of Organic Bananas」+「Large Lemon」而言：

> 買Bag of Organic Bananas後，買Large Lemon的
機率約為11.02%

> 買Large Lemon後，買Bag of Organic Bananas的
> 機率約為17.94%

聰明的您馬上就知道，行銷資源要最先投放在購買Large Lemon的消費者，並且推薦他們購買Bag of Organic Bananas這個商品，如此明確又落地的方法，還不快手刀實做。

### 您可能會想問
> 購買組合「互相的購買機率」怎麼會不同？

這個原因在可能是您在常理與演算法之間有所衝突，那換個比喻。會買球針（幫籃球打氣的針）的消費者一定有買（過）籃球，但有買籃球的消費者，不一定有買過球針（跟別人借就好，不一定要人手一根針）。

> 要如何產出精準名單？
![精準名單](https://i.imgur.com/zcvllPv.png)

只需要將ordered_statistics欄位中的內容進行「爆破」即可，利用explode這個方法（pandas0.25版本以上），可以將陣列欄位中的數值分成多個row。經由「爆破」後，整理出以下欄位：

* A產品：以該欄位的產品為角度。
* B產品：A產品相對的商品組合對象。
* 信賴度：該商品組合的信賴度。
* 提昇度：該商品組合的提昇度。

整理出以下欄位後就簡單了，直接排序信賴度欄位，即可找到最高的推薦產品，因此如圖9中，信賴度最高為0.481481，代表購買「Honeycrisp Apple」的消費者，有48%的機率購買「Banana」。依照這個流程，並考量自身的行銷預算，便可以明確的選擇要打廣告的受眾與商品。
