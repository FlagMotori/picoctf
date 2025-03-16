---
tags:
  - WEB
---
# Search_Source

سلام بر همه‌ی علاقه مندان دنیای ctf !
 با یه چالش وب اومدیم از 
 [picoCTF](https://play.picoctf.org/practice/challenge/295?category=1&difficulty=2&page=1) 
 و قراره راه حلش رو شرح بدیم.

خب طبق توضیحات نویسنده‌ی چالش احتمالا یه فایلی هست که راحت میشه دانلودش کرد و یه جایی از اون فلگه.
خب بیشترین احتمال رو میدیم که این فایله یکی از فایل های javascript یا css یا html  باشه.
برای اینکه پیدا کنیم فلگ رو یک راهکار این هست که دونه دونه فایل ها رو باز کنیم و محتوای فایل رو بررسی کنیم و دنبال فلگ باشیم. 

ولی خب چون کد زدن به شدت باحال تره پس میایم یه کد میزنیم که برامون پیدا کنه فایل رو و خودش دنبال فلگ بگرده :

``` py linenums="1"
import sys
from colorama import Fore, Back, Style, init
import requests
from bs4 import BeautifulSoup
import os

results_counter = 0

def crawl_over_every_thing(url, max_depth):
    
    global results_counter
        
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")
    
    if max_depth >= 0:
        if 'pico' in response.text:
            fil = open(str(results_counter)+'.txt', 'w')
            fil.write(response.text)
            
            os.system('grep pico ' + str(results_counter)+'.txt')
            results_counter += 1

    if max_depth == 0:
        return

    links_list = soup.select('link[href]')
    for found_url in links_list:
        new_url = url + found_url['href']
        
        if 'http' in found_url['href']:
            new_url = found_url['href']
            
        print(Fore.WHITE + '[+] ' + Style.RESET_ALL, end='')
        print(Fore.GREEN + 'Searching through ' + Fore.BLUE + new_url + Style.RESET_ALL)
        
        crawl_over_every_thing(new_url, max_depth - 1)
    
if __name__ == '__main__':
    
    init(autoreset=True)
    
    
    for url in sys.argv :
        if url == 'crawler.py':
            continue
        print(Fore.WHITE + '[+] ' + Style.RESET_ALL, end='')
        print(Fore.GREEN + 'Searching through ' + Fore.BLUE + url + Style.RESET_ALL)
        crawl_over_every_thing(url, 10)

```

اینجا با توجه به خط 27 تو صفحه‌هایی که مربوط به تگ link هستن می‌گرده که فلگ تو همون جاست. برای اینکه بخوایم تغییرش بدیم برای مثال بخواد تو همه‌ی فایل های مربوط به تگ a بگرده باید به جای link از a استفاده کنیم.

و اما البته که فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{1nsp3ti0n_0f_w3bpag3s_8de925a7}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
