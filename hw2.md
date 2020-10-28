###### tags: `ctf`
# Hw2 write_up

## ID
bruce30709
## RSA
本題為簡單的RSA分解題目，但是由$\ p,q1,q2$三個大數相乘，分別為$q1=2 * p$後的第一個質數，$q2=3 * q1$後的第一個質數，經過簡單的數學運算相乘後，可以得知$p * q1 * q2 > 12 * p^3$，故得知要從將$n/12$開三方後的結果往前去找質數。

***這邊開三方是使用網路上的方法***
```python=
def find_invpow(x,n):
    """Finds the integer component of the n'th root of x,
    an integer such that y ** n <= x < (y + 1) ** n.
    """
    high = 1
    while high ** n <= x:
        high *= 2
    low = long(high/2)
    while low < high:
        mid = (low + high) // 2
        if low < mid and mid**n < x:
            low = mid
        elif high > mid and mid**n > x:
            high = mid
        else:
            return mid
    return mid + 1

```
接著往前去檢查，並和原本的$n$去比對，如果相同則為正確。

```python=
while 1:
    y=y-1
    #print(y)
    if(isPrime(y)):
        q1 = next_prime(2 * y)
        q2 = next_prime(3 * q1)
        if n == y*q1*q2 :
            print('got')
            break
```

最後再照RSA的解密流程解回解出m可得到明文。
```python=
q1 = next_prime(2 * y)
q2 = next_prime(3 * q1)            
d = inverse(65537,(q1-1)*(q2-1)*(y-1))
m = pow(c,d,n)
print(long_to_bytes(m))
```
得到flag!

![](https://i.imgur.com/MENnmno.png)



## LSB
本題為LSB之變型，最後回傳的bit為0,1,2，故要使用3進位的方法去回推，唯一要注意的是要使用3分搜尋方法，也就是每次都分3段，一直做下去，一樣做1024次，最後得到的那個區段就會是正解。
```python=
_2e = pow(3, e, n)
L, R = 0, 1

for i in range(1024):
    c = (c * _2e) % n
    r.sendline(str(c))
    m = int(r.recvline().split(b' = ')[1])
    
    L, R = L * 3, R * 3
    if m==0:
        L+=0
        R-=2
    elif m==1:
        L+=1
        R-=1
    else:
        L+=2
        R-=0
print(long_to_bytes(L * n // pow(3, 1024)))
print(long_to_bytes(R * n // pow(3, 1024)))
```
得到flag了!

![](https://i.imgur.com/s1dYl9P.png)
