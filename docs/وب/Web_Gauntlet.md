---
tags:
  - وب
---
# Web_Gauntlet

سلام بر همه‌ی دوستان گرامی ctf  
 با یه چالش وب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/88?category=1&difficulty=2&page=3) 
 قراره کلی injection انجام بدیم :))


میدونیم که تو لینک [filters](http://jupiter.challenges.picoctf.org:9683/filter.php) کاراکترهای فیلتر شده رو برامون نوشته و باید بدون استفاده از اونها injetion انجام بدیم.
[site](http://jupiter.challenges.picoctf.org:9683/) باید اینجا سعی کنیم در 5 راند متوالی به عنوان ادمین وارد بشیم.


##راند یک:
از `or` نمی‌تونیم استفاده کنیم.

!!! payload
    `admin' ; -- `

صرفا کافیه که بیایم یوزرنیم رو برابر admin بذاریم و باقی ماجرا رو کامنت کنیم.


##راند دو:

####رشته های غیر مجاز:

`or` `and` `like` `=` `--`

payload:

`admin'; # `

همون داستان قبلی فقط به جای `--` از `#` استفاده می‌کنیم!

##راند سه:

####رشته های غیر مجاز:

` ` `or` `and` `=` `like` `>` `<` `--`

اینجا حتی نمی‌تونیم از space استفاده کنیم !!!

payload:

`admin';#`

همون قبلی مثل اینکه جواب میده!!!

!!! نکته
    اگر اینجا مشکل space داشتیم می‌تونستیم از `/**/` استفاده کنیم به جای space.

##راند چهار:

####رشته های غیر مجاز:

` ` `or` `and` `=` `like` `>` `<` `--` `admin`

خب عالی شد. حتی از `admin` هم نمی‌تونیم استفاده کنیم.
احتمالا باید بگردیم دنبال concat کردن استرینگ تو کامندهای sql.

payload:

`admi'||'n';#`

ظاهرا `||` به عنوان concat کار می‌کنه!

##راند  آخر:
` ` `or` `and` `=` `like` `>` `<` `--` `admin` `union`
خب ما که تو راند قبلی از `union` استفاده نکردیم. پس بیاین اینجا همون payload قبلی رو استفاده کنیم.

payload:

`admi'||'n';#`


فلگ رو یافتیم !!!

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{y0u_m4d3_1t_d846125f7bdbf4d6e89cbc5edb6fa739}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
