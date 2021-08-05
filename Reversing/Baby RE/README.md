# Baby RE
AUTHOR: Xh4H
## Discription
> Show us your basic skills! (P.S. There are 4 ways to solve this, are you willing to try them all?) [baby](https://github.com/Henry1601/HackTheBox-Writeup/blob/main/Reverse%20Engineering/Baby%20RE/baby)
## Solution
```bash
	$ file baby
	baby: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=25adc53b89f781335a27bf1b81f5c4cb74581022, for GNU/Linux 3.2.0, not stripped
	$ chmod +x baby
	$ ./baby
	Insert key: 
	aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
	Try again later.
```
So far we know that it is an executable file and it's not stripped, which means we can debug it. But first, use command `strings` to see if we could find anything. And I found this:
```bash
	$ strings baby
	...
	HTB{B4BYH
	_R3V_TH4H
	TS_Ef
	[]A\A]A^A_
	Dont run `strings` on this challenge, that is not the way!!!!
	Insert key: 
	abcde122313
	Try again later.
	...
```
Now try the key we found here
```bash
	$ ./baby
	Insert key:
	abcde122313
	HTB{B4BY_R3V_TH4TS_EZ}
```
Kind of easy right? In the discription they say there are 4 ways to solve this challenge. Let's find the others!
I try `ltrace` and got this:
```bash
	$ ltrace ./baby
	puts("Insert key: "Insert key: 
	)						= 13
	fgets(aaaaaaaaaaa
	"aaaaaaaaaaa\n", 20, 0x7f31bed55980)		= 0x7fff0c26ba30
	strcmp("aaaaaaaaaaa\n", "abcde122313\n")	= -1
	puts("Try again later."Try again later.
	)						= 17
	+++ exited (status 0) +++
```
Now, try using debugger
```bash
	$ r2 -Ad baby
	...
	[0x7fb7329f7090]> afl | grep "main"
	0x55fe06f42fe0    1 4124         reloc.__libc_start_main
	0x55fe06f40155    4 152          main
	[0x7fb7329f7090]> s main
	[0x55fe06f40155]> V
```
And in the disassembler, I found this:
```bash
	0x55fe06f40190      488d35bc0e00.  lea rsi, str.abcde122313_n    ; 0x55fe06f41053 ; "abcde122313\n"
	0x55fe06f40197      4889c7         mov rdi, rax
	0x55fe06f4019a      e8b1feffff     call sym.imp.strcmp     ;[3]
	0x55fe06f4019f      85c0           test eax, eax
	0x55fe06f401a1      7537           jne 0x55fe06f401da
	0x55fe06f401a3      48b84854427b.  movabs rax, 0x594234427b425448    ; 'HTB{B4BY'
	0x55fe06f401ad      48ba5f523356.  movabs rdx, 0x3448545f5633525f    ; '_R3V_TH4'
	0x55fe06f401b7      488945c0       mov qword [var_40h], rax
	0x55fe06f401bb      488955c8       mov qword [var_38h], rdx
	0x55fe06f401bf      c745d054535f.  mov dword [var_30h], 0x455f5354    ; 'TS_E'
	0x55fe06f401c6      66c745d45a7d   mov word [var_2ch], 0x7d5a    ; 'Z}'
```

*Have fun hacking!*
## Flag
`HTB{B4BY_R3V_TH4TS_EZ}`
