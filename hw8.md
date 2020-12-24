###### tags: `ctf`
# Hw8 write_up

## ID
bruce30709
## SecureContainProtect
本題一開始不知道洞在哪，就輕鬆愉快的開始解數獨，想說看看會怎樣，結果得知最後要再輸入一個Z後，還要再輸入關鍵還要再輸入關鍵的action code，所以去看他Z之後做了甚麼，在main有找到他call了一個function。
![](https://i.imgur.com/dmTMI4G.png)
可以看出前2個for迴圈感覺是在驗證數獨結果對不對，所以專心看第三個function，發現她會去一個6015大小的memory中拿值出來跟我們的input做xor，然後會一陸把xor出來的東西加總，最後去比較這個數值是否=257498，但實在沒想到怎麼反推，就寫了個程式去爆破各個ascii code，我的猜測是這樣，會不會他都用同一個ascii 數值來組成ascii art，那我爆破不就可以看出圖形了嗎?所以就這樣去嘗試。
![](https://i.imgur.com/7yfX7fj.png)
於是寫了個c++ a.cpp 來爆破，一樣照著ida去刻，把table的東西和soduku的結果還有我們的輸入去xor，然後指印看得出來的符號，其他印空白。
![](https://i.imgur.com/MvbDlKf.png)
然後去觀察生成的圖形，其實有些都滿有flag的影子的，但看不清楚，然後就一直往下翻，翻到第16個，發現了action code!!
![](https://i.imgur.com/xi9jDuF.png)
輸入後就得到ascii art flag了~
![](https://i.imgur.com/XkSN6oo.png)




## wishMachine
本題因為是static link的關係?所以會找不到symbol，但好像有看到start，就從start往下去尋找，發現一個看起來很關鍵的code，所以就開始朝他研究，又發現前面好像只是在做printf和scanf，所以去研究sub_400E0A這個function
![](https://i.imgur.com/qhN5MW3.png)
進去function後，觀察程式碼後，把變數名稱改一改，發現是function pointer，然後會去table的位置，我是去看memory，發現有一個值為0x3FA21E的數字存在，然後會把它加上i1，初始值是0x6FB8，相加剛好等0x4011D6，所以就猜是去叫sub_4011D6這個function，隨著next值會增加，會去table裡面每10個Q_word大小的memory去依序拿fun_ptr(也就是i1+第1個Q_word的值),i2,i3,i4,i5出來。
![](https://i.imgur.com/ISjICb4.png)
反覆去觀察sub_4011D6和sub_400E0A以及table的memory，結果發現i2是輸入的str的index，i3是之後進function會跑的次數，i4和i5分別為要帶進function進行比較的變數，那大概也清楚程式的運作流程，可以開始看function去回推回去，我們要符合function中的v1，也就是function會去我們的輸入的70大小的str的第(i2)的位置和位置+1(不一定有)，取出值後經過運算，要和i4,i5相同，經過觀察i3的值發現，i5不一定會有，如果i3是1則function只會跑一次。
![](https://i.imgur.com/q4A89mY.png)
經過計算後發現，會呼叫的function只有5個，所以寫了python去回推
1. sub_4011D6() 做fibonacci，暴力去算但要注意的是python不會overflow，所以每次overflow的時候，都要從0開始(也就是減去0x100000000)
    ![](https://i.imgur.com/EjXOiqf.png)
2. sub_401138() 做奇數位和偶數為要減去不同的值的運算，這裡要注意的點是要轉成signint去運算，先減去88035316後，再去看%(30600+120)會不會有餘數，如果有那代表是奇數，所以除30720去回推時，除了\*2(也就是看有幾對偶+奇數pair，也就是for迴圈跑了幾次)之外，因為是奇數的關係，所以再多跑了一次，回傳值要加1。
    ![](https://i.imgur.com/FQ2RIzF.png)

3. sub_4010C8() 做xor，回推方法很簡單，直接做xor
    ![](https://i.imgur.com/8YRynRB.png)

4. sub_40102D() 做奇數位和偶數位要加不同的值的運算，想法和2.相同
    ![](https://i.imgur.com/MHdf5Nt.png)

6. sub_400FBE() 做*135，回推方法就/135回去
    ![](https://i.imgur.com/07EcriQ.png)

最後只要依序每10個去讀讀table裡面的東西，去產生應該放進index中的輸入，再放進dict排序一下，跑完1000次後，就可以得出flag了，我是在commandline輸入

python3 exploit.py | ./wishMachine

就拿到flag啦!!!

![](https://i.imgur.com/awWwR3f.png)
## Curse
本題有用PEbear去看，再重聽了助教上課講的內容，大概知道他問題出在哪，就是他的NT header被搞爆了，所以ida變智障~
![](https://i.imgur.com/DCwXkvP.png)
解法式要想辦法去重建header，但時間不夠所以沒有想到怎麼解QQ






