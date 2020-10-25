###### tags: `ctf`
# Hw0 write-up
## ID
amamooo
## Web
本題的漏洞主要出現在server get ${cute} parameter時，只有做re的 true/false\$ 結尾的檢查，
造成client端可以插入一些語句進行攻擊。
故我插入的攻擊語句如下:
```htmlembedded=
false,"admin":true}&givemeflag=yes#givemeflag=yes
&data={"cute":true,"admin":true

```
首先，先繞過json格式，並且順便將admin設成true，
接著，用&帶入givemeflag參數，並設成yes，
最後，用#來讓後面的部分都變成只有在瀏覽器端才作用的部分，即可達成攻擊payload

**要注意的是，攻擊payload還要經過encode才可以在瀏覽器直接送出，我是使用url encoder，我是用api tester送，可以滿清楚看到傳送的情況。**
```htmlembedded=
https://owohub.zoolab.org/auth?username=123&cute=false
%2C%22admin%22%3Atrue%7D%26givemeflag%3Dyes%23givemeflag
%3Dyes%26data%3D%7B%22cute%22%3Atrue%2C%22admin%22%3Atrue

```

送出後，就可拿到flag了!。
## Pwn
本題為傳統的BOF題型，要蓋掉main的return address，換成func1的並且進入特定位置，繞過判斷式的檢查。

首先，先慢慢測試，發現32個symbol會成功蓋掉rbp，也就是return address。
![](https://i.imgur.com/zJbzn1U.png)

接著，寫python code作為攻擊，並利用pwntool的function可以輕鬆處理little endian的問題，
有觀察x64組語，原本想利用蓋掉$rax 0xcafecafecafecafe繞過判斷式，但後來發現直接跳後面0x401195，即可拿到shell。
![](https://i.imgur.com/0wWz95O.png)

最後remote連過去，並在/home/Cafeoverflow/flag中，就可以拿到flag了!
![](https://i.imgur.com/pdDdQN9.png)

## Misc
本題並沒有甚麼思路，有看懂程式，是有想到可能是要從cin的參數動手腳，因為cin.ignore()可以忽視某些symbol，但測試後還是沒解出來。

## Cryptography
本題有看懂程式，是要輸入一串長度為16的flag，之後進行加密，但我發現實力不足無法解出QQ。

## Reversing Engineering
觀察後發現是用c#寫的，故可以使用工具進行反編譯，我用的是ILSPY+reflexil，就可以在觀察程式碼的時候，直接進行修改，

```csharp=
list[13].Image = originalPicture[14];
list[14].Image = originalPicture[13];

```
我的修改方式是將originalPicture的offset改成相同的數字，就可以把原本錯亂的圖倒過來

![](https://i.imgur.com/u7umhJB.png)


之後再重新編成新的程式，上下移動一下，就可拿到flag了!

