---
tags:
  - ReverseEngineering
---
# FastCheck

 اومدیم با چالش سنگین مهندسی معکوس!
 [picoCTF](https://play.picoctf.org/practice/challenge/416) 
 و ببینیم دنیا دست کیه.

ابتدا با تحلیل اولیه فایل شروع می‌کنیم:
``` sh
└─$ file bin                    
bin: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d134239fc06b6e50d2b04696cac10504a052fcfd, for GNU/Linux 3.2.0, not stripped
```

فایل یک باینری ۶۴ بیتی ELF است.

در ادامه، رشته‌های جالب را بررسی می‌کنیم:
``` sh
└─$ strings -n 6 bin
/lib64/ld-linux-x86-64.so.2
libstdc++.so.6
GLIBC_2.2.5
[]A\A]A^A_
picoCTF{picoCTF{wELF_d0N3
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
crtstuff.c
```

اوه! به نظر می‌رسد ابتدای پرچم را پیدا کردیم.
تحلیل استاتیک در Ghidra
برای ادامه، فایل را در Ghidra دیکامپایل می‌کنیم و کد را بررسی می‌کنیم. فایل را در Ghidra وارد کرده و با تنظیمات پیش‌فرض تحلیل کنید. روی تابع main دوبار کلیک کنید تا نسخه دیکامپایل‌شده آن نمایش داده شود:

``` cpp
undefined8 main(void)

{
  char cVar1;
  char *pcVar2;
  long in_FS_OFFSET;
  allocator<char> local_249;
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_248 [32];
  basic_string local_228 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_208 [32];
  basic_string local_1e8 [32];
  basic_string local_1c8 [32];
  basic_string local_1a8 [32];
  basic_string local_188 [32];
  basic_string local_168 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_148 [32];
  basic_string local_128 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_108 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_e8 [32];
  basic_string local_c8 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_a8 [32];
  basic_string local_88 [32];
  basic_string local_68 [32];
  basic_string<char,std::char_traits<char>,std::allocator<char>> local_48 [40];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  std::allocator<char>::allocator();
                    /* try { // try from 001012cf to 001012d3 has its CatchHandler @ 00101975 */
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_248,(allocator *)"picoCTF{wELF_d0N3");
  std::allocator<char>::~allocator(&local_249);
  /* ... ادامه کد ... */
```

در ابتدای کد، دوباره بخش اول پرچم را می‌بینیم. هر خطی که به شکل زیر است:
``` cpp
std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_228,(allocator *)&DAT_0010201d);
```

به یک کاراکتر اشاره دارد. می‌توانید روی ارجاع حافظه (مثلاً DAT_0010201d) دوبار کلیک کنید تا ببینید به چه کاراکتری مربوط است، سپس آن را با Rename Global به چیزی مثل char_X تغییر نام دهید.

تحلیل دینامیک در GDB
این روش کمی پیچیده و زمان‌بر است، بنابراین به رویکرد دینامیک با استفاده از gdb سوئیچ می‌کنیم. ابتدا gdb را با افزونه GEF اجرا می‌کنیم:

``` sh
└─$ gdb-gef -q bin         
GEF for linux ready, type `gef' to start, `gef config' to configure
gef➤
```

می‌خواهیم در اولین دستور اسمبلی پس از اضافه شدن کاراکتر } توقف کنیم (آدرس نسبی که با 0x01860 پایان می‌یابد):

``` sh
gef➤  disass main
Dump of assembler code for function main:
   0x0000000000001289 <+0>:     endbr64
   0x0000000000001299 <+16>:    mov    rax,QWORD PTR fs:0x28
   0x000000000000185b <+1490>:  call   0x1100 <_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEpLEc@plt>
   0x0000000000001860 <+1495>:  mov    ebx,0x0
```

آفست موردنظر main+1495 است. یک نقطه توقف (breakpoint) تنظیم کرده و برنامه را اجرا می‌کنیم:
``` sh
gef➤  break *main+1495
Breakpoint 1 at 0x1860
gef➤  run
```

وقتی برنامه متوقف می‌شود، پرچم کامل در رجیسترهای RAX و RDI و همچنین روی استک قابل مشاهده است:

``` sh
Breakpoint 1, 0x0000555555555860 in main ()
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007fffffffdae0  →  0x000055555556b2d0  →  "picoCTF{wELF_d0N3_mate_e9da2c0e}"
$rdi   : 0x00007fffffffdae0  →  0x000055555556b2d0  →  "picoCTF{wELF_d0N3_mate_e9da2c0e}"
$rip   : 0x0000555555555860  →  <main+1495> mov ebx, 0x0
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdae0│+0x0010: 0x000055555556b2d0  →  "picoCTF{wELF_d0N3_mate_e9da2c0e}"    ← $rax, $rdi
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
 → 0x555555555860 <main+1495>      mov    ebx, 0x0
gef➤
```

و فلگ پیدا شد !

??? success "FLAG :triangular_flag_on_post:"
    <div dir="ltr">`picoCTF{wELF_d0N3_mate_e9da2c0e}`</div>

--- 

!!! نویسنده
    [sw33tw3as3l](https://github.com/sw33tw3as3l)
