---
tags:
  - Forensics
---
# CanYouSee

 اومدیم با یک چالش فارنزیک!
 [picoCTF](https://play.picoctf.org/practice/challenge/408) 
 و سریع راهش رو شرح بدیم.


ابتدا فایل زیپ را باز می‌کنیم:

``` shell
└─$ unzip unknown.zip  
Archive:  unknown.zip
  inflating: ukn_reality.jpg
└─$ file ukn_reality.jpg 
ukn_reality.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 4308x2875, components 3
```

یک فایل JPEG داریم. می‌توانید از ابزارهایی مثل eog یا feh برای مشاهده آن در لینوکس استفاده کنید.

در مرحله بعد، متادیتای Exif را با exiftool بررسی می‌کنیم:
``` sh
└─$ exiftool ukn_reality.jpg
ExifTool Version Number         : 12.52
File Name                       : ukn_reality.jpg
Directory                       : .
File Size                       : 2.3 MB
File Modification Date/Time     : 2024:02:15 23:40:14+01:00
File Access Date/Time           : 2024:02:15 23:40:14+01:00
File Inode Change Date/Time     : 2024:02:15 23:40:14+01:00
File Permissions                : -rwxrwxrwx
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 72
Y Resolution                    : 72
XMP Toolkit                     : Image::ExifTool 11.88
Attribution URL                 : cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==
Image Width                     : 4308
Image Height                    : 2875
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 4308x2875
Megapixels                      : 12.4
```

داده‌های موجود در Attribution URL به نظر می‌رسد که به صورت base64 رمزنگاری شده باشند.
در نهایت، از base64 برای رمزگشایی استفاده می‌کنیم و پرچم را به دست می‌آریم:

``` sh
└─$ echo "cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==" | base64 -d
picoCTF{ME74D47A_HIDD3N_d8c381fd}
```

و فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{ME74D47A_HIDD3N_d8c381fd}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
