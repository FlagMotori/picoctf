---
tags:
  - WEB
---
# WeirdSnake

 این چالش یک نکته کلیدی رو به همراه داره!!
 [picoCTF](https://play.picoctf.org/practice/challenge/428) 
 و قراره راه حلش رو شرح بدیم.


سورس کد فرانت در مرورگر:
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Help!</title>
</head>
<body>
    <h1>وب‌سایت من هک شده است!</h1>
    <p>لطفاً پیام مخفی را پیدا کنید!</p>
    <!-- Hmmm what could they have done -->
</body>
</html>
```

از آنجا که کد منبع چیزی بیشتر از این نشان نمی‌دهد، از ابزارهای توسعه‌دهنده مرورگر (DevTools) استفاده می‌کنیم. با زدن کلید F12 یا راست‌کلیک و انتخاب "Inspect"، تب‌های مختلف را بررسی می‌کنیم:

تب Elements: همان کد HTML را نشان می‌دهد.
تب Network: درخواست‌های شبکه را بررسی می‌کنیم. صفحه را رفرش می‌کنیم و می‌بینیم که فقط یک درخواست به / (صفحه اصلی) ارسال شده است.
تب Sources: فایل‌های بارگذاری‌شده را نشان می‌دهد، اما چیزی جز فایل HTML نیست.

در چالش‌های وب، فایل robots.txt اغلب حاوی سرنخ است. این فایل معمولاً برای موتورهای جستجو استفاده می‌شود، اما گاهی هکرها یا طراحان چالش اطلاعاتی در آن مخفی می‌کنند. آدرس را تست می‌کنیم:

``` text
User-agent: *
Disallow: /secret-message.txt
```

فایل robots.txt به ما می‌گوید که ربات‌ها نباید به /secret-message.txt دسترسی پیدا کنند. این یک سرنخ واضح است که باید این مسیر را بررسی کنیم!

``` text
picoCTF{4ll_y0u_h4d_t0_d0_w45_ch3ck_r0b0t5_570b3b8c}
```

این دقیقاً پرچم است! چالش به همین سادگی بود؛ هکرها (یا طراحان چالش) پرچم را در یک فایل متنی مخفی قرار داده‌اند که از طریق robots.txt قابل کشف بود.

و فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{4ll_y0u_h4d_t0_d0_w45_ch3ck_r0b0t5_570b3b8c}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
