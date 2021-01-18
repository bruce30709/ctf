###### tags: `ctf`
# Bonus write_up

## ID
bruce30709
參加balsn ctf ，沒有解出來但有推理過程，只有拿到歡迎的flag
![](https://i.imgur.com/deAF1aP.png)

## babyrev
本題是scala語言寫成的程式，本題只有給.calss檔案，所以就使用JD-GUI來靜態的觀察，其中比較有興趣的是這個地方，他把一大串的數字存在a名稱的array中。
![](https://i.imgur.com/W7pk1kq.png)
然後b名稱的array是放broken.apply(60107) % (long)Math.pow(2.0D, 62.0D)).toByteArray()
![](https://i.imgur.com/mtgIrxP.png)
然後最後會把這2個東西做xor就得到flag了
![](https://i.imgur.com/Ha9pw5k.png)

這題是我第一次看scala的calss所以花了很多時間，當初是想說broken.apply會去call一個flatmap的function，所以如果能知道flagmap在做甚麼，然後回傳的值經過% (long)Math.pow(2.0D, 62.0D)).toByteArray()後再跟a做xor就可以拿到flag了，但那時候沒有搞懂flatmap到底做了甚麼，所以就卡在這裡了~
![](https://i.imgur.com/XqfdMYY.png)


