---
tags:
  - Forensics
---
# Verify

 اومدیم با یک چالش نسبتا ساده ی فارنزیک!
 [picoCTF](https://play.picoctf.org/practice/challenge/450) 
 و قراره بترکونیمش.

توضیحات چالش :

People keep trying to trick my players with imitation flags. I want to make sure they get the real thing! I'm going to provide the SHA-256 hash and a decrypt script to help you know that my flags are legitimate.


همین فایل‌ها از طریق SSH در اینجا قابل دسترسی هستند:
ssh -p 52722 ctf-player@rhea.picoctf.net
با رمز عبور 83dcefb7. اثر انگشت را با "yes" بپذیرید و پس از اتصال با "ls" شروع کنید. 
به یاد داشته باشید که در شل، رمزهای عبور مخفی هستند!

چک‌سام: 467a10447deb3d4e17634cacc2a68ba6c2bb62a6637dad9145ea673bf0be5e02
برای رمزگشایی فایل پس از تأیید هش، دستور زیر را اجرا کنید: ./decrypt.sh files/<file>

نکات:
1. چک‌سام‌ها به شما اجازه می‌دهند بفهمید آیا فایل کامل است و از منبع اصلی آمده یا نه. اگر هش مطابقت نداشته باشد، فایل متفاوتی است.
2. می‌توانید چک‌سام SHA یک فایل را با دستور sha256sum <file> یا برای همه فایل‌های یک دایرکتوری با sha256sum <directory>/* ایجاد کنید.
3. به یاد داشته باشید که می‌توانید خروجی یک دستور را با | به دستور دیگری هدایت کنید. اگر گیر کردید، چالش "First Grep" را تمرین کنید!


ابتدا فایل زیپ را باز می‌کنیم:

``` shell
└─$ unzip challenge.zip 
Archive:  challenge.zip
   creating: home/ctf-player/drop-in/
   creating: home/ctf-player/drop-in/files/
 extracting: home/ctf-player/drop-in/files/LmicJDs8  
 extracting: home/ctf-player/drop-in/files/c6c8b911  
 extracting: home/ctf-player/drop-in/files/3eJU0bPR  
 extracting: home/ctf-player/drop-in/files/EfRHiDLP  
 extracting: home/ctf-player/drop-in/files/DBQbeL0I
```

``` shell
└─$ sha256sum home/ctf-player/drop-in/files/* | grep 467a10447deb3d4e17634cacc2a68ba6c2bb62a6637dad9145ea673bf0be5e02
467a10447deb3d4e17634cacc2a68ba6c2bb62a6637dad9145ea673bf0be5e02  home/ctf-player/drop-in/files/c6c8b911
└─$ cat home/ctf-player/drop-in/files/c6c8b911
Salted__���05�.Q�+�P��&pE�?B�{M:��e�Wm�a4Wua��l�5�yU!����NA�
```

فایل رمزنگاری‌شده است.
در قدم بعدی، سعی می‌کنیم فایل را رمزگشایی کنیم:

فایل را با openssl به این صورت رمزگشایی می‌کنیم:

``` shell
└─$ openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -salt -in files/c6c8b911 -k picoCTF
picoCTF{trust_but_verify_e018b574}
```

و فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{trust_but_verify_e018b574}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
