# Liber語言文檔

Liber語言，全稱「Liber是世界上最好的語言」，是Librian使用的劇本語言。

## 基本規則

大部分情況下Liber語言是按行解析的，換行即爲語句的終結。

預處理時，劇本被按行分割。

之後，編譯器將所有行使用各個正則表達式匹配，分別得到它們的類型和參數，包裝成dict。

最後，這些dict被按順序組成list，交給Librian虛擬機解釋。

## 語法

### 人物對話和表情
```
潘大爺 (笑)「你好啊！」
```

人物對話的設計取材於慣用的劇本格式。

格式是 `人名` `空格` `(表情)` `[特效]` `「話語」`。  
`表情` 和 `特效` 是可選項。

如果沒有對話，格式是 `人名` `空格` `(表情)` `[特效]`。    
`特效` 是可選項，但 `表情` 不是。

之所以需要空格是爲了避免 `QED「495年的波纹」` 之類的。  
但實際上沒有什麼意義，還不如等出問題了再轉義。


### 插入代碼

    ```python
    print('你好啊！')
    ```

插入代碼的格式取材於Markdown，這語法幾乎一模一樣了<small>(笑)</small>。

格式是 `三個點` `代碼類型` `換行` `代碼內容` `換行` `三個點`  
`代碼類型` 和 `代碼內容` 是可選項。如果沒有 `代碼內容` 就不需要第二個 `換行` 。

儘管Librian只支持三種代碼類型，實際上 `代碼類型` 的取值種類應當取決於實現。此外，插入代碼的語義並非總是立即執行，這也取決於實現。

### 函數調用

```
> BG 魔王城內
```

函數調用的寫法取材於Shell。

格式是 `>`(`"字符串"` 或 `字符串`)×n。  
即 `>` 後接着許多用 `空格` 隔開的字符串，帶有雙引號的字符串內部不會分割。

<small>(其實我也已經看不懂那代碼寫的是什麼了……)</small>

格式也可以是 `一個其他語言的函數調用` ，是否允許以及哪種語言都取決於實現。

### 插入圖

```
=== 第一章
```

插入圖的格式取材於Markdown的標題語法。  
雖然Markdown是寫在下面的，但是爲了解析方便於是就這樣做了。

格式是 `===` `圖片名` ，等號可以任意超過三個。

### 鏡頭

```
+ {潘大爺: [100, 120, 1.5]}
```

鏡頭的格式非常讓人迷惑。

它的格式其實是 `加號` `YAML對象` 。

這個YAML對象必須能被解析爲list或dict。  
list元素依次爲鏡頭中各個人物的名稱。  
dict鍵爲人物名稱，值爲人物在鏡頭中的位置。

<small>(之所以不是JSON是因爲JSON太難用。)</small>

### 選項

```
? 接受治療 -> 壞結局.liber
```

選項的格式取材於ink。

格式是 `?` `空格` `選項名` `->` `文件名` `,` `位置`

`文件名` 和 `位置` 都是可選項，沒有位置的話也不用寫逗號。

### 人物操作

```
@潘大爺 + 西裝
```

格式是 `@` `人物名` `操作符` `目標`。

這個 `@` 的意思其實不太對得上。

### 註釋

```
# 我覺得這段劇本還要修改。
```

註釋看起來像是Python或者Shell的註釋，其實不太一樣。

格式是 `#` `註釋內容`

註釋同樣是行解析的一部分，這意味着 `#` 必須在行首。

此外，註釋被丟棄不應該在編譯時而在運行時。

### 躍點

`* 起點`

格式是 `*` `躍點名`。

<small>(好像沒什麼可說的……)</small>

### 旁白

```
潘大爺來了！
```

旁白的格式其實是最簡單的，只要一行不能匹配以上任何一種，就會被當成旁白。

## 續行

在預處理時將需要續行的情況連成一行。

`人物對話` 和 `嵌入代碼` 在劇本中可能超過一行，因此它們會續行而不應該被分割。

因此還有三個續行規則。如果當前行能匹配到任意一個續行規則，就把下一行加入當前行，然後重新進行續行規則匹配。

+ 強制續行
:   只要以 `\` 結尾就行了。

+ 多行對話續行
:   對話匹配到了前面的部分，唯獨缺了最後的 `」` 的時候。

+ 代碼段續行
:   代碼段沒有遇到結尾的 `三個點` 的時候。


## 停滯點

停滯點是一個重要的概念。

當Librian虛擬機解析到劇本中的停滯點的時候，就會向前端發送一組狀態數據以更新畫面，之後需要點擊一下鼠標來繼續。

在語法上，只有 `旁白` 、 `人物對話` 、 `插入圖` 、 `選項` 是停滯點。

此外，並非所有選項都是停滯點。當多個選項同時出現時，只有最後一個是停滯點。

### 樣例

```liber
> BG 黑
> BG 白
> BG 黑
潘大爺 「我好暈啊。」
```

這是一個錯誤樣例，原本想要實現的是背景圖片發生「黑-白-黑」的過渡，然後 `潘大爺` 說他很暈。

但是實際上這並不能實現，背景總是黑色的。  
這是因爲這段劇本中只有 `人物對話` 一個停滯點，而之前三次調用 `BG` 沒有傳送到前端就互相覆蓋了。

<small>正確的做法應該是使用CSS動畫來實現……</small>

### 與嵌入代碼的相互作用

嵌入代碼並不是一個停滯點，但是也會有停滯的現象。這是因爲有一些函數會控制Librian虛擬機引起停滯。

`goto` 和 `call` 不會引發停滯，只有跳轉到目標劇本並到達下一個停滯點纔會停滯。因此如果你在副線程調用它，前端並不會變化。

`choice` 會引發停滯。在運行完一段嵌入的代碼時，如果有未處理的選項，爲了阻止讀取選項之後的內容，會發生停滯。

### 微妙的實現

即使在函數調用中播放了視頻，也不會停滯。  
雖然它看起來停滯了，實際上是因爲視頻實際上是和下一句同時發送到前端的，只是前端把他們錯開了。

