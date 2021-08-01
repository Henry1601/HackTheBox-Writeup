# Bypass
AUTHOR: Baikuya
## Discription
> The Client is in full control. Bypass the authentication and read the key to get the Flag. [Bypass.exe](https://github.com/Henry1601/HackTheBox-Writeup/blob/main/Reverse%20Engineering/Bypass/Bypass.exe)
## Solution
```bash
	$ file Bypass.exe             
	Bypass.exe: PE32 executable (console) Intel 80386 Mono/.Net assembly, for MS Windows
```
Look like we are having a Windows executable file. So, let's move to Windows! On Windows, we have many tools similar to `file` in Linux and the most popular which I'm using is TrID (`trid`).
> You can install it from here: [Marco Pontello's Home - Software - TrID](https://mark0.net/soft-trid-e.html)

```bash
	>trid Bypass.exe
	
	TrID/32 - File Identifier v2.24 - (C) 2003-16 By M.Pontello
	Definitions found:  13983
	Analyzing...
	
	Collecting data from file: Bypass.exe
	 72.5% (.EXE) Generic CIL Executable (.NET, Mono, etc.) (73123/4/13)
	 10.4% (.EXE) Win64 Executable (generic) (10525/10/4)
	  6.5% (.DLL) Win32 Dynamic Link Library (generic) (6578/25/2)
	  4.4% (.EXE) Win32 Executable (generic) (4505/5/1)
	  2.0% (.EXE) OS/2 Executable (generic) (2029/13)
```
As we can see, the codebase was written in .NET, this will be our first clue. If you are using Linux, `strings` also gives us same result.
```bash
	$ strings Bypass.exe
	...
	HTBChallange
	.NETFramework,Version=v4.5.2
	FrameworkDisplayName
	.NET Framework 4.5.2
	Copyright 
	  2019
	_CorExeMain
	mscoree.dll
	...
```
Let's run the program to see what it does.
```bash
	>Bypass.exe
	Enter a username: henry
	Enter a password: henry
	Wrong username and/or password
	Enter a username: 
```
The program keep prompting for username and password until they are correct. Now, I use **dnSpy** - a specific tool for debugging and reversing .NET source code to reverse this program.
