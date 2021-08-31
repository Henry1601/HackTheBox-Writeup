# Impossible Password
AUTHOR: decoder
## Discription
> Are you able to cheat me and get the flag? [impossible_password.bin](https://github.com/Henry1601/HackTheBox-Writeup/blob/main/Reverse%20Engineering/Impossible%20Password/impossible_password.bin)
## Solution
```bash
	$ file impossible_password.bin 
	impossible_password.bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=ba116ba1912a8c3779ddeb579404e2fdf34b1568, stripped
	$ chmod +x impossible_password.bin
	$ ./impossible_password.bin
	* henry
	[henry]
```
So far we know that this is an executable file and it's stripped, which means there is no debug info for us to work on. By the way, each time we input, it return back the input string and exit. I tried using `ltrace` command and found this:
```bash
	$ ltrace ./impossible_password.bin
	__libc_start_main(0x40085d, 1, 0x7fffe75e06f8, 0x4009e0 <unfinished ...>
	printf("* ")						= 2
	__isoc99_scanf(0x400a82, 0x7fffe75e05e0, 0, 0* henry
	)							= 1
	printf("[%s]\n", "henry"[henry]
	)							= 8
	strcmp("henry", "**************")			= 21
	exit(1 <no return ...>
	+++ exited (status 1) +++
```
As you can see, the program compare the input string with the key `SuperSeKretKey` then exit. Let's try the key
```bash
	$ ./impossible_password.bin 
	* **************
	[**************]
	** 
```
Look like there are multilevel password protection. Again, I'm using `ltrace` to look into the program process.
```bash
	$ ltrace ./impossible_password.bin
	__libc_start_main(0x40085d, 1, 0x7fff7a2ad728, 0x4009e0 <unfinished ...>
	printf("* ")
	__isoc99_scanf(0x400a82, 0x7fff7a2ad610, 0, 0* **************
	)
	printf("[%s]\n", "**************"[**************]
	)
	strcmp("**************", "**************")
	printf("** ")
	__isoc99_scanf(0x400a82, 0x7fff7a2ad610, 0, 0** henry
	)
	time(0)
	srand(0x9518ccc9, 10, 0x939b25b8, 0)
	malloc(21)
	rand(0x1484ac0, 21, 33, 0x1484ad0)
	...
	strcmp("henry", "9$LGP*n(1R/)"DDF5%6;")
```
I've tried a few times and relized that each time I input into the second level, the program generated different key although input strings are the same. The process also contain a lot of `rand()` and `srand()` functions (these are random number generators). By the way, this program had been stripped, which means it difficult for us to analyze it step by step since all the debug info are deleted. Let's take a look inside to see if we can find anything.
```bash
	$ r2 -Ad impossible_password.bin
	[0x7f73c4480090]> afl | grep "main"
	0x00400610    1 6            sym.imp.__libc_start_main
	0x0040085d    5 283          main
	[0x7f73c4480090]> s main
	[0x0040085d]> V
```
And here is the main function:
```bash
┌ 283: main ();
│           ; var int64_t var_50h @ rbp-0x50
│           ; var int64_t var_44h @ rbp-0x44
│           ; var int64_t var_40h @ rbp-0x40
│           ; var int64_t var_3fh @ rbp-0x3f
│           ; var int64_t var_3eh @ rbp-0x3e
│           ; var int64_t var_3dh @ rbp-0x3d
│           ; var int64_t var_3ch @ rbp-0x3c
│           ; var int64_t var_3bh @ rbp-0x3b
│           ; var int64_t var_3ah @ rbp-0x3a
│           ; var int64_t var_39h @ rbp-0x39
│           ; var int64_t var_38h @ rbp-0x38
│           ; var int64_t var_37h @ rbp-0x37
│           ; var int64_t var_36h @ rbp-0x36
│           ; var int64_t var_35h @ rbp-0x35
│           ; var int64_t var_34h @ rbp-0x34
│           ; var int64_t var_33h @ rbp-0x33
│           ; var int64_t var_32h @ rbp-0x32
│           ; var int64_t var_31h @ rbp-0x31
│           ; var int64_t var_30h @ rbp-0x30
│           ; var int64_t var_2fh @ rbp-0x2f
│           ; var int64_t var_2eh @ rbp-0x2e
│           ; var int64_t var_2dh @ rbp-0x2d
│           ; var int64_t var_20h @ rbp-0x20
│           ; var int64_t var_ch @ rbp-0xc
│           ; var int64_t var_8h @ rbp-0x8
│           0x0040085d      55             push rbp
│           0x0040085e      4889e5         mov rbp, rsp
│           0x00400861      4883ec50       sub rsp, 0x50
│           0x00400865      897dbc         mov dword [var_44h], edi
│           0x00400868      488975b0       mov qword [var_50h], rsi
│           0x0040086c      48c745f8700a.  mov qword [var_8h], str.**************    ; 0x400a70 ; "**************"
│           0x00400874      c645c041       mov byte [var_40h], 0x41    ; 'A' ; 65
│           0x00400878      c645c15d       mov byte [var_3fh], 0x5d    ; ']' ; 93
│           0x0040087c      c645c24b       mov byte [var_3eh], 0x4b    ; 'K' ; 75
│           0x00400880      c645c372       mov byte [var_3dh], 0x72    ; 'r' ; 114
│           0x00400884      c645c43d       mov byte [var_3ch], 0x3d    ; '=' ; 61
│           0x00400888      c645c539       mov byte [var_3bh], 0x39    ; '9' ; 57
│           0x0040088c      c645c66b       mov byte [var_3ah], 0x6b    ; 'k' ; 107
│           0x00400890      c645c730       mov byte [var_39h], 0x30    ; '0' ; 48
│           0x00400894      c645c83d       mov byte [var_38h], 0x3d    ; '=' ; 61
│           0x00400898      c645c930       mov byte [var_37h], 0x30    ; '0' ; 48
│           0x0040089c      c645ca6f       mov byte [var_36h], 0x6f    ; 'o' ; 111
│           0x004008a0      c645cb30       mov byte [var_35h], 0x30    ; '0' ; 48
│           0x004008a4      c645cc3b       mov byte [var_34h], 0x3b    ; orax
│           0x004008a8      c645cd6b       mov byte [var_33h], 0x6b    ; 'k' ; 107
│           0x004008ac      c645ce31       mov byte [var_32h], 0x31    ; '1' ; 49
│           0x004008b0      c645cf3f       mov byte [var_31h], 0x3f    ; '?' ; 63
│           0x004008b4      c645d06b       mov byte [var_30h], 0x6b    ; 'k' ; 107
│           0x004008b8      c645d138       mov byte [var_2fh], 0x38    ; '8' ; 56
│           0x004008bc      c645d231       mov byte [var_2eh], 0x31    ; '1' ; 49
│           0x004008c0      c645d374       mov byte [var_2dh], 0x74    ; 't' ; 116
│           0x004008c4      bf7f0a4000     mov edi, 0x400a7f
│           0x004008c9      b800000000     mov eax, 0
│           0x004008ce      e82dfdffff     call sym.imp.printf
│           0x004008d3      488d45e0       lea rax, [var_20h]
│           0x004008d7      4889c6         mov rsi, rax
│           0x004008da      bf820a4000     mov edi, str._20s           ; 0x400a82 ; "%20s"
│           0x004008df      b800000000     mov eax, 0
│           0x004008e4      e887fdffff     call sym.imp.__isoc99_scanf
│           0x004008e9      488d45e0       lea rax, [var_20h]
│           0x004008ed      4889c6         mov rsi, rax
│           0x004008f0      bf870a4000     mov edi, str.__s__n         ; 0x400a87 ; "[%s]\n"
│           0x004008f5      b800000000     mov eax, 0
│           0x004008fa      e801fdffff     call sym.imp.printf
│           0x004008ff      488b55f8       mov rdx, qword [var_8h]
│           0x00400903      488d45e0       lea rax, [var_20h]
│           0x00400907      4889d6         mov rsi, rdx
│           0x0040090a      4889c7         mov rdi, rax
│           0x0040090d      e81efdffff     call sym.imp.strcmp
│           0x00400912      8945f4         mov dword [var_ch], eax
│           0x00400915      837df400       cmp dword [var_ch], 0
│       ┌─< 0x00400919      740a           je 0x400925
│       │   0x0040091b      bf01000000     mov edi, 1
│       │   0x00400920      e85bfdffff     call sym.imp.exit
│       └─> 0x00400925      bf8d0a4000     mov edi, 0x400a8d
│           0x0040092a      b800000000     mov eax, 0
│           0x0040092f      e8ccfcffff     call sym.imp.printf
│           0x00400934      488d45e0       lea rax, [var_20h]
│           0x00400938      4889c6         mov rsi, rax
│           0x0040093b      bf820a4000     mov edi, str._20s           ; 0x400a82 ; "%20s"
│           0x00400940      b800000000     mov eax, 0
│           0x00400945      e826fdffff     call sym.imp.__isoc99_scanf
│           0x0040094a      bf14000000     mov edi, 0x14               ; 20
│           0x0040094f      e839feffff     call 0x40078d
│           0x00400954      4889c2         mov rdx, rax
│           0x00400957      488d45e0       lea rax, [var_20h]
│           0x0040095b      4889d6         mov rsi, rdx
│           0x0040095e      4889c7         mov rdi, rax
│           0x00400961      e8cafcffff     call sym.imp.strcmp
│           0x00400966      85c0           test eax, eax
│       ┌─< 0x00400968      750c           jne 0x400976
│       │   0x0040096a      488d45c0       lea rax, [var_40h]
│       │   0x0040096e      4889c7         mov rdi, rax
│       │   0x00400971      e802000000     call 0x400978
│       └─> 0x00400976      c9             leave
└           0x00400977      c3             ret
```
Now, just focus on the main process of the program. At address `[0x004008ce]` is a `printf` function, after that is a `scanf` function at `[0x004008e4]` which prompts for `"%20s"` input (20-character string). I guess this might be the first password level that we have found at the beginning "**SuperSeKretKey**".

As you remember, each time we input at the first level, the program return the input in square bracket "[ ]". So my guess is true since there is a `printf` function at `[0x004008fa]` which print out `"[%s]\n"` and further, the `strcmp` function at `[0x0040090d]` compares `[var_8h]` ("SuperSeKretKey") and `[var_20h]` (input string).

Move on at `[0x0040092f]` and `[0x00400945]` are another "printf-scanf" pair, we are here at the second password level. After doing something at `[0x0040094f]` (this must be the random key generator we've mentioned at first), the program compares key with `[var_20h]` (second input). If two strings are match, the program would execute `[0x00400971]` (might be the flag!), or else, exit.

Instead of finding out what the function do inside to generate key, patching the program to change the way it execute is another way to reveal the flag. We can rewrite command `jne` at `[0x00400968]` into `je` so that it will always lead to flag-reveal-function at `[0x00400971]` no matter what we input.

Now, reopen the program in write-mode and patch the binary
```bash
	$ r2 -Aw impossible_password.bin
	...
	[0x004006a0]> s main
	[0x0040085d]> V
```
> `-w` parameter is to open file in write mode

Using arrow keys to find the command we want to change, press "Shift + A" and rewrite `je 0x400976`, then hit Enter and confirm. Now the program has been changed, let's quit radare and try our result:
```bash
	$ ./impossible_password.bin
	* **************
	[**************]
	** henry
	HTB{***************}
```

*Have fun hacking!*
Henry.
