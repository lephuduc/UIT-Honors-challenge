# ASIS CTF

## Figole - rev
### Description
>This recipe for [Figole](https://asisctf.com/tasks/figole.apk_2b2eb4b66298cf8b30bf75115796d26fc10bcf27.txz), by ASIS, is from APK is a Full Android, one of the codebooks created in Android. We'll help you start your own personal codebook! It's easy and fun.

> Trong lúc làm bài này không hiểu sao mình bị thao túng tâm lí :))), bị nó lừa nên là mất thời gian khá nhiều mới có thể fix. Ah mình ngu quá...

Đề cho mình 1 file apk, thử mở bằng jadx
![](https://i.imgur.com/Sa4RrtX.png)

Đầu tiên mình chỉ xem mỗi class MainActivity mình nghĩ nó là luồng thực thi chính của chương trình, nhưng không...

![](https://i.imgur.com/inj3BLJ.png)

Lúc mình chưa nhận ra mình làm sai chổ nào mà không ra flag thì xem lại mainfest mới biết là DActivity mới là luồng thực thi chính. Đấy mình bị lừa chổ này thôi, phần còn lại của bài rev khác là dễ.

![](https://i.imgur.com/t17VruY.png)

Đối với bài này không cần cài đặt mà chúng ta chỉ cần chú ý vào hàm main, có thể thấy đây là một bài check flag bình thường.

![](https://i.imgur.com/FQFW5Cj.png)

Đầu tiên nó lấy và 2 chuỗi `q` và `y`, sau đó input text của mình sẽ được đem xử lí cùng `q` và so sánh với `y`.

`q` và `y` lần lượt lấy từ `l2()` và `l1()`

![](https://i.imgur.com/B7jMMvf.png)

Về `l1()` có thể thấy nó gọi 1 method khác có tên là `gf()`

![](https://i.imgur.com/xRPyKTR.png)

`gf()` sẽ gọi `gd()`, có thể thấy nó lấy chuỗi đầu tiên từ database tên calibri.

![](https://i.imgur.com/qNbJK6D.png)

![](https://i.imgur.com/qM7CObh.png)

Tìm file `calibri.ttf`, đổi đuôi file thành `.db` và mở bằng SQLite ta có 2 chuỗi như sau:

![](https://i.imgur.com/8DlD8vd.png)

Ta có base64 sẽ là `y` và tương tự với `l2()`, chuỗi còn lại sẽ là `q`

Trước khi gọi`l3()`, q được xử lí qua `cdec()`, `cdec()` chỉ là xor `q` với "key".

![](https://i.imgur.com/IJaT54x.png)

![](https://i.imgur.com/AzIwJhu.png)

Còn về `l3()` sẽ gọi method có tên là `ech()`

![](https://i.imgur.com/HC04O3n.png)

![](https://i.imgur.com/9wjoBeZ.png)

`ech()` sẽ cộng `q` với 1 chuỗi sau khi được decode bằng base64, và xử lí bằng `getK()`, `getK()` sẽ sẽ trả về chuỗi ở vị trí chẳn để làm key, còn chuỗi ở vị trí lẻ sẽ làm IV, sau đó trả về kết quả là base64 và so sánh với y.

Tới đây có vẻ mọi thứ đã rõ ràng, mình dùng script này để decode flag:

```python
from Crypto.Cipher import AES
import base64
from pwn import *
y = base64.b64decode(b'7mePfqpM6Wd1El2sj4dlUboU6PieF7La8IJ1e76cfp4=')
k = b"key"
x = base64.b64decode(b'Mzg2OTM3NjEzNDc0MzYzMTM1MzUzMjM2MzMzMjMxMzA=').decode()
q = '58504e58564f5d504a59534a58544e59564d5d504a5e524d585c4f5a564f5c57'
getK = bytes.fromhex(xor(bytes.fromhex(q),k).decode() + x)
key = []
iv = []
for i in range(len(getK)):
    if i%2==0:
        key.append(getK[i])
    else:
        iv.append(getK[i])
key =bytes(key)
iv = bytes(iv)
cipher = AES.new(key,AES.MODE_CBC,iv[:16])
plaintext = cipher.decrypt(y)
print(plaintext)
#b'ASIS{D3x_iZ_n0t_fOn7!}\n\n\n\n\n\n\n\n\n\n'
```
Flag: `ASIS{D3x_iZ_n0t_fOn7!}`

Bài này làm mình mất thời gian ở chổ phân tích sai class với lại IV và Key  mình đều tính ra là length = 24 cho nên mình nghĩ là sai chổ này, ai ngờ là sai flow chương trình, xem như qua bài này mình cần cẩn thận hơn, tìm hiểu kĩ hơn và bình tĩnh giải quyết.

## Chaffymasking - Crypto

## Description

[Chaffy masking](https://asisctf.com/tasks/chaffymasking_ee328d22852581d833c4792c61b8735ae140d81b.txz) is a popular cryptography technique that is used to protect cryptographic implementations against several attacks.

    nc 65.21.255.31 31377

Thử netcat tới server và nhập thử:

![](https://i.imgur.com/cUmfmuV.png)

Xem thử file đề cho:

![](https://i.imgur.com/WXxpnDz.png)

Ta thấy có 1 chổ là flag sẽ xor với `out_1` và `out_2` sau đó trả về và print ra:

Có nghĩ là để tìm flag ta chỉ cần xor ngược kết quả trả về và `out_1` và `out_2` và ta phải biết input của mình là gì để có thể lấy được `out_1` `out_2` trên máy mình.

![](https://i.imgur.com/qg3hzDg.png)

Vậy chổ này đổi thành return `out_1`,`out_2`

![](https://i.imgur.com/DwlInXi.png)

Tới đây chỉ cần lấy output của server rồi xor với `out_1`,`out_2` là xong

![](https://i.imgur.com/GSQefld.png)

![](https://i.imgur.com/mn2Cfpl.png)
```python
if __name__ == '__main__':

	mask = '2a202f66d560d5831f1a0550f14ed5840e17395dcf5fdca8081c0a59c75fdd98052c0f41f15bd5843400095ac143db98041c095ac143db98042c0354dd75958a'
	mask = list(bytes.fromhex(mask))
	mask = np.array(mask,dtype=int)
	salt = b'a'*64 + b'b'*64
	out1 , out2 = chaffy_mask(salt,main(),512, 64)
	out1 = np.reshape(out1,(64,))
	out2 = np.reshape(out2,(64,))
	print(bytes((out1^out2^mask)%256).replace(b'\x00',b''))
    #ASIS{Lattice_based_hash_collision_it_was_sooooooooooooooo_easY!}
```

Flag: `ASIS{Lattice_based_hash_collision_it_was_sooooooooooooooo_easY!}`
