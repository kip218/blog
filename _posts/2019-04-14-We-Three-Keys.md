---
layout: post
title: "CTF Writeup: We Three Keys"
excerpt: "SwampCTF 2019 Crypto Challenge: We Three Keys"
---

# SwampCTF 2019 : [We Three Keys](https://play.swampctf.com/challenges#We%20Three%20Keys)

[Original Writeup on Github](https://github.com/kip218/CTF-writeups/tree/master/We_Three_Keys)

**Category:** Crypto  
**Points:** 144  
**Author:** noopnoop  
**Description:**  
>In this modern world of cyber, keys are the new kings of the world. In fact, we will let you use the 3 best keys that I could find, make sure to see what they offer to you!
>
> nc chal1.swampctf.com 1441
>
>[serv.py](serv.py)

## Write-up

### Challenge explanation
When I connect to the server, I am met with the following message:

![](/assets/images/We_Three_Keys/image1.png)

The server allows me to encrypt messages and decrypt messages with three different keys. The source code reveals that the server is using AES in CBC mode. It seems like I will have to recover the initialization vector (IV) or the three keys to get the flag.

### Challenge solution
The source code provided by the challenge includes the functions used to encrypt and decrypt the messages.


```python
def encrypt_message(key, IV):
    print("What message woud you like to encrypt (in hex)?")
    ptxt = raw_input("<= ")
    ptxt = pkcs7_pad(ptxt.decode('hex'))
    cipher = AES.new(key, AES.MODE_CBC, IV)
    ctxt = cipher.encrypt(ptxt)
    print ctxt.encode('hex')

def decrypt_message(key, IV):
    print("What message would you like to decrypt (in hex)?")
    ctxt = raw_input("<= ")
    ctxt = ctxt.decode('hex')
    if (len(ctxt) % 16) != 0:
        print "What a fake message, I see through your lies"
        return
    cipher = AES.new(key, AES.MODE_CBC, IV)
    ptxt = cipher.decrypt(ctxt)
    print ptxt.encode('hex')
```

The two functions take the key and the IV as parameters. Further down the soure code, I can see how the two functions are being called in the main function.

```python
print("1) Encrypt a message")
print("2) Decrypt a message")
print("3) Choose a new key")
print("4) Exit")
choice = str(raw_input("<= "))
if choice == "1":
    encrypt_message(key, key)
elif choice == "2":
    decrypt_message(key, key)
elif choice == "3":
    key = new_key()
else:
    exit()
```

The important thing to notice here is that the server is using the key as the IV. This is probably not a good idea. If I can recover the IV, I get the key since the IV *is* the key. And because the server enables me to decrypt any message of my choice, I have a method of recovering the IV.

Let's first examine how AES in CBC mode encrypts blocks of plaintext. In AES CBC, every plaintext block is xor-ed with the previous ciphertext block, and *then* encrypted using the key. For the first block, an initialization vector (IV) is used since it has no previous ciphertext block.

If C1 is the first ciphertext block and P1 is the first plaintext block, the encryption method can be expressed as:

```
C1 = Encrypt(P1 ^ IV)
C2 = Encrypt(P2 ^ C1)
C3 = Encrypt(P3 ^ C2)
```

The decryption method is basically the reverse of the encryption method. Every ciphertext is first decrypted using the key, and then xor-ed with the previous ciphertext. This can be expressed as:

```
Decrypt(C1) ^ IV = P1
Decrypt(C2) ^ C1 = P2
Decrypt(C3) ^ C2 = P3
```

Using the first two expressions above, I can derive the following equation:

```
Decrypt(C1) ^ IV ^ Decrypt(C2) ^ C1 = P1 ^ P2
```

Which can be rearranged as:

```
IV = P1 ^ P2 ^ Decrypt(C1) ^ Decrypt(C2) ^ C1
```

Conveniently, I can choose what ciphertext blocks to send. If I were to send two blocks of ciphertext that both has the value of 0, the equation would become:

```
IV = P1 ^ P2 ^ Decrypt(0) ^ Decrypt(0) ^ 0
```

I don't really care about the value of Decrypt(0), because xor-ing them cancels them out. On top of that, xor-ing something with 0 results in itself, so I can ignore the 0. So I arrive at the following equation:

```
IV = P1 ^ P2
```

Since the server allows me to send any ciphertext of my choice and returns the decrypted plaintext, all I need to do is send some ciphertext so that C1 = 0 and C2 = 0. The server will give me P1 and P2, which I can xor with each other to get the IV, which is also the key.

Using the following script, I am able to recover all three keys and putting them together gives me the flag.

```python
from pwn import *

def get_decrypted_msg(key):
    r = remote('chal1.swampctf.com', 1441)
    r.recvuntil('Pick your key, and pick wisely!\n')
    r.sendline(str(key))
    r.recv()
    r.sendline('2')
    r.recvuntil('What message would you like to decrypt (in hex)?\n')
    r.sendline('0' * 16 * 2 * 2)  # Two blocks of plaintext in hex
    r.recvuntil('<=')
    decrypted_msg = r.recvline().strip()
    r.close()
    return str(decrypted_msg, 'utf-8')

def get_flag(key):
    decrypted_msg = get_decrypted_msg(key)
    plaintext1 = decrypted_msg[:32]
    plaintext2 = decrypted_msg[32:]
    flag = int(plaintext1, 16) ^ int(plaintext2, 16)
    return bytes.fromhex(hex(flag)[2:])

print(get_flag(1) + get_flag(2) + get_flag(3))
```

![](/assets/images/We_Three_Keys/image2.png)