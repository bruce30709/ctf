###### tags: `ctf`
# Hw1 write-up
## ID
bruce30709
## POA
本題的漏洞是因為padding在加密的時可以利用XOR的方式反推，把iv、密文、和我們爆搜的bit可以在送request時，去通過padding error的檢測。

觀察題目跟範例程式碼發現是padding的方式不同，是在最後加一個byte的0x80。
```python=
def pad(data):
    padlen = 16 - len(data) % 16
    #print(int('1' + '0' * 
    (padlen * 8 - 1), 2).
    to_bytes(padlen, 'big').hex())
    return data + 
    int('1' + '0' * (padlen * 8 - 1),
    2).to_bytes(padlen, 'big')

```
所以修改助教的程式碼後，由於送的oracle request後面xor的byte皆為0所以不用去管他，因為xor0x00等於自己，最後解碼時xor0x80並且加回ans並且一個byte一個byte的破解iv，即可解完一個block。

```python=
if j == 0 and k == iv[15]:
        continue
if oracle(iv[:16 - 1 - j]
+ bytes([k]) +  xor(iv[-j:], ans)
+ block):
    a=int.from_bytes(b'\x80',
    byteorder=  'big')
    ans = bytes([iv[16 - 1 - j]
    ^ a ^ k ]) + ans
    print(ans)
    break
```

**但試了很久都沒辦法順利解出第二個block，只得到一半的flag**
![](https://i.imgur.com/sxeyd7A.png)


## COR
本題可以使用暴力破解法，因為只有6個bytes要解，
故使用暴力解法，用6個for迴圈去爆破，再餵給generate去比較，放著跑了10幾天，就得出flag。

```python=
for c1 in chars:
 for c2 in chars:
  for c3 in chars:
   for c4 in chars:
    for c5 in chars:
	 for c6 in chars:
	  passwd = c1+c2+c3+c4+c5+c6
		passwd = passwd.encode()
		 if( comp( passwd, ans) ):
                 print(passwd)
		   return			
```

![](https://i.imgur.com/WbuRait.png)
