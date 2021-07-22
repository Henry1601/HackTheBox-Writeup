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
So far we know that this is an executable file and it's stripped, which means there are less debug info for us to work on. By the way, each time we input, it return back the input string and exit. I tried using `ltrace` command and found this:
```bash
	$ ltrace ./impossible_password.bin
	__libc_start_main(0x40085d, 1, 0x7fffe75e06f8, 0x4009e0 <unfinished ...>
	printf("* ")						= 2
	__isoc99_scanf(0x400a82, 0x7fffe75e05e0, 0, 0* henry
	)							= 1
	printf("[%s]\n", "henry"[henry]
	)							= 8
	strcmp("henry", "SuperSeKretKey")			= 21
	exit(1 <no return ...>
	+++ exited (status 1) +++
```
As you can see, the program compare the input string with the key `SuperSeKretKey` then exit. Let's try the key
```bash
	$ ./impossible_password.bin 
	* SuperSeKretKey
	[SuperSeKretKey]
	** 
```
