---
tags:
  - ReverseEngineering
---
# Packer

 اومدیم با چالش ریورس!
 [picoCTF](https://play.picoctf.org/practice/challenge/421) 
 که فلگ رو پیدا کنیم.

نام چالش و نکته درباره اندازه فایل به ما می‌گوید که از یک ابزار فشرده‌سازی (packer) برای این باینری استفاده شده است.

ابتدا تحلیل اولیه فایل را شروع می‌کنیم:
``` sh
└─$ file out            
out: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

فایل یک باینری ELF 64 بیتی است. حالا رشته‌ها را بررسی می‌کنیم:
``` sh
└─$ strings -n 8 out
AWAVAUATUS
7069636fH
Enter the password
o unlock,is file: 
lagX7069636f4354467b5539585f
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 3.95 Copyright (C) 1996-2018 the UPX Team. All Rights Reserved. $
/proc/self/exe
```

مشخص شد که از UPX (Ultimate Packer for eXecutables) استفاده شده و به نظر می‌رسد به دنبال یک رمز عبور هستیم.

فایل‌های فشرده‌شده با UPX را می‌توان با دستور upx -d باز کرد:
``` sh
└─$ upx -d -o unpacked out   
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
[WARNING] bad b_info at 0x4b718

[WARNING] ... recovery at 0x4b714

    877724 <-    336520   38.34%   linux/amd64   unpacked

Unpacked 1 file.

└─$ file unpacked       
unpacked: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=2e06e54daad34a6d4b0c7ef71b3e1ce17ffbf6db, for GNU/Linux 3.2.0, not stripped
```

حالا دوباره رشته‌ها را بررسی می‌کنیم:
``` sh
└─$ strings -n 8 unpacked | more     
AWAVAUATUSH
[]A\A]A^A_
7069636fH
4354467bH
Enter the password to unlock this file: 
You entered: %s
Password correct, please see flag: 7069636f4354467b5539585f556e5034636b314e365f42316e34526933535f65313930633366337d
Access denied
xeon_phi          <--- رمز عبور احتمالی
../csu/libc-start.c
FATAL: kernel too old
```

یک رمز عبور احتمالی (xeon_phi) و چیزی که به نظر می‌رسد پرچم کدگذاری‌شده در مبنای هگز است پیدا کردیم. این رشته با 7069636f شروع می‌شود که در ASCII به معنای pico است.

رمز عبور (xeon_phi) اشتباه بود، اما می‌توانیم پرچم را با استفاده از دستورات تک‌خطی در python یا xxd به دست آوریم.

python :

``` py
└─$ python -c "print(bytes.fromhex('7069636f4354467b5539585f556e5034636b314e365f42316e34526933535f65313930633366337d'))"
b'picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_5de...}'
```

و فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_5de...}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
