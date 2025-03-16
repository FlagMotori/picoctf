---
tags:
  - Cryptography
---
# Custom_encryption

 اومدیم با چالش خفن کریپتوگرافی!
 [picoCTF](https://play.picoctf.org/practice/challenge/412) 
 که راهش رو توضیح بدیم.

فایل enc_flag شامل اطلاعات زیر است:
``` text
a = 94
b = 29
cipher is: [260307, 491691, 491691, 2487378, 2516301, 0, 1966764, 1879995, 1995687, 1214766, 0, 2400609, 607383, 144615, 1966764, 0, 636306, 2487378, 28923, 1793226, 694152, 780921, 173538, 173538, 491691, 173538, 751998, 1475073, 925536, 1417227, 751998, 202461, 347076, 491691]
```

و کد سورس پایتون به این صورت است:
``` py
from random import randint
import sys

def generator(g, x, p):
    return pow(g, x) % p

def encrypt(plaintext, key):
    cipher = []
    for char in plaintext:
        cipher.append(((ord(char) * key*311)))
    return cipher

def is_prime(p):
    v = 0
    for i in range(2, p + 1):
        if p % i == 0:
            v = v + 1
    if v > 1:
        return False
    else:
        return True

def dynamic_xor_encrypt(plaintext, text_key):
    cipher_text = ""
    key_length = len(text_key)
    for i, char in enumerate(plaintext[::-1]):
        key_char = text_key[i % key_length]
        encrypted_char = chr(ord(char) ^ ord(key_char))
        cipher_text += encrypted_char
    return cipher_text

def test(plain_text, text_key):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    a = randint(p-10, p)
    b = randint(g-10, g)
    print(f"a = {a}")
    print(f"b = {b}")
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    shared_key = None
    if key == b_key:
        shared_key = key
    else:
        print("Invalid key")
        return
    semi_cipher = dynamic_xor_encrypt(plain_text, text_key)
    cipher = encrypt(semi_cipher, shared_key)
    print(f'cipher is: {cipher}')

if __name__ == "__main__":
    message = sys.argv[1]
    test(message, "trudeau")
```

از کد مشخص است که رمزنگاری ترکیبی از موارد زیر است:

معکوس کردن ترتیب کاراکترها در تابع dynamic_xor_encrypt (با استفاده از plaintext[::-1])،
عملیات XOR در تابع dynamic_xor_encrypt (با ord(char) ^ ord(key_char) )،
ضرب مقادیر ASCII کاراکترها در تابع encrypt (با ord(char) * key * 311).
برای رمزگشایی، باید عملیات‌ها را به ترتیب معکوس و با عملیات معکوس انجام دهیم:

تقسیم به جای ضرب در تابع decrypt،
XOR با همان کلید در تابع dynamic_xor_decrypt (چون معکوس XOR همان XOR است)،
معکوس کردن ترتیب کاراکترها.

اسکریپت نهایی رمزگشایی به این صورت است:

``` py
# داده‌شده در فایل enc_flag
a = 94
b = 29
cipher = [260307, 491691, 491691, 2487378, 2516301, 0, 1966764, 1879995, 1995687, 1214766, 0, 2400609, 607383, 144615, 1966764, 0, 636306, 2487378, 28923, 1793226, 694152, 780921, 173538, 173538, 491691, 173538, 751998, 1475073, 925536, 1417227, 751998, 202461, 347076, 491691]

# کلید متنی که به تابع test ارسال شده
text_key = "trudeau"

# تابع بدون تغییر
def generator(g, x, p):
    return pow(g, x) % p

# تقسیم به جای ضرب
def decrypt(ciphertext, key):
    plain = []
    for num in ciphertext:
        plain.append(chr(int(num / key / 311)))
    return plain

# بدون معکوس کردن ترتیب کاراکترها
def dynamic_xor_decrypt(ciphertext, key):
    text = ""
    key_length = len(key)
    for i, char in enumerate(ciphertext):
        key_char = key[i % key_length]
        decrypted_char = chr(ord(char) ^ ord(key_char))
        text += decrypted_char
    return text

if __name__ == "__main__":
    # کد بدون تغییر از تابع test
    p = 97
    g = 31
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    shared_key = None
    if key == b_key:
        shared_key = key
    else:
        print("کلید نامعتبر")
        exit(1)  # خروج به جای بازگشت
    
    plain = decrypt(cipher, shared_key)
    flag = dynamic_xor_decrypt(plain, text_key)
    print(flag[::-1])
```

اسکریپت را اجرا می‌کنیم تا فلگ به دست آید

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{custom_d2cr0pt6d_751a...}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
