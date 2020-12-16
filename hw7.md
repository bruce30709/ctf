###### tags: `ctf`
# Hw7 write_up

## ID
bruce30709
## Going Crazy
本題是golang寫成的題目去做反編譯，但是golang語言不會儲存symbol，所以要使用golanghelper 才能看到function出現symbol
![](https://i.imgur.com/T8bcMFB.png)
之後就可以在ida pro按F5看組語，首先觀察main.main後可以發現牠會有2層的指標，並且會並且會把總長存在v27[1]，而且發現她會去看第一個字和最後一個字是否是"x"，而且總長度要>3，故可以推論出要輸入x.......x格式的字串，但發現中間不管輸入甚麼都會噴錯，所以去用gdb觀察。
![](https://i.imgur.com/uX4mwOY.png)
之後看程式碼有看出來，他會把第一個字元split掉，之後就一直嘗試餵不同的符號進去，直到後來在stack有發現一串奇怪的字串，丟進去後發現，","之後的字串被切開了，所以猜是用"，"去分隔字串

![](https://i.imgur.com/mfJlD6g.png)

還有就是有在程式碼裡面發現有去用到"，"，所以就拿"，"去gdb看看嘗試。

![](https://i.imgur.com/SOhjsHs.png)
結果發現用x\,\,\,\,\,\.\.\.\.\,\,\,\,x15個逗點後，可以進checkinput的第二層loop。

![](https://i.imgur.com/wCOyUWY.png)
我是分別下了3個斷點，分別斷在checkinput的前後2個if判斷式，還有beuz裡面裡面從rchvf出來那，去觀察register和satck，這裡就叫他們$\ b1,b2,b3$，我發現$b1$會去檢查類似逗點的長度，之後寫到後面才知道那是**前面有幾個逗點**，b2會去檢查**rchvf funct出來後生成的數字**是否跟記憶體位置裡面的相同，b3則是檢查餘數是否會==1，這也是之後看rchvf程式碼看出來的。
![](https://i.imgur.com/aG4bOqm.png)
這張是挖出來的**rchvf funct出來後生成的數字**
![](https://i.imgur.com/8TH8qdp.png)
這張是挖出來的**前面有幾個逗點**
![](https://i.imgur.com/Gyx7Pra.png)
最後也就是最關鍵的部分了，就是要想辦法生成**rchvf funct出來後生成的數字**，我去看rchvf後，發現她會做輾轉相除法，而第一個除數會去被經過最後一行的運算之後，形成一個負數，之後會把這個數加4224019091，存進記憶體中。
故我用c++重新刻了這個function，之後用4224019091去減我們生成的值，之後會發現，有些值找不到，後來想到可能是這個值一開始就太小了，所以加過不只一次4224019091，但其實感覺他是生成的時候就直接overflow成一個正數了，果然沒猜錯，所以就直接去找那個數字，我是使用aaa.cpp去生成**rchvf funct出來後生成的數字**，有附在檔案裡，也有寫成aaa.txt，可以直接搜尋ex.4224019091-1511754201=2712264890 就去aaa.txt找這個數字，但會是負數，那如果找不到，就直接搜尋沒有用4224019091減過的數字。 
![](https://i.imgur.com/ecQEq1t.png)
之後把找到的數字全部列在紙上，左邊是輸入後可以過各個**rchvf funct出來後生成的數字**檢查的數字，右邊是把第二個表重新列一次方便對照，最後只要把右邊的箭頭對應的數字，可以想像成新的index，去查相對的數字，之後依序放進input， ex.0隊到的是25，在去查25隊到的值是70，則70即是第一個輸入，以此類推即可走完36個loop~~
![](https://i.imgur.com/o3E2Axg.jpg)
最後的輸入長這樣
![](https://i.imgur.com/GHism75.png)
感覺就是ascii啊，所以直接翻，flag就出來了!
![](https://i.imgur.com/YXv3mIU.png)






